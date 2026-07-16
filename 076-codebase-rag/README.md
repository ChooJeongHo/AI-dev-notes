# 076 - MovieFinder 코드베이스 미니 RAG — Room DB를 벡터 저장소로

**API 키 없이 Room(SQLite) 기반으로 RAG 파이프라인을 완성했고, "검색이 별로다"로 오인하기 쉬운 조용한 한글 토큰화 버그를 실측으로 발견해 고쳤다**

---

## 목적

070~074가 "AI가 코드를 어떻게 다루는가"에 집중했다면, 076일차는 **"AI가 지식을 어떻게 검색하고 답하는가"**라는 새로운 축이다.
별도 벡터DB 서버 없이, MovieFinder가 이미 쓰던 **Room을 그대로 벡터 저장소로 재사용**해서 코드베이스 자연어 질의응답 도구를 만든다.

---

## 아키텍처 — 왜 별도 모듈이 필요했나

```
:app                    ← 기존 MovieFinder 앱 (Android)
:tools:codebase-rag     ← 신규, 순수 Kotlin/JVM 모듈 (신규 추가)
```

RAG 도구는 "개발자용 CLI 스크립트"이지 앱 기능이 아니다. 그래서:
- APK에 절대 포함되지 않도록 **완전히 분리된 모듈**로 만듦
- 실행은 `./gradlew :tools:codebase-rag:ragIndex` / `ragAsk`로만 가능

### 기술적 난관 — Android 없이 Room을 어떻게 쓰나

Room은 원래 Android 프레임워크에 의존하는데, 이 모듈은 순수 JVM이다.

```kotlin
// Room 2.7+ 부터 지원하는 KMP(Kotlin Multiplatform) 방식
plugins {
    alias(libs.plugins.kotlin.multiplatform)  // kotlin("jvm") 단일 모듈로는 
    alias(libs.plugins.ksp)                    // Room의 kspJvm 변형이 지원 안 됨
    alias(libs.plugins.room)
}
```

`androidx.sqlite:sqlite-bundled` (순수 JVM용 SQLite 네이티브 바인딩)를 쓰면 **Android 기기·에뮬레이터·Robolectric 없이도 완전한 Room 데이터베이스를 데스크톱 JVM에서 그대로 돌릴 수 있다.** MovieFinder의 `MovieDatabase`와 개념은 똑같고 실행 환경만 다른 것.

> **리스크 관리:** 이 부분이 가장 실패 확률이 높은 설정이라, 나머지 코드를 쓰기 전에 **먼저 컴파일부터 검증**하고 다음 단계로 넘어갔다.

---

## 임베딩 제공자 — 인터페이스로 추상화

```kotlin
interface EmbeddingProvider {
    val dimension: Int
    fun embed(text: String): FloatArray
}
```

Claude에는 별도 임베딩 엔드포인트가 없어서(Anthropic 공식 추천 파트너는 Voyage AI), API 키 없이 지금 바로 전체 파이프라인을 검증하기 위해 **로컬 무료 구현(`LocalHashingEmbeddingProvider`)**을 기본값으로 택했다.

> **설계 포인트:** 나중에 실제 품질이 필요하면 `VoyageEmbeddingProvider`나 `OpenAiEmbeddingProvider`로 **이 인터페이스만 구현해서 1줄 교체**하면 된다. 지금은 품질보다 "파이프라인 전체가 실제로 동작하는가"를 검증하는 게 목적.

### 로컬 임베딩의 원리 — Hashing Trick (Feature Hashing)

진짜 신경망 임베딩이 아니라, 코드 식별자를 camelCase/snake_case로 토큰화한 뒤 **고정 차원(512) 벡터에 TF 가중치를 해시로 누적**하는 방식 (Vowpal Wabbit 등에서 쓰는 기법과 동일).

---

## 실전에서 발견한 두 개의 조용한 버그

### 버그 1 — 한글 질문이 전부 "0벡터"가 되고 있었다

```kotlin
// Before: ASCII만 "단어 문자"로 인정
private val NON_ALNUM = Regex("[^a-zA-Z0-9]+")

// After: 유니코드 프로퍼티 클래스로 한글도 포함
private val NON_ALNUM = Regex("[^\\p{L}\\p{N}]+")
```

**왜 위험한 버그인가:** 에러가 안 난다. 그냥 결과가 조용히 나빠지기만 해서(유사도가 전부 0.0000), 겉보기엔 "그냥 검색이 별로네"로 오인하기 쉽다.

> **일반화된 교훈:** 정규식으로 "단어 문자"를 정의할 때 `[a-zA-Z0-9]`처럼 ASCII만 가정하면 비영어권 텍스트가 조용히 사라진다. `\p{L}`, `\p{N}` 같은 유니코드 프로퍼티 클래스를 기본값으로 삼는 습관이 이런 실패를 막는다.

### 버그 2 — 함수 청크에 코드 위 한글 주석이 빠져 있었다

```kotlin
/** declLine 바로 위에 연속된 KDoc/라인주석/annotation 블록이 있으면 
 *  그 시작 줄까지 포함시킨다.
 *  Korean 주석("// 즐겨찾기 토글" 등)이 코드 바로 위에 붙는 이 프로젝트 스타일에서,
 *  이 블록이 빠지면 한글 질문과 매칭될 토큰이 거의 사라진다. */
```

MovieFinder는 함수 위에 한글 주석을 다는 스타일인데, 청커가 함수 본문만 자르고 주석을 놓치고 있었다. 이 프로젝트 특성(067~074일차 코드 전체에서 계속 봐온 한글 주석 스타일)을 RAG 설계에도 반영해야 했다.

### 추가 개선 — 한글 조사 스트리핑

```kotlin
// "추가는", "파일들을"처럼 명사에 조사가 붙어 코드 주석의 원형("추가", "파일")과
// 어긋나는 문제 완화. 원본은 유지하고 조사 뗀 변형을 "추가"만 함(교체 아님)
private val PARTICLES_1 = setOf("은", "는", "이", "가", "을", "를", ...)
```

---

## 인덱싱 결과 (실측)

| 항목 | 값 |
|---|---|
| 대상 `.kt` 파일 수 | 240개 |
| 생성된 청크 수 | 1,224개 |
| 파일 탐색 | 13ms |
| 청크 분할 | 88ms |
| 임베딩 | 52ms |
| Room 저장 | 245ms |
| **총 소요 시간** | **404ms** |

---

## 실전 테스트 — 3가지 질문 결과

### Q1. "영화 즐겨찾기 추가는 어떤 파일들을 거쳐서 처리돼?"

**검색된 청크:** `FavoriteRepository.kt`(0.213), `FavoriteMovieDao.kt`(0.208), `ToggleFavoriteUseCase.kt`(0.183) 등

**답변:** `DetailFragment`(FAB 클릭) → `ToggleFavoriteUseCase.invoke()` → `FavoriteRepository.toggleFavorite()` → `FavoriteRepositoryImpl` → Room `FavoriteMovieDao.toggleFavorite()`(`@Transaction`으로 원자적 처리) — **정확한 흐름을 재구성했다.**

### Q2. "TMDB API 인증은 어떻게 되어 있어?" — 스코프 밖 발견

RAG가 정확히 찾아낸 건 **TMDB "계정 로그인" 인증**(`TmdbAuthApiService`, v4 request_token 발급 등)이었다. 앱 전체에 붙는 API 키 헤더 인터셉터는 인덱싱 범위(`domain/presentation/data`) 밖이라 못 찾았지만, **CLAUDE.md 요약에는 없던 기능을 RAG가 새로 발견**한 셈이다.

### Q3. "테스트 커버리지가 낮은 Repository가 어디야?" — 정직한 한계 인정 (가장 중요한 결과)

> **이 질문은 이 도구가 원리적으로 답할 수 없다.** 인덱스엔 소스 코드 텍스트만 있고 JaCoCo 커버리지 수치는 없어서, "Repository"라는 단어와 어휘적으로만 겹치는 파일들이 나온 것뿐이다(유사도가 가장 높은 것도 우연). 커버리지 %를 정말 알려면 `./gradlew jacocoTestReport` 결과를 읽어야 한다.

> **왜 이게 중요한 결과인가:** RAG 시스템의 흔한 실패 패턴은 "모르는 것도 그럴듯하게 답하는 것"이다. 여기선 검색된 청크와 실제 질문의 의미적 간극을 인지하고 **"이 데이터로는 답할 수 없다"고 정직하게 인정**했다 — 이게 실제 프로덕션 RAG 설계에서 가장 중요하게 다뤄야 할 지점이다.

---

## 성능 실측 — 직관과 다른 병목 발견

**측정 결과:** 질문당 총 233~259ms인데, 그중 **Room 전체 로드가 214~237ms로 대부분**이고 실제 코사인 계산은 10~13ms뿐이었다.

> **예상:** "벡터 연산이 느릴 것" → **실측:** "SQLite BLOB을 매번 전부 역직렬화하는 게 진짜 병목"

이건 072일차("쪼개면 느릴 것"이라는 직관이 실측으로 뒤집힌 사례)와 같은 패턴이다 — **최적화 전에는 반드시 실측**해야 한다는 교훈이 이번 5차 시리즈 전체에서 반복됐다.

---

## 확장성 분석 — 파일 수천 개로 커지면?

Brute-force 방식은 청크 수 N에 정확히 **선형(O(N))**이다.

| 규모 | 예상 Room 로드 시간 | 예상 총 시간 |
|---|---|---|
| 현재 (240파일, 1,224청크) | 214ms | 233~259ms |
| 20배 (5,000파일, ~25,500청크) | 4.3초 | 4초 이상 |

**대안 (파일 수천 개 문턱을 넘을 때만 고려):**
- `sqlite-vec` — SQLite 확장으로 ANN 인덱스를 SQLite 안에 직접 추가 (지금 구조를 가장 적게 바꿈)
- 별도 벡터 스토어(FAISS, pgvector, Milvus)로 이전
- BM25/역색인으로 후보군을 먼저 좁힌 뒤 그 안에서만 코사인 계산 (하이브리드 검색)

> **지금 규모에선 이 트레이드오프를 신경 쓸 필요가 없다.** ANN 인덱스를 지금 도입하면 오히려 구축/업데이트 복잡도만 추가되는 손해다.

---

## 오늘 배운 것 — 한 줄 정리

1. Room 2.7+의 KMP 지원으로, **Android 없이 순수 JVM에서도 Room을 그대로 쓸 수 있다.**
2. `EmbeddingProvider`처럼 **인터페이스로 추상화**해두면, 지금은 무료 로컬 구현으로 시작하고 나중에 실제 API로 1줄 교체할 수 있다.
3. 정규식 토큰화에서 **ASCII만 가정하면 비영어권 텍스트가 에러 없이 조용히 사라진다** — `\p{L}`, `\p{N}`을 기본값으로.
4. 이 프로젝트처럼 **한글 주석이 코드 스타일의 일부**라면, RAG 청킹 로직도 그 스타일을 반영해야 한다 (선언부 위 주석 포함).
5. **"모른다"고 정직하게 답하는 것도 RAG 설계의 핵심 기능이다** — Q3의 자기 한계 인정이 가장 가치 있는 결과였다.
6. 성능 최적화 전에는 **반드시 실측**해야 한다 — 직관(벡터 연산이 무거울 것)과 실측(BLOB 역직렬화가 병목)이 다를 수 있다.
7. Brute-force는 지금 규모에선 충분하지만 O(N)이라 **선형으로 나빠진다** — ANN은 "문턱을 넘을 때"만 도입하면 된다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-07-16*
