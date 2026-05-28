# 045 - Claude Code로 코드 기술 부채 자동 측정

**14,305줄 코드베이스를 자동 분석해서 기술 부채를 수치화 — 전체 점수 28/100 🟢 양호**

---

## 목적

코드리뷰로 발견된 이슈들이 전체 코드베이스 관점에서 얼마나 심각한지 수치로 파악.  
038~042일차 코드리뷰 시리즈의 마무리 — "발견 → 분류 → 등록 → 수치화" 완성.

---

## 측정 결과

**전체 부채 점수: 28 / 100 🟢 양호**

> 대상: `app/src/main/java/com/choo/moviefinder/` 전체 `.kt` 파일  
> 총 14,305 lines / 466 functions

| 발견 항목 | 건수 | 심각도 |
|---------|------|--------|
| God Class (500줄+) | 3개 Fragment | 🔴 높음 |
| TOCTOU race condition | 2건 (NOTE 주석) | 🟠 중간 |
| SharingStarted.Eagerly 잔존 | 1건 | 🔴 즉시 수정 |
| !! 위험 사용 | 2건 (SearchViewModel) | 🟠 중간 |
| launchWithErrorHandler 미통일 | 3개 ViewModel | 🟡 낮음 |
| @Suppress("TooManyFunctions") | 10개 파일 | 구조적 원인 |
| TODO/FIXME/HACK | **0건** | ✅ 우수 |

---

## 복잡도 높은 파일 TOP 3

| 순위 | 파일 | 라인 수 |
|------|------|--------|
| 1 | SearchFragment.kt | 785줄 |
| 2 | DetailFragment.kt | 688줄 |
| 3 | FavoriteFragment.kt | 500줄+ |

→ 모두 Presentation 레이어 Fragment — Domain/Data 레이어는 깨끗함

---

## 핵심 발견

### @Suppress가 가장 정직한 기술 부채 지표

```kotlin
@Suppress("TooManyFunctions") // 10개 파일
```

Detekt가 이미 문제를 감지했는데 개발자가 경고를 끈 것 → **도구가 "여기가 부채다"라고 표시한 위치 그대로**

### 부채가 레이어별로 비대칭

```
Domain 레이어  → 깨끗함 ✅
Data 레이어    → 깨끗함 ✅
Presentation   → God Class 집중 ⚠️
```

Android의 Fragment 라이프사이클이 뷰 로직을 한 곳에 모으게 만드는 **구조적 압력** 때문.  
Compose 전환이 근본 해결책이다.

### 즉시 수정 가능한 항목 2개

```kotlin
// 1. FavoriteViewModel:189 - 1줄 교체
SharingStarted.Eagerly → WhileSubscribed5s

// 2. SearchViewModel:171, 177 - !! 제거
result!!.data → result?.data ?: return
```

---

## 038~045일차 코드 품질 시리즈 종합

| 실험 | 핵심 결론 |
|------|---------|
| 038 CLAUDE.md 경량화 | 컨텍스트 26% 줄이니 리뷰 품질 8.2→8.6 |
| 039 모델별 비교 | Sonnet이 회귀 버그 탐지 최강 |
| 040 Claude vs Gemini | 버그 탐지 vs 아키텍처 개선 역할 분담 |
| 041 GEMINI.md 효과 | 컨텍스트 설정이 분석 깊이 결정 |
| 042 버그 분류 파이프라인 | 52건→36건 중복 제거, 9건 자동 등록 |
| 045 기술 부채 측정 | 28/100 양호, TODO 0건, Presentation 집중 |

**"AI로 코드 품질을 발견하고, 분류하고, 등록하고, 수치화하는" 풀 파이프라인 완성.**

---

## 느낀 점

코드 품질을 "느낌"이 아닌 **숫자**로 말할 수 있게 됐다.

"Fragment가 좀 크긴 한데..." 대신 "Presentation 레이어 God Class 3개, @Suppress 10개 파일, 부채 점수 28/100" 이라고 말할 수 있다.

특히 TODO/FIXME 0건은 그냥 나온 결과가 아니다 — 42일차 파이프라인으로 발견된 이슈들이 이미 GitHub Issues로 등록됐기 때문에 코드에 TODO로 남길 필요가 없었던 것이다.

**AI 도구로 코드 품질을 관리하는 사람이라는 스토리가 수치로 완성됐다.**

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-05-28*
