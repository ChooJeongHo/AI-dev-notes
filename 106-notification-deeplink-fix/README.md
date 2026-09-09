# 106 - 알림 딥링크 버그 수정 — 101/103/104일차와는 아예 다른 계열의 원인이었다

**같은 "딥링크가 이상하게 동작한다"는 증상이었지만, 이번엔 Navigation API 오용이 아니라 implicit Intent와 매니페스트 intent-filter 간 URI path 불일치였다 — 그리고 검증 중 104일차 테스트가 한 번도 실행된 적 없었다는 사실까지 드러났다**

---

## 목적

104일차에서 "범위 밖"으로 메모리에만 기록해두고 넘겼던 WatchGoalNotificationHelper의 stats 알림 딥링크 버그를 처리한다.

---

## 작업 1 — 원인 재확인: 101/103/104일차와 완전히 다른 계열이었다

101/103/104일차는 전부 MainActivity의 Navigation 백스택 조작(popBackStack/popUpTo) API 오용 계열이었다. 하지만 이번 버그는 implicit Intent와 매니페스트 intent-filter 간 URI path 불일치였다.

```
moviefinder://stats
    ↓
path 세그먼트가 없어서 실제 path가 "빈 문자열"
    ↓
Navigation Safe Args가 생성한 intent-filter는
<data android:path="/" /> (정확 일치)를 요구
    ↓
PackageManager가 매칭 실패 → "Activity not started"
```

왜 stats 알림만 유일하게 걸렸는지:

| 딥링크 대상 | intent-filter 방식 | 걸렸나 |
|---|---|---|
| movie/person | pathPrefix="/" (접두어 일치) | 안 걸림 |
| search/favorite 정적 단축키 | targetClass 명시로 매칭 자체를 우회 | 안 걸림 |
| stats 알림 | implicit intent가 정확 일치 요구 path와 안 맞음 | 걸림 |

---

## 작업 2 — 수정

WatchGoalNotificationHelper.kt에서 shortcuts.xml과 동일하게 MainActivity를 explicit component로 지정하도록 buildStatsDeepLinkIntent()로 분리했다. nav_graph/MainActivity의 화면 이동 로직 자체는 건드리지 않았다 — 원인이 인텐트 매칭 단계에 있었으므로, 그 지점만 정확히 고쳤다.

---

## 작업 3 — 테스트

WatchGoalNotificationHelperTest.kt에 setClass 호출을 검증하는 테스트를 추가했다. 103일차의 AppShortcutManagerTest와 동일한 mockkConstructor 패턴을 재사용했다. 전체 유닛 테스트 + detekt 통과.

---

## 작업 4 — 실기기 검증에서 드러난 예상 밖 발견

### 버그 재현과 수정 확인

원래 버그(implicit intent, "Activity not started")를 먼저 재현 확인 → 실제로 시청 목표 알림을 발생시켜 탭 → 통계 화면 정상 진입 확인. 104일차 딥링크 4종(movie/stats/search/favorite) 재검증 결과 회귀 없음.

### 더 중요한 발견 — 104일차 계측 테스트가 한 번도 실행된 적 없었다

104일차에 만든 MainActivityDeepLinkBackStackTest를 실기기에서 처음 실행해보니, 4개 전부 NoActivityResumedException으로 실패했다.

원인은 이번 수정과 무관한 기존 결함이었다 — Espresso의 pressBack()이 "앱 완전 종료"를 예외로 취급하는데, 이게 정확히 이 테스트가 검증하려던 상황(뒤로가기 시 앱 종료)과 충돌하는 하네스 자체의 버그였다. 104일차 이후 이 계측 테스트는 한 번도 실기기에서 실행된 적이 없었다 — 유닛 테스트/컴파일은 통과했지만, 실제 Espresso 계측 테스트로는 처음부터 실행 불가능한 상태였던 것.

**서브에이전트의 보고를 그대로 믿지 않고 직접 근거를 확인:** 실기기 검증을 맡긴 서브에이전트가 "실패 원인이 기존 버그"라고 보고했을 때, 그걸 그대로 받아들이지 않고 실제 HTML 테스트 리포트(`app/build/reports/androidTests/connected/debug/...html`)를 직접 열어서 확인했다. 스택트레이스에 찍힌 `NoActivityResumedException: Pressed back and killed the app`이라는 메시지 자체가 서브에이전트의 주장을 뒷받침하는 증거였다 — "Pressed back and killed the app"은 테스트가 원래 검증하려던 상황(뒤로가기 시 앱 완전 종료)이 정상적으로 일어났다는 뜻인데, Espresso의 `pressBack()`이 남은 activity가 없는 이 상태를 "실패"로 취급해서 예외를 던지는 특성 때문이었다.

> 080일차(judge)에서 확립한 "AI의 보고를 그대로 믿지 말고 근거를 직접 재확인한다"는 원칙이 여기서도 그대로 적용됐다 — 서브에이전트의 결론이 맞았지만, 그 맞음을 실제 증거 파일로 재확인한 절차 자체가 신뢰도를 한 단계 더 높인다.

이건 이번 작업 범위 밖이라 고치지 않고 기록만 남겼다.

### 추가 — 같은 세션에서 바로 해소

범위 밖이라고 남겨뒀던 이 Espresso 하네스 버그를, 같은 세션에서 바로 처리했다. `pressBackExpectingAppExit()` 헬퍼를 추가해서 `pressBack()`을 `try/catch(NoActivityResumedException)`로 감싸도록 했다 — 앱이 완전히 종료되어 resumed activity가 없어질 때 Espresso가 던지는 이 예외를 흡수하고, 실제 종료 여부는 기존 그대로 `scenario.state == DESTROYED` 단언으로 검증한다. **검증 로직 자체는 바뀌지 않았고, 잘못된 방식으로 예외를 던지던 하네스만 고쳤다.**

실기기(SM-S926N)에서 `connectedDebugAndroidTest`로 재실행한 결과 4개 테스트 전부 통과했다:
- `movieDeepLink_backPress_finishesInsteadOfReturningToOnboarding`
- `statsDeepLink_backPress_finishesInsteadOfReturningToOnboarding`
- `searchShortcutDeepLink_backPress_stillFinishes_regression`
- `favoriteShortcutDeepLink_backPress_stillFinishes_regression`

detekt도 통과했다. **104일차에 만들어진 이후 한 번도 실행되지 못했던 이 테스트가, 106일차에 처음으로 실기기에서 통과하는 걸 확인**했다.

---

## 작업 5 — 정리

메모리에 기록 완료 (project_watchgoal_stats_notification_deeplink_fix.md, 기존 project_deeplink_onboarding_backstack_fix.md와 상호 링크, MEMORY.md 인덱스 갱신).

---

## 101~106을 관통하는 "실기기 검증 계열 문제"의 다섯 번째 변주

| 일차 | 원인 계열 |
|---|---|
| 101 | XML 리소스 타입 (res/anim vs res/animator) |
| 102 | 원인 없음 (이미 정상) |
| 103 | Navigation API(popBackStack) 오용 |
| 104 | 103일차 화이트리스트 함수의 커버리지 누락 |
| 106 | implicit Intent ↔ intent-filter URI path 불일치 (Navigation API와 무관) |

"딥링크/네비게이션이 이상하다"는 증상이 다섯 번 나왔는데, 실제 원인은 매번 완전히 다른 계층(리소스 타입, 정상, API 오용, 커버리지 누락, 인텐트 매칭)이었다. 이게 105일차 포트폴리오에서 정리한 "같은 결과도 원인은 매번 다르다"는 원칙의 다섯 번째 사례다.

---

## 부수 발견의 가치 — 검증 도구 자체의 신뢰성도 검증해야 한다

104일차에서 만든 테스트가 작성된 이후 한 번도 실행되지 않은 채로 방치되어 있었다는 게, 오늘 우연히 다른 버그를 고치려다 드러났다. 이건 073/087/088일차의 "테스트가 있다고 검증됐다는 보장은 없다"는 원칙의 또 다른 변주다 — 테스트가 존재하는 것과 그 테스트가 실제로 실행되어 통과한 것은 다른 문제다.

---

## 오늘 배운 것 — 한 줄 정리

1. 같은 "딥링크가 이상하다"는 증상도, Navigation API 문제(103/104)와 Intent 매칭 문제(106)는 완전히 다른 계층이다 — 매번 원인부터 정확히 재확인해야 한다.
2. implicit Intent는 매니페스트 intent-filter의 path 매칭 규칙(정확 일치 vs 접두어 일치)에 따라 조용히 실패할 수 있다.
3. explicit component 지정은 이런 매칭 문제를 원천적으로 피하는 확실한 방법이다.
4. 테스트가 작성됐다는 것과 그 테스트가 실제로 실행되어 검증됐다는 것은 다른 문제다 — 실기기 계측 테스트는 특히 "작성 후 방치"되기 쉽다.
5. 작업 범위 밖에서 우연히 발견한 문제는, 범위를 지키며 기록만 남기고 다음 작업으로 넘기는 것이 맞는 판단이다.
6. 서브에이전트의 보고가 맞아 보여도, **실제 증거(테스트 리포트, 로그 등)를 직접 열어서 재확인**하면 신뢰도가 한 단계 더 올라간다 — 080일차 judge 원칙이 실기기 검증 과정에서도 그대로 재현됐다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*
*작성일: 2026-09-09*
