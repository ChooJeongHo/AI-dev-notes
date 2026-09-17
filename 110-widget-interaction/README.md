# 110 - Glance 위젯 인터랙션 강화 — 092일차 우려는 기우였고, 대신 WorkManager 백오프 버그를 새로 찾았다

**092일차에서 걱정했던 R8 리플렉션 문제는 실측으로 기우였음이 확인됐지만, 095~109일차 계열의 새로운 변종 — dumpsys jobscheduler로만 드러나는 WorkManager 백오프 버그를 대신 발견했다**

---

## 목적

086일차에 만든 BoxOfficeGlanceWidget에 인터랙션(일별/주간 토글)을 추가한다.

---

## 작업 1 — 조사

새로고침 버튼은 086일차 이후 이미 존재했다. 일별/주간 토글은 없었다.

---

## 작업 2 — 설계

083일차의 GetWeeklyBoxOfficeWithTmdbMatchUseCase + BoxOfficePeriod enum과 기존 문자열 리소스를 재사용하고, 기간별(DAILY/WEEKLY) 캐시를 분리하는 구조로 설계했다.

---

## 작업 3 — 구현: BoxOfficePeriodToggleAction

위젯 헤더의 "일별"/"주간" 배지를 탭하면 실행되는 ActionCallback으로, 3단계로 동작한다.

1. 해당 위젯 인스턴스(glanceId)에 저장된 현재 기간을 읽어 DAILY↔WEEKLY로 반전시켜 Glance state에 저장
2. BoxOfficeWidget().update(context, glanceId)를 명시적으로 호출해 헤더 라벨을 즉시 재구성 (Glance 공식 문서 권장 패턴 — 안 하면 재구성이 보장 안 됨)
3. BoxOfficeWidgetWorker.enqueueOneTimeRefresh()를 호출해 새 기간의 실제 데이터를 백그라운드로 예약 (캐시가 있으면 그동안은 캐시된 데이터가 즉시 보임)

### 086일차 대비 새로 필요했던 것

- BoxOfficePeriod를 per-glanceId로 저장(다중 위젯 인스턴스가 각자 독립적인 기간을 가질 수 있도록) + per-period 스냅샷 캐시 키 분리
- Worker가 각 위젯 인스턴스의 저장된 기간을 읽어 daily/weekly UseCase 중 선택 호출
- 헤더에 기간 배지 UI 추가 — 새 벡터 아이콘 없이 텍스트 배지 + pill 배경 drawable 2개(day/night)만 추가. 새 직렬화 클래스는 없어서 ProGuard 규칙 추가는 불필요했다.

---

## 092일차 우려는 기우였다

092일차에서 걱정했던 "R8이 리플렉션 기반 클래스명을 난독화하면 조용히 깨질 수 있다"는 우려를, 이번엔 실측으로 직접 검증했다.

R8 매핑을 확인한 결과, 새로 만든 BoxOfficePeriodToggleAction이 기존 Glance 액션과 동일하게 이름이 보존됨을 확인했다 — 우려는 기우였다.

---

## 새로 발견한 버그 — 095~109일차 계열의 새로운 변종

092일차 우려는 기우로 끝났지만, 대신 완전히 새로운 유형의 실기기 전용 버그를 하나 발견했다.

```
enqueueOneTimeRefresh()가 ExistingWorkPolicy.KEEP을 사용
    ↓
최초 KOFIC 서버 타임아웃 이후, WorkManager가 지수 백오프로 재시도 대기 중일 때
    ↓
그 상태에서 새로고침/토글을 눌러도 "이미 대기 중인 작업이 있다"며
   그 작업을 그대로 둘 뿐, 새 요청을 반영 안 함
    ↓
최대 10분 이상 위젯이 무반응처럼 보이는 문제
```

수정: ExistingWorkPolicy.KEEP → REPLACE로 변경해서, 사용자가 새로고침/토글을 누르면 대기 중인 예전 작업을 취소하고 즉시 새 작업으로 교체하도록 했다.

이 버그는 086일차 새로고침 버튼에도 원래 있던 버그였다 — 오늘 토글 기능을 만들다가 우연히 발견됐다.

### 왜 095~109일차 계열과 같은 성격인가

이 버그는 유닛 테스트나 빠른 네트워크 환경에서는 절대 안 보이고, dumpsys jobscheduler로 백오프 타이머를 직접 들여다봐야만 드러난다. 101(리소스 타입)/103(API 오용)/104(커버리지 누락)/106(Intent 매칭)/109(ViewModel 스코프)일차와 마찬가지로, 정적분석으로는 절대 못 잡고 실기기의 특정 타이밍 상태를 직접 관찰해야만 발견되는 버그였다.

---

## 작업 4 — 테스트: 20/20 (신규 5 + 갱신 15)

| 파일 | 변화 |
|---|---|
| BoxOfficeWidgetStateTest | 3개 → 7개 (기간 무관 캐시 독립성, readPeriod 기본값/저장값/폴백 등 4개 신규) |
| BoxOfficeWidgetWorkerTest | 5개 → 6개 (WEEKLY UseCase 호출 검증 1개 신규) |
| BoxOfficeWidgetSnapshotTest | 7개 (데이터 모델 자체는 안 건드려서 변경 없음) |

합계 20개 — 신규 5개, 나머지 15개는 시그니처 변경에 맞춰 갱신한 기존 테스트.

---

## 작업 5 — 실기기 검증 (SM-S926N)

토글 즉시 재구성, per-period 캐시 즉시 표시, R8 리플렉션 이상 없음까지 전부 정상 동작을 확인했고, 유일하게 발견된 문제가 위의 ExistingWorkPolicy 버그였다.

---

## 변경 파일

BoxOfficePeriodToggleAction.kt(신규), BoxOfficeWidget.kt, BoxOfficeWidgetContent.kt, BoxOfficeWidgetWorker.kt, 관련 테스트 2개, 문자열 리소스, 배지 배경 drawable 2개. 커밋은 아직 안 된 상태다.

---

## 070~110을 관통하는 패턴 — 예상한 위험과 실제 위험은 다를 수 있다

092일차에서 예상했던 위험(R8 리플렉션)은 실측으로 기우였지만, 전혀 예상 못 했던 곳(WorkManager 백오프 정책)에서 진짜 버그가 나왔다. 이건 099/100일차("클린의 성격이 매번 다르다")와도 이어지는 통찰이다 — 미리 걱정한 지점이 안전하다고 확인됐다고 해서 전체가 안전한 게 아니라, 실기기에서 끝까지 확인해야 진짜 위험 지점이 드러난다.

---

## 오늘 배운 것 — 한 줄 정리

1. 과거에 걱정했던 위험(092일차 R8)은 실측으로 검증하면 기우였는지 실제 위험이었는지 확실히 가려진다 — 걱정만 하고 넘어가지 않는 게 중요하다.
2. ExistingWorkPolicy.KEEP은 "이미 대기 중인 작업이 있으면 새 요청을 무시한다"는 뜻이라, 사용자가 즉시 반응을 기대하는 새로고침류 액션에는 REPLACE가 맞다.
3. WorkManager의 지수 백오프 상태는 dumpsys jobscheduler로 직접 들여다봐야만 확인 가능한, 유닛 테스트로는 절대 재현 안 되는 영역이다.
4. Glance 위젯에서 상태 변경 후 update()를 명시적으로 호출하지 않으면 재구성이 보장 안 된다 — 공식 문서가 권장하는 패턴을 지켜야 한다.
5. 새 기능을 만들다가 관련 없어 보이는 기존 기능(새로고침 버튼)의 숨은 버그를 우연히 발견하는 경우가 있다 — 오늘도 그랬다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*
*작성일: 2026-09-17*
