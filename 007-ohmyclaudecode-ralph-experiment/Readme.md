# AI 활용 기록 007 - oh-my-claudecode ralph 모드 실험

> 5일차 ulw, 6일차 team에 이어 ralph 모드로 oh-my-claudecode 3부작 완성

## 목적

oh-my-claudecode의 `ralph` 모드를 실험하여 ulw, team 모드와의 차이를 확인  
새 기능 구현 + 테스트 코드 작성을 ralph 모드에게 맡겨 오류를 스스로 수정하는 과정 기록

## 사용한 도구

- Claude Code CLI (터미널)
- oh-my-claudecode v4.8.2 `ralph` 모드
- 실험 대상 프로젝트: [MovieFinder](https://github.com/ChooJeongHo/MovieFinder)

---

## ralph 모드란?

**"완성될 때까지 멈추지 않는"** 집요한 실행 모드

```
ralph [작업 내용]
```

- 오류가 발생해도 스스로 수정하며 계속 진행
- 빌드/테스트 실패 시 원인을 분석하고 재시도
- 내부적으로 ultrawork + 백그라운드 에이전트를 자동 생성
- 이름의 유래: 시지프스 신화 ("The boulder never stops rolling")

---

## oh-my-claudecode 3가지 모드 비교

| 모드 | 특징 | 언제 쓰나 |
|------|------|----------|
| `ulw` | 병렬 에이전트로 빠르게 | 독립적인 작업 동시 처리 |
| `/team` | 역할 명시적 분배 | 레이어별 명확한 분업 |
| `ralph` | 완성될 때까지 집요하게 | 오류 수정이 반복되는 복잡한 작업 |

---

## 실험: 시청 목표 기능 추가

MovieFinder 통계/설정 화면에 이번 달 시청 목표를 설정하고 달성률을 확인하는 기능을 ralph 모드로 구현

### 요청한 프롬프트

```
ralph MovieFinder에 시청 목표 기능을 추가해줘.
기능 내용:
- 이번 달 시청 목표 편수 설정 (설정 화면에서)
- 현재 달성률 표시 (통계 화면에서)
- 목표 달성 시 알림
기존 Clean Architecture + MVVM 패턴 유지하고,
테스트 코드까지 작성해줘.
오류가 생기면 스스로 수정하면서 완성해줘.
```

**총 소요 시간: 14분 39초**

---

## ralph 모드 실행 특이점

ralph 모드를 실행했더니 내부적으로 **백그라운드 에이전트를 자동 생성**해서 병렬로 돌렸다:

```
Agent "US-003: Settings UI watch goal" completed
Agent "US-004: Stats UI goal progress" completed
Agent "US-005: Goal achievement notification" completed
```

ralph 키워드 하나만 입력했는데 스스로 작업을 User Story 단위로 쪼개서 병렬 처리했다.  
ulw나 team 모드처럼 별도 지시 없이 자동으로 최적화한 것이 인상적이었다.

---

## 구현 결과

### 새로 생성된 파일 (3개)

| 파일 | 설명 |
|------|------|
| GetMonthlyWatchGoalUseCase.kt | 월간 시청 목표 조회 UseCase |
| SetMonthlyWatchGoalUseCase.kt | 월간 시청 목표 설정 UseCase |
| WatchGoalNotificationHelper.kt | 목표 달성 알림 헬퍼 |

### 수정된 파일 (15개)

- UserSettings.kt — monthlyWatchGoal, lastGoalNotifiedMonth 필드 추가
- PreferencesRepositoryImpl.kt — get/set goal, get/set notified month 4개 메서드 구현
- PreferencesRepository.kt — 4개 인터페이스 메서드 추가
- WatchStats.kt — monthlyWatchGoal 필드 추가
- GetWatchStatsUseCase.kt — 6개 Flow combine
- SettingsViewModel.kt — monthlyWatchGoal StateFlow + setMonthlyWatchGoal()
- SettingsFragment.kt — NumberPicker 다이얼로그 + 목표 표시
- StatsFragment.kt — 목표 달성률 카드 + LinearProgressIndicator
- DetailViewModel.kt — WatchGoalNotificationHelper 주입
- MovieFinderApp.kt — watch_goal_channel 알림 채널 추가
- nav_graph.xml — moviefinder://stats 딥링크 추가
- fragment_settings.xml — 시청 목표 설정 항목 추가
- fragment_stats.xml — 목표 달성률 카드 추가
- strings.xml (ko/en) — 10개 새 문자열

### 테스트 결과

- 기존 145개 → **154개** (9개 자동 추가)
- SettingsViewModelTest: +4개
- PreferencesRepositoryImplTest: +5개
- Detekt PASS, 빌드 SUCCESS

---

## 실행 화면

| 설정 화면 - 목표 설정 | 통계 화면 - 달성률 |
|:---:|:---:|
| <img width="250" alt="시청 목표 설정" src="https://github.com/user-attachments/assets/setting_screenshot" /> | <img width="250" alt="달성률 화면" src="https://github.com/user-attachments/assets/stats_screenshot" /> |

> 설정에서 NumberPicker로 목표 편수 설정 → 통계 화면에서 달성률 + 프로그레스바 + 남은 편수 확인

---

## oh-my-claudecode 3부작 종합 비교

| 항목 | ulw (5일차) | /team (6일차) | ralph (7일차) |
|------|------------|--------------|--------------|
| 모드 특성 | 병렬 빠른 처리 | 역할 명시 분업 | 집요한 완성 |
| 에이전트 분배 | AI 자동 | 직접 지정 | AI 자동 (User Story 단위) |
| 테스트 자동 작성 | 없음 | 없음 | 있음 (9개 자동 추가) |
| 오류 자동 수정 | 제한적 | 제한적 | 핵심 특기 |
| 소요 시간 | 8분 2초 | 31분 27초 | 14분 39초 |
| 적합한 작업 | 빠른 기능 추가 | 레이어 명확한 작업 | 복잡한 기능 + 테스트 |

---

## 느낀 점

ralph 모드는 단순히 집요한 게 아니라 **스스로 작업을 구조화하는 능력**이 있었다.

`ralph` 키워드 하나만 입력했는데, 내부적으로 User Story 단위로 작업을 쪼개고 백그라운드 에이전트를 생성해서 병렬로 처리했다. ulw처럼 따로 지시하지 않아도 알아서 최적화한 것이다.

테스트 코드도 자동으로 작성해줬는데, 145개 → 154개로 9개가 추가됐다. 기능 구현과 테스트를 동시에 맡길 수 있다는 게 실무에서 가장 유용할 것 같다.

5~7일차를 통해 ulw → team → ralph 세 가지 모드를 모두 써봤는데,  
**상황에 따라 골라 쓰는 것**이 핵심이라는 걸 알게 됐다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-03-23*
