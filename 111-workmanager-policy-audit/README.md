# 111 - WorkManager ExistingWorkPolicy 전체 점검 — "KEEP은 나쁘고 REPLACE가 좋다"는 틀린 직관이었다

**110일차 버그를 계기로 프로젝트 전체(4곳)를 점검했더니, 새 버그는 0건이었다 — 대신 "같은 API니까 통일해야 한다"는 직관이 이번 사례에서 정확히 틀렸다는 걸 확인했다**

---

## 목적

110일차에서 발견한 ExistingWorkPolicy.KEEP 버그(사용자 액션에 무반응처럼 보이는 문제)가 프로젝트의 다른 WorkManager 사용처에도 있는지 전수 점검한다.

---

## 작업 1 — 전수 조사: Worker 3개, unique work 호출 4곳

| # | 위치 | work name | 요청 타입 | 정책 | 트리거 |
|---|---|---|---|---|---|
| 1 | ReleaseNotificationScheduler.kt:60 | release_$movieId | OneTime | KEEP | 워치리스트 추가 시 자동 |
| 2 | WatchlistReminderScheduler.kt:40-44 | watchlist_reminder_$movieId | OneTime | REPLACE | 사용자가 날짜/시간 직접 선택 |
| 3 | BoxOfficeWidgetWorker.kt:129-133 | box_office_widget_periodic | Periodic | KEEP | 앱/위젯 설치 시 재등록 housekeeping |
| 4 | BoxOfficeWidgetWorker.kt:149-153 | box_office_widget_refresh | OneTime | REPLACE | 새로고침/토글 (110일차에 이미 수정됨) |

---

## 작업 2 — 판단 기준 수립 (110일차 교훈의 일반화)

110일차 사례를 일반화해서, 4가지 질문으로 판단 기준을 세웠다.

1. 이 Worker가 실제로 백오프에 빠질 수 있는가? (Result.retry() 반환 + 네트워크 등 실패 가능성) — 로컬 알림만 발행하고 실패 케이스가 사실상 없다면 KEEP/REPLACE 논쟁 자체가 무의미하다.
2. 같은 unique work name을 재호출하는 트리거가 "지금 당장 다시 해달라"는 명시적 사용자 의도인가? (새로고침, 토글, 날짜 재선택) → REPLACE. 반대로 "조용히 반복되는 housekeeping 재등록"이라면 → KEEP.
3. 재호출 전에 항상 명시적 cancel()을 거치는 경로인가? 그렇다면 이전 job의 백오프 상태가 이미 지워졌으므로 KEEP이어도 안전하다. 위험한 건 "취소 없이 같은 이름으로 재호출되는" 경로뿐이다.
4. Periodic work는 원칙적으로 KEEP이어야 한다 — REPLACE는 재등록마다 주기를 리셋시켜 영원히 첫 실행이 안 될 위험이 있다(공식 문서 권고). "즉시 실행"이 필요하면 별도의 one-time unique work로 분리한다.

---

## 작업 3 — 실제 판단: 4곳 모두 정확했다, 새 버그 0건

| # | 판정 | 근거 |
|---|---|---|
| 1 (KEEP) | 정확 | ReleaseNotificationWorker는 네트워크 없이 로컬 알림만 발행해 질문1(백오프 시나리오 자체가 없음)에 해당. 재스케줄이 필요한 유일한 경로(워치리스트 제거→재추가)는 항상 cancel()을 먼저 거침(질문3 충족) |
| 2 (REPLACE) | 정확 | 사용자가 날짜 피커로 직접 시각을 재설정하면 즉시 반영돼야 함(질문2: 명시적 "다시 설정" 의도) |
| 3 (KEEP, periodic) | 정확 | 질문4 그대로 — periodic은 원칙적으로 KEEP이 맞음 |
| 4 (REPLACE) | 이미 110일차에 수정된 사례 | — |

프로덕션 코드는 변경하지 않았다. 대신 이 판단이 나중에 실수로 뒤집히지 않도록, 정책 자체를 검증하는 테스트가 없던 2곳(2번, 3/4번)에 회귀 테스트를 추가했다.

---

## 작업 4 — 테스트 작성

WatchlistReminderScheduler는 테스트 파일 자체가 없었고, BoxOfficeWidgetWorker의 enqueuePeriodic/enqueueOneTimeRefresh는 정책을 고정하는 테스트가 없었다.

- WatchlistReminderSchedulerTest.kt 신규 (5개 케이스)
- BoxOfficeWidgetWorkerSchedulingTest.kt 신규 (5개 케이스)

전체 866개(기존 856 + 신규 10) 유닛 테스트 전부 통과, detekt 클린.

---

## 작업 5 — 실기기 검증: "검증할 수정이 없다"는 결론

이번 조사에서는 프로덕션 코드를 전혀 수정하지 않았다 — 판정 결과 4곳 모두 이미 올바르게 설정돼 있었고, 추가한 건 회귀 테스트뿐이라 기기에서 관찰 가능한 동작 변화가 없었다. 그래서 110일차처럼 dumpsys jobscheduler로 대조 검증할 대상 자체가 없었다 — "검증할 수정이 없다"는 것 자체가 이번 검증의 결론이다.

---

## 작업 6 — 정리

| 항목 | 수치 |
|---|---|
| 패턴이 존재하는 곳 | 4곳 |
| 실제 버그였던 곳 | 1곳 (110일차에 이미 발견/수정 완료) |
| 이번 조사에서 새로 발견된 버그 | 0곳 |
| 추가한 회귀 방지 테스트 | 10개 |

---

## 핵심 통찰 — "같은 API니까 통일해야 한다"는 직관은 틀렸다

BoxOfficeWidgetWorker의 periodic 등록(KEEP)과 one-time 새로고침(REPLACE)은 같은 파일, 같은 클래스 안에서 정반대 정책을 쓴다.

> periodic이 KEEP인 이유(REPLACE면 주기가 리셋됨)와 one-time 새로고침이 REPLACE인 이유(KEEP이면 백오프 대기가 버튼 무반응으로 보임)는 정반대 근거로 정반대 정책을 쓴다 — "같은 API니까 통일해야 한다"는 직관은 이 경우 틀리다.

또한 ReleaseNotificationScheduler가 KEEP으로 안전한 이유는 정책 값 자체가 아니라 호출 경로 구조 덕분이었다.

> 재스케줄이 필요한 유일한 경로가 항상 cancel()을 먼저 거치기 때문에, KEEP이 막을 "취소 없이 재호출되는" 상황 자체가 코드상 존재하지 않는다. 이런 건 정책 값만 봐서는 안 보이고 호출부를 추적해야 드러난다.

---

## 110 → 111일차 발전 요약

```
110: 우연히 발견한 버그 1건 (BoxOfficeWidgetWorker one-time refresh, KEEP→REPLACE)
    ↓
111: 그 버그를 일반화한 4가지 판단 기준을 세워서 프로젝트 전체 재점검
     → 나머지 3곳은 전부 올바른 설정이었음을 확인 (새 버그 0건)
     → "정책은 통일이 아니라 각 상황의 근거로 판단해야 한다"는 원칙 확립
     → 판단 근거가 없던 곳에 회귀 테스트 10개 추가
```

---

## 오늘 배운 것 — 한 줄 정리

1. 같은 API(ExistingWorkPolicy)를 쓰는 곳이라도, "통일"이 아니라 각 상황의 성격(사용자 즉시 반응 요구 vs 조용한 housekeeping)에 따라 정반대 정책이 정답일 수 있다.
2. Periodic work는 원칙적으로 KEEP이어야 한다 — REPLACE는 주기를 계속 리셋시켜 실행 자체가 안 될 위험이 있다.
3. 정책이 안전한 이유가 정책 값 자체가 아니라 호출 경로의 구조(항상 cancel 먼저 거침)에 있을 수 있다 — 이건 호출부를 추적해야만 보인다.
4. "전체 점검했는데 새 버그가 없었다"는 결과도 의미 있는 성과다 — 판단 기준을 명문화하고, 그 판단을 검증하는 테스트를 남긴 것 자체가 다음 실수를 막는다.
5. "검증할 수정이 없다"도 정당한 실기기 검증 결론이다 — 꼭 뭔가를 고쳐야만 검증이 의미 있는 게 아니다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*
*작성일: 2026-09-18*
