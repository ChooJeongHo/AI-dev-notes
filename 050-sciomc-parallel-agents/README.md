# 050 - /sciomc 병렬 에이전트 아키텍처 건강도 분석

**5개 에이전트가 병렬로 코드베이스를 분석 — 이전 5번의 감사에서 놓쳤던 치명적 버그 2개 신규 발견**

---

## 목적

oh-my-claudecode의 `/sciomc` 명령어로 병렬 scientist 에이전트 실험.  
045일차 단일 에이전트 기술 부채 측정과 비교해서 병렬 에이전트의 차별점 확인.

---

## /sciomc란?

여러 scientist 에이전트가 각자 다른 영역을 병렬로 분석하고, 교차 검증 에이전트가 결과를 통합하는 방식.

**이번 실험 구성:**

| 스테이지 | 담당 영역 | 모델 |
|---------|---------|------|
| Stage 1 | 레이어 의존성 인벤토리 | Sonnet |
| Stage 2 | UseCase/Repository 패턴 | Sonnet |
| Stage 3 | Data 레이어 분석 | Sonnet |
| Stage 4 | Presentation 패턴 일관성 | Sonnet |
| Stage 5 | DI 구조 + 종합 평가 | Sonnet |
| 교차 검증 | 핵심 발견 5건 검증 | Sonnet |

---

## 종합 평가: 핵심 구조 A- / Hilt 경계 밖(위젯) D

### 🟢 매우 건강한 부분

- 레이어 준수율 **99.2%** (242개 파일 중 위반 2개, 모두 위젯에 격리)
- domain→data 역참조 **0건**
- UseCase 65개 invoke/네이밍 **100% 일관**
- DAO 패턴 **100% 일관**
- 오늘 추가한 BaseMoviePagingSource **4/4 적용 확인**

---

### 🔴 치명 (이전 감사 5번에서 모두 놓침 — 즉시 수정 권고)

**F1. 위젯 DB 전체 삭제 버그**
```
FavoriteMoviesRemoteViewsFactory.kt:136
WatchGoalWidget.kt:112
→ fallbackToDestructiveMigration(dropAllTables=true) 설정인데
   addMigrations() 9개 전부 누락
→ 앱 업그레이드 직후 위젯이 먼저 DB를 열면
   즐겨찾기/시청기록 전체 삭제
```

**F2. 리마인더 고스트 알림**
```
ReminderRepositoryImpl.kt:27
→ WorkManager 등록 → DB insert 순서
→ 사이 실패 시 DB 기록 없는 알림 발화
```

---

### 🟠 높음

- **F4**: `WatchGoalNotificationHelper.kt`에 도메인 로직 + core→app/domain 역참조
- **F5**: `FavoriteViewModel`이 Context/WorkManager 직접 참조 → JVM 테스트 불가, 의존성 15개

---

### 🟡 중간 (백로그 후보)

- `CreditsResponse.toDomain()` 데드코드
- Detail/Stats Fragment `stopShimmer()` 누락
- UiState sealed class 사용 50/50 분열 (8개 ViewModel 중 4개만 적용)

---

## 045일차 vs 050일차 비교

| 항목 | 045 단일 에이전트 | 050 병렬 에이전트 (/sciomc) |
|------|:---------------:|:-------------------------:|
| 분석 방식 | 단일 Claude Code | 5개 에이전트 병렬 + 교차 검증 |
| 부채 점수 | 28/100 🟢 | 핵심 구조 A- / 위젯 D |
| 치명 이슈 | 0건 | **2건 신규 발견** |
| 레이어 위반 | Presentation 집중 | 99.2% 준수, 위젯에 격리 |
| 분석 깊이 | 파일별 측정 | 시스템 관점 교차 분석 |

---

## 핵심 발견 — 왜 이전 감사에서 못 잡았나?

> "이전 감사들은 '레이어 규칙 준수'(grep으로 잡히는 것)에 집중했지만, 이번 치명 발견은 **두 코드 경로가 같은 리소스(DB 파일)를 다른 설정으로 여는 교차 관심사 문제** — 단일 파일 검사로는 안 보이고 시스템 관점에서만 보입니다."

**검증 단계의 가치**: 교차 검증이 치명 발견을 1곳→2곳으로 확장하고, 스테이지 간 외견상 모순을 정확히 조정했다.

---

## 느낀 점

5번의 코드리뷰와 기술 부채 측정에서 한 번도 안 잡혔던 버그가 병렬 에이전트에서 나왔다.

단일 에이전트는 파일 하나씩 보지만, 병렬 에이전트는 시스템 전체를 동시에 보기 때문에 **두 파일이 같은 리소스를 다르게 접근하는 문제**가 보인다.

**"혼자 깊이 보는 것"과 "여럿이 동시에 보는 것"은 다른 버그를 잡는다.**

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-06-05*
