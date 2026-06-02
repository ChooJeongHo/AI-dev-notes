# 048 - Claude Code Plan Mode 실험

**실행 전 계획서를 먼저 확인하고 승인 — Auto Mode와 비교해서 어떤 상황에 어떤 모드가 적합한지 결론 도출**

---

## 목적

047일차 Auto Mode 실험의 연장선.  
동일한 작업을 Plan Mode로 실행해서 두 모드의 차이를 수치와 경험으로 비교.

---

## Plan Mode란?

Claude가 코드베이스를 분석하고 계획서를 작성한 뒤, 실행 전에 사용자 승인을 기다리는 읽기 전용 단계.

**Shift+Tab 두 번** 눌러서 활성화.

---

## 실험 결과

**요청:**
```
MovieFinder 앱의 코드 품질 개선점 3가지를 찾아서 직접 수정하고 커밋해줘
```

**Plan Mode 흐름:**
1. 코드베이스 분석 (읽기만)
2. 계획서 출력 (3개 파일, 변경 내용, 검증 방법까지 포함)
3. 사용자 승인 (1회)
4. 실행 → 컴파일 → 커밋

**총 소요 시간: 4분 41초**

---

## 자동 수정된 내용

| 파일 | 개선 내용 | 유형 |
|------|---------|------|
| SettingsViewModel.kt | `disconnectTmdb()` snackbar 누락 버그 수정 + `launchWithErrorHandler` 적용 | 버그 수정 |
| DetailViewModel.kt | `submitTmdbRating()` CancellationException 처리 프로젝트 표준 패턴으로 통일 | 패턴 통일 |
| RateLimiter.kt | `now` 루프 내 재계산 + `MAX_CAS_RETRIES = 5` 방어 한도 추가 | 방어 코드 |

---

## Auto Mode vs Plan Mode 비교

| 항목 | Auto Mode (047) | Plan Mode (048) |
|------|----------------|----------------|
| 소요 시간 | 6분 29초 | **4분 41초** |
| 계획 확인 | 없음 | 실행 전 전체 계획 확인 ✅ |
| 사용자 개입 | 0번 | 계획 승인 1번 |
| Detekt 오류 | 발생 후 자동 수정 | **없음** (계획 단계에서 미리 고려) |
| 예측 가능성 | 낮음 | **높음** |

**Plan Mode가 더 빠르고 Detekt 오류도 없었다** — 실행 전 계획 단계에서 이미 Detekt 규칙을 고려했기 때문.

---

## Claude Code 4가지 모드 정리

| 모드 | 표시 | 동작 |
|------|------|------|
| Default | 없음 | 매번 수동 승인 |
| Accept Edits | `accept edits on` | 파일 수정 자동, bash는 승인 필요 |
| Auto | `auto mode on` | 파일+bash 모두 AI 자동 판단 |
| Plan | `plan mode on` | 계획서 먼저, 승인 후 실행 |

---

## 모드별 추천 상황

| 상황 | 추천 모드 |
|------|----------|
| 복잡한 멀티파일 리팩토링 | **Plan** |
| 장시간 자율 작업 | **Auto** |
| 단순 파일 수정 | **Accept Edits** |
| 중요한 코드 변경 (실수 방지) | **Plan** |

---

## ★ Insight (Claude Code 발언)

> "disconnectTmdb()의 val token = tmdbAccessToken.value ?: return을 코루틴 바깥으로 꺼낸 것이 핵심입니다 — StateFlow.value는 동기 프로퍼티이므로 suspend 컨텍스트 없이도 읽을 수 있고, 덕분에 launchWithErrorHandler의 return@lambda 제약을 우회할 수 있습니다."

---

## 느낀 점

Auto Mode가 편하지만 Plan Mode가 오히려 더 빠르고 안정적이었다.

계획 단계에서 Detekt 규칙, import 충돌 등을 미리 고려하기 때문에 실행 후 오류가 나지 않는다.

**복잡한 작업일수록 Plan Mode가 유리하다. "빠르게 가려면 먼저 계획하라."**

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-06-02*
