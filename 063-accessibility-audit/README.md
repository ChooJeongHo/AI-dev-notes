# 063 - Claude Code로 접근성 자동 감사

**35개 XML + 프레젠테이션 레이어 전체 분석 — MAJOR 4건 + MINOR 5건 발견, 색상 대비 WCAG 기준 자동 검증**

---

## 목적

032일차 스크린샷 기반 접근성 점검의 코드 레벨 심화 버전.  
XML 레이아웃, Kotlin 코드, 색상 리소스를 정적 분석으로 접근성 이슈 자동 탐지.

---

## 감사 범위

| 항목 | 내용 |
|------|------|
| 대상 | app/src/main/res/layout/ (35개 XML), 프레젠테이션 레이어 전체 |
| 기준 | WCAG 2.1 AA, Android Accessibility Guidelines, Material Design Accessibility |
| 분석 방법 | 정적 코드 분석 (병렬 실행) |
| 결과물 | ACCESSIBILITY_REPORT.md (306줄) |

---

## MAJOR 이슈 4건

| # | 이슈 | 위치 | 기준 |
|---|------|------|------|
| M-1 | `colorSecondary` (#01B4E4) 흰 배경 위 대비 **2.43:1** (기준 4.5:1) | values/themes.xml | WCAG AA |
| M-2 | 다크모드 `colorPrimaryDark` (#90CEA1) 위 흰 텍스트 **1.82:1** | values-night/colors.xml | WCAG AA |
| M-3 | `announceForAccessibility` / `LiveRegion` 코드베이스 전체 **0건** — 에러 표시, FAB 토글, 검색 완료 시 TalkBack 미발화 | DetailFragment, SearchFragment 등 | Android TalkBack |
| M-4 | `alpha="0.5"` 일반 텍스트 대비 **~3.95:1** (기준 4.5:1) | item_person_search.xml:56 | WCAG AA |

---

## MINOR 이슈 5건

- CircularRatingView 터치 타겟 40dp (기준 48dp)
- Mini FAB 터치 타겟 40dp
- `rating_red` 텍스트 색상 대비 부족
- 커스텀 차트 뷰 `AccessibilityDelegate` 미구현
- Settings 항목 역할(Role) 미지정

---

## 색상 대비 자동 계산 결과

| 색상 쌍 | 대비 | WCAG |
|--------|------|------|
| colorPrimary 텍스트 on 흰 배경 | 15.62:1 | ✅ PASS |
| colorSecondary on 흰 배경 | **2.43:1** | ❌ FAIL |
| 다크모드 primaryDark on 흰 | **1.82:1** | ❌ FAIL |
| 일반 텍스트 (alpha 0.5) | ~3.95:1 | ❌ FAIL |

---

## 잘 구현된 사항 ✅

- 어댑터 8종 모두 루트 `contentDescription` 동적 설정
- shimmer `noHideDescendants` 적용
- FAB 상태별 `contentDescription` 업데이트
- 장식용 이미지 전체 `importantForAccessibility="no"` 처리
- HistogramView, BarChartView 등 커스텀 뷰 `contentDescription` 동적 생성
- 공유 액션 요소 전환 이름(`TransitionName`) 설정

---

## 032일차 vs 063일차 비교

| 항목 | 032일차 (스크린샷 기반) | 063일차 (코드 레벨) |
|------|:--------------------:|:-----------------:|
| 분석 방법 | 이미지 시각적 분석 | **정적 코드 분석** |
| 색상 대비 계산 | 시각적 추정 | **WCAG 수치 자동 계산** |
| 발견 이슈 수 | 시각적 이슈 중심 | **MAJOR 4건 + MINOR 5건** |
| 결과물 | 분석 코멘트 | **306줄 감사 리포트** |

---

## 가장 임팩트 큰 수정 — M-3 LiveRegion

FAB 토글 후 한 줄 추가만으로 TalkBack 사용자 경험이 크게 개선된다:

```kotlin
// FAB 상태 변경 시
binding.fabFavorite.announceForAccessibility(newDescription)
```

---

## 느낀 점

접근성은 "나중에 하면 되는 것"으로 미루기 쉬운데, 코드 레벨에서 자동으로 감사하니까 숫자로 보인다.

colorSecondary 대비 2.43:1은 눈으로는 괜찮아 보여도 WCAG 기준(4.5:1)의 절반 수준이다. 시각 장애가 있는 사용자에겐 실제로 읽기 어려운 색상이다.

**"접근성은 배려가 아니라 품질이다" — AI가 수치로 증명해준다.**

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-06-24*
