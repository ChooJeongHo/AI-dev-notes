# Claude Code Skills — 완전 분석 가이드

> Claude.ai의 Skills 시스템이 무엇인지, 왜 유용한지, 어떻게 만드는지에 대한 완전 분석  
> 실제 `/mnt/skills/public/` 내부를 직접 탐색하여 작성

---

## 📌 목차

1. [스킬이란?](#1-스킬이란)
2. [다른 AI의 스킬과의 차이](#2-다른-ai의-스킬과의-차이)
3. [Anthropic 공식 제공 스킬 목록](#3-anthropic-공식-제공-스킬-목록)
4. [스킬의 장점](#4-스킬의-장점)
5. [스킬 내부 구조 해부](#5-스킬-내부-구조-해부)
6. [스킬 만드는 법 — 단계별 가이드](#6-스킬-만드는-법--단계별-가이드)
7. [직접 만든 예시 스킬 — Android 코드리뷰](#7-직접-만든-예시-스킬--android-코드리뷰)
8. [공식 스킬 활용 예시 (pptx 스킬)](#8-공식-스킬-활용-예시-pptx-스킬)

---

## 1. 스킬이란?

Claude Code Skills는 **Claude가 특정 작업을 수행할 때 먼저 읽는 마크다운 지침서 파일**이다.

```
일반 Claude:
  "이 코드 리뷰해줘" → 일반적인 수준의 리뷰

Skills가 있는 Claude:
  "이 코드 리뷰해줘"
    → android-code-review/SKILL.md 파일을 view 도구로 읽음
    → Clean Architecture 위반 여부, 코루틴 패턴, 메모리 누수 등
      정해진 체크리스트에 따라 체계적으로 리뷰
    → 일관된 전문가 수준의 결과물 생성
```

**핵심 원리:**
- 스킬 = `SKILL.md` 마크다운 파일
- Claude가 사용자 요청에서 특정 키워드를 감지하면 해당 파일을 자동으로 읽음
- 파일 안의 지침을 그 자리에서 습득하여 작업 수행
- 코드 실행이 아닌 **"지식 주입"** 방식

---

## 2. 다른 AI의 스킬과의 차이

| 구분 | 다른 AI의 스킬 (Copilot, Alexa 등) | Claude Skills |
|------|--------------------------------------|----------------|
| **형태** | API 함수 / 플러그인 / 외부 서비스 | 마크다운 텍스트 파일 |
| **동작 방식** | 코드를 실행하거나 외부 서버 호출 | Claude가 파일을 읽고 지침을 따름 |
| **만드는 방법** | 코딩 필수 (API 명세 작성) | 마크다운 작성만으로 가능 |
| **저장 위치** | 외부 서버 / 클라우드 | `/mnt/skills/` 로컬 파일 시스템 |
| **확장성** | 개발자만 만들 수 있음 | 누구든 텍스트 파일만 만들면 됨 |
| **AI 역할** | 스킬을 호출하는 주체 | 스킬을 읽고 스스로 실행하는 주체 |

> **비유:** 다른 AI의 스킬이 "자동화 버튼"이라면, Claude Skills는 "전문가 매뉴얼"이다.

---

## 3. Anthropic 공식 제공 스킬 목록

실제 `/mnt/skills/` 디렉토리 구조:

```
/mnt/skills/
├── public/                        ← Anthropic 공식 스킬 (항상 사용 가능)
│   ├── pptx/SKILL.md              → PowerPoint 생성/편집
│   ├── docx/SKILL.md              → Word 문서 생성/편집
│   ├── xlsx/SKILL.md              → Excel 스프레드시트 생성/편집
│   ├── pdf/SKILL.md               → PDF 생성, 병합, OCR
│   ├── frontend-design/SKILL.md   → 고품질 UI/UX 컴포넌트
│   └── product-self-knowledge/    → Anthropic 제품 정보 정확성
│
├── examples/                      ← 참고/예시 스킬
│   ├── skill-creator/             → 새 스킬 설계·테스트 메타스킬
│   ├── mcp-builder/               → MCP 서버 구축 가이드
│   ├── slack-gif-creator/         → Slack GIF 생성
│   ├── algorithmic-art/           → 알고리즘 기반 예술 생성
│   ├── canvas-design/             → Canvas API 디자인
│   ├── brand-guidelines/          → Anthropic 브랜드 스타일
│   └── web-artifacts-builder/     → 웹 아티팩트 생성
│
└── user/                          ← 사용자 커스텀 스킬 추가 위치
```

### 공식 스킬 상세

| 스킬 | 트리거 키워드 | 핵심 기능 |
|------|-------------|---------|
| **pptx** | "PPT", "슬라이드", "발표자료", "deck" | 디자인 팔레트, 레이아웃, QA 포함 |
| **docx** | "Word", "워드", ".docx", "보고서" | XML 언팩 방식, 트랙 변경 지원 |
| **xlsx** | "Excel", "엑셀", "스프레드시트" | 수식, 차트, 데이터 정제 |
| **pdf** | "PDF", "폼 작성", "병합" | 생성/병합/OCR/암호화 |
| **frontend-design** | "UI", "컴포넌트", "웹 디자인" | 고품질 프로덕션 수준 UI |
| **product-self-knowledge** | "Claude 기능", "요금", "API" | 제품 정보 정확성 보장 |

---

## 4. 스킬의 장점

### ① 일관성 보장

스킬이 없으면 Claude는 매번 다른 방식으로 리뷰한다.  
스킬이 있으면 항상 동일한 체크리스트를 빠짐없이 확인한다.

```
# 코드리뷰 스킬이 강제하는 표준
1. 아키텍처 규칙 위반 여부 (레이어 간 의존성)
2. 코루틴 패턴 (CancellationException rethrow 여부)
3. 메모리 누수 (onDestroyView에서 binding null 처리)
4. 에러 처리 누락
5. 테스트 가능성
```

### ② 전문 지식의 캡슐화

도메인 전문가의 노하우를 파일에 담아두면,  
Claude가 매번 새로 학습하지 않아도 전문가 수준으로 작업한다.

### ③ 3단계 점진적 로딩 (메모리 효율)

```
레벨 1: 메타데이터 (name + description)
        → 항상 메모리에 있음 (~100 단어)
        → 트리거 여부 판단에 사용

레벨 2: SKILL.md 본문
        → 스킬이 트리거될 때만 로딩 (500줄 권장)
        → 핵심 절차와 규칙

레벨 3: 번들 리소스 (scripts/, references/, assets/)
        → 필요할 때만 로딩 (무제한)
        → 대용량 참조 문서, 실행 스크립트
```

### ④ 코딩 없이 확장 가능

마크다운만 쓸 줄 알면 Claude에게 새로운 능력을 추가할 수 있다. 개발자가 아니어도 된다.

### ⑤ 테스트·개선 사이클 지원

`skill-creator` 예시 스킬을 통해 테스트 케이스를 작성하고,  
스킬 있을 때 vs 없을 때 결과를 비교하며 반복 개선할 수 있다.

---

## 5. 스킬 내부 구조 해부

### SKILL.md 파일 구조

```markdown
---
name: skill-name                     ← 스킬 식별자
description: "언제 이 스킬을 쓸지,   ← 트리거 메커니즘 (가장 중요!)
  무엇을 하는지 상세히 기술"
license: 라이선스 정보 (선택)
---

# 스킬 제목

## 개요
## Quick Reference 테이블
## 상세 절차
## 참조 파일
## QA 체크리스트
```

### description 필드가 핵심인 이유

```yaml
# 나쁜 예 — 너무 좁아서 필요할 때 안 트리거됨
description: "코드 리뷰"

# 좋은 예 — 다양한 상황에서 정확하게 트리거됨
description: "Use this skill whenever the user asks to review Android/Kotlin code.
  Triggers: '코드 리뷰', '코드 봐줘', 'PR 리뷰', 'ViewModel 봐줘', 'Repository 패턴'"
```

### 번들 리소스 구조

```
my-skill/
├── SKILL.md          ← 필수 (진입점)
├── references/       ← 대용량 참조 문서
├── scripts/          ← 재사용 유틸리티 스크립트
└── assets/           ← 템플릿, 아이콘, 폰트
```

---

## 6. 스킬 만드는 법 — 단계별 가이드

### Step 1: 의도 정의

```
1. 이 스킬은 무엇을 가능하게 하는가?
2. 언제 트리거되어야 하는가?
3. 예상 출력 형식은 무엇인가?
```

### Step 2: SKILL.md 작성

```yaml
---
name: my-skill
description: >
  [무엇을 하는지]
  [언제 사용할지 - 구체적인 트리거 표현과 키워드 포함]
---
```

**본문 작성 원칙:**

```
✅ 명령형 문체 ("확인하라", "사용하라")
✅ 왜 중요한지 이유 설명 포함
✅ 구체적인 예시 포함
✅ 흔한 실수 목록 포함
✅ QA 체크리스트 포함
✅ 500줄 이내 (넘으면 references/ 파일로 분리)
```

### Step 3: 디렉토리 배치

```bash
/mnt/skills/user/my-skill/SKILL.md
# Claude가 자동으로 description을 읽고 트리거 여부 판단
```

### Step 4: 테스트

1. 테스트 프롬프트 2~3개 작성
2. 스킬 있을 때 vs 없을 때 결과 비교
3. SKILL.md 개선 → 반복

---

## 7. 직접 만든 예시 스킬 — Android 코드리뷰

실제 프로젝트 [MovieFinder](https://github.com/ChooJeongHo/MovieFinder)의 코드를 리뷰하면서 만든 스킬이다.  
MovieFinder는 TMDB API를 활용한 Android 앱으로, Clean Architecture + MVVM + Hilt + Paging3 등 실무 수준의 기술 스택을 사용한다.

---

### `examples/android-code-review/SKILL.md`

```markdown
---
name: android-code-review
description: >
  Use this skill whenever the user asks to review Android/Kotlin code,
  check architecture, find bugs, or improve code quality.
  Triggers: '코드 리뷰', '코드 봐줘', 'PR 리뷰', '이 코드 어때',
  'ViewModel 봐줘', 'Repository 패턴', 'Coroutine 확인', 'Clean Architecture'
---

# Android Kotlin 코드리뷰 스킬

## 개요

Clean Architecture + MVVM 기반 Android 프로젝트를 리뷰할 때 사용한다.
일관된 체크리스트로 아키텍처 위반, 메모리 누수, 코루틴 패턴 오류를 빠짐없이 확인한다.

---

## 리뷰 체크리스트

### 1. 아키텍처 규칙
- Presentation → Domain (O) / Presentation → Data (X)
- Domain은 순수 Kotlin — Android 프레임워크 import 금지
- ViewModel이 Repository를 직접 참조하면 UseCase 레이어 누락

### 2. 코루틴 패턴
- CancellationException은 반드시 rethrow하라
- viewModelScope + repeatOnLifecycle 사용 여부 확인
- StateFlow 초기값 설정 누락 확인

### 3. 메모리 누수
- onDestroyView()에서 _binding = null 처리 여부
- RecyclerView의 onViewRecycled()에서 이미지 dispose() 호출 여부
- ShimmerFrameLayout의 stopShimmer() 호출 여부

### 4. 에러 처리
- API 에러를 UI에 전달하는 방식 (Channel/SharedFlow 권장)
- 부분 실패 처리: 여러 API 병렬 호출 시 일부 실패해도 동작해야 함
- try-catch에서 CancellationException rethrow 확인

### 5. 동시성
- 중복 API 호출 방지 (Mutex.tryLock() 패턴)
- FAB 연타 스태킹 방지 (animate().cancel())
- @Transaction으로 DB 원자적 처리 여부

---

## 리뷰 출력 형식

항상 아래 형식으로 출력하라:

### 🔴 심각 (즉시 수정 필요)
버그, 크래시 가능성, 보안 문제

### 🟡 개선 권장
아키텍처 위반, 성능 저하 가능성

### 🟢 잘된 점
좋은 패턴, 잘 구현된 부분

### 💡 제안
선택적 개선 사항

---

## QA 체크리스트
- [ ] 레이어 의존성 방향이 올바른가?
- [ ] CancellationException rethrow 누락 없는가?
- [ ] 메모리 누수 가능성 없는가?
- [ ] 에러 처리가 빠진 코드 경로 없는가?
- [ ] 테스트 가능한 구조인가?
```

---

### 실제 MovieFinder 코드에 적용한 리뷰 사례

#### 사례 1 — DetailViewModel 중복 호출 방지 🟢 잘된 점

```kotlin
private val loadingMutex = Mutex()

fun loadMovieDetail(movieId: Int) {
    if (!loadingMutex.tryLock()) return  // 중복 호출 차단
    viewModelScope.launch {
        try {
            // API 호출
        } finally {
            loadingMutex.unlock()  // 에러 후 재시도 가능하도록 반드시 해제
        }
    }
}
```

`Mutex.tryLock()`으로 중복 API 호출을 원자적으로 차단하면서,  
`finally`로 항상 잠금 해제 → 에러 후 재시도도 가능한 패턴. ✅

---

#### 사례 2 — 6개 API 병렬 호출 + 부분 실패 처리 🟢 잘된 점

```kotlin
coroutineScope {
    val detailDeferred = async { getMovieDetailUseCase(movieId) }
    val creditsDeferred = async { getCreditsUseCase(movieId) }
    val similarDeferred = async { getSimilarMoviesUseCase(movieId) }

    val detail = detailDeferred.await()  // 필수 — 실패 시 전파
    val credits = runCatching { creditsDeferred.await() }.getOrDefault(emptyList())  // 선택
    val similar = runCatching { similarDeferred.await() }.getOrDefault(emptyList())  // 선택
}
```

상세 정보는 필수(실패 시 전파), 출연진/비슷한 영화는 부분 실패 허용(`runCatching`) → 앱 크래시 방지. ✅

---

#### 사례 3 — ErrorType 분리로 Context 의존성 제거 🟢 잘된 점

```kotlin
// ViewModel: Android Context 없이 ErrorType만 보유
sealed class ErrorType { object Network; object Timeout; object Server }

// Fragment: Context가 있는 View 레이어에서 메시지 변환
val message = ErrorMessageProvider.getMessage(context, errorType)
```

ViewModel이 Context에 의존하지 않아 단위 테스트가 쉬워진다.  
MockK로 Context 없이도 ViewModel 테스트 145개 작성 가능한 이유. ✅

---

#### 사례 4 — CancellationException rethrow 🟢 잘된 점

```kotlin
fun setThemeMode(mode: ThemeMode) {
    viewModelScope.launch {
        try {
            setThemeModeUseCase(mode)
        } catch (e: CancellationException) {
            throw e  // 반드시 rethrow — 코루틴 취소 정상 동작 보장
        } catch (e: Exception) {
            Timber.e(e)
        }
    }
}
```

`CancellationException`을 catch 후 무시하면 코루틴 취소가 동작하지 않아  
메모리 누수와 예측 불가능한 동작이 발생한다. 올바르게 rethrow. ✅

---

### 스킬 없이 리뷰했을 때 vs 스킬 있을 때

| | 스킬 없음 | 스킬 있음 |
|---|---|---|
| **체크 항목** | 눈에 띄는 것만 | 5개 카테고리 빠짐없이 |
| **일관성** | 매번 다름 | 항상 동일한 기준 |
| **출력 형식** | 자유 형식 | 🔴/🟡/🟢/💡 구조화 |
| **누락 가능성** | 높음 | 체크리스트로 최소화 |

---

## 8. 공식 스킬 활용 예시 (pptx 스킬)

### 디자인 색상 팔레트

| 테마 | Primary | Secondary | Accent |
|------|---------|-----------|--------|
| Midnight Executive | `#1E2761` navy | `#CADCFC` ice blue | `#FFFFFF` |
| Coral Energy | `#F96167` coral | `#F9E795` gold | `#2F3C7E` |
| Warm Terracotta | `#B85042` | `#E7E8D1` sand | `#A7BEAE` |

### QA 절차

```bash
python -m markitdown output.pptx
python scripts/office/soffice.py --headless --convert-to pdf output.pptx
pdftoppm -jpeg -r 150 output.pdf slide
```

### 스킬이 금지하는 것들

```
❌ 타이틀 아래 장식선 (AI 생성물처럼 보임)
❌ 텍스트만 있는 슬라이드 (시각 요소 필수)
❌ 모든 색상에 동일한 비중 (주색 60-70% 원칙)
```

---

## 요약

```
스킬   = 마크다운 지침서 파일 (SKILL.md)
트리거 = description 필드의 키워드 매칭
로딩   = Claude가 view 도구로 SKILL.md를 읽음
실행   = 지침에 따라 작업 수행

장점:
  - 일관성: 항상 동일한 표준 절차
  - 전문성: 도메인 노하우 캡슐화
  - 효율성: 3단계 점진적 로딩
  - 접근성: 마크다운만 쓸 줄 알면 누구든 만들 수 있음
```

---

*분석 기준: `/mnt/skills/public/` 및 `/mnt/skills/examples/` 직접 탐색*  
*예시 프로젝트: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-03-10*
