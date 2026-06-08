# 051 - /autoresearch 자율 테스트 커버리지 향상 실험

**목표 설정만 하면 Claude가 자율적으로 테스트를 작성해서 커버리지 목표 달성 — Line 66%→75%, Branch 63%→70%**

---

## 목적

oh-my-claudecode의 `/autoresearch` 명령어 실험.  
목표와 평가 기준만 설정하면 Claude가 반복 루프로 자율적으로 개선하는 방식 검증.

---

## /autoresearch란?

`/sciomc`가 "한 번 분석하고 리포트"라면, `/autoresearch`는 **평가 기준을 통과할 때까지 반복 루프**를 도는 방식.

**실행 순서:**
```
/deep-interview --autoresearch
    ↓
미션 + 목표 + 평가자(evaluator) 설정
    ↓
autoresearch 루프 실행 (5회 이상 이터레이션)
    ↓
evaluator 통과 시 종료
```

**총 소요 시간: 33분 38초**  
반복당 테스트 작성 → 빌드 → JaCoCo 측정 → 갭 분석 → 다음 타깃 선정 사이클.

---

## 설정값

| 항목 | 값 |
|------|---|
| 미션 | 테스트 커버리지 향상 |
| 목표 | Line ≥ 75%, Branch ≥ 70% |
| 평가자 | JaCoCo XML 자동 파싱 |
| 시작 커버리지 | Line 66%, Branch 63% |

---

## 결과

| 지표 | 시작 | 종료 | 달성 |
|------|------|------|------|
| Line Coverage | 66% | **75.06%** | ✅ |
| Branch Coverage | 63% | **70.15%** | ✅ |
| 추가 테스트 코드 | - | **~1,500줄** | - |
| 커밋 | - | 62c99fc | - |

---

## 새로 만든 테스트 파일 (15개)

- `ApiExtTest` — toDomainException() 모든 분기 (20 tests)
- `TokenRepositoryImplTest`
- `MemoRepositoryImplTest`
- `TmdbAuthRepositoryImplTest`
- `ReminderRepositoryImplTest`
- `WatchlistRepositoryImplTest`
- `FavoriteRepositoryImplTest`
- `CompleteOnboardingUseCaseTest`
- `GetWatchProvidersUseCaseTest`
- `SearchLocalMoviesUseCaseTest`
- `SyncTmdbAccountUseCaseTest`

---

## 기존 테스트 확장 (9개)

| 파일 | 추가 내용 |
|------|---------|
| ErrorMessageProviderTest | DomainException 9개 분기 추가 |
| WatchHistoryRepositoryImplTest | saveWatchHistory, getGenreCounts 등 +8 |
| UserRatingRepositoryImplTest | getUserRating, deleteUserRating 등 +5 |
| PreferencesRepositoryImplTest | 잘못된 themeMode 복구, setOnboardingCompleted +2 |
| FavoriteWatchlistUseCasesTest | sortOrder 오버로드 커버 |
| PreferencesTagUseCasesTest | 추가 케이스 |
| TagRepositoryImplTest | 추가 케이스 |
| MemoRatingPersonUseCasesTest | 엣지 케이스 추가 |
| CoroutineExtTest | 엣지 케이스 추가 |

## 이터레이션별 커버리지 진행

| 이터레이션 | Line | Branch | 주요 작업 |
|-----------|------|--------|---------|
| 시작 | 66% | 63% | - |
| 1 | 69.3% | 62.3% | CompleteOnboardingUseCase, GetWatchProvidersUseCase 등 |
| 2 | 70.0% | 68.0% | ApiExtTest, ErrorMessageProviderTest DomainException 추가 |
| 3 | 73.1% | 70.15% | WatchlistRepositoryImpl, ReminderRepositoryImpl, TmdbAuthRepositoryImpl |
| 4 | 74.48% | 70.15% | Branch ✅ 달성, WatchHistoryRepositoryImpl, SearchLocalMoviesUseCase |
| 5 | 74.99% | 70.15% | FavoriteWatchlistUseCasesTest, PreferencesTagUseCasesTest 추가 |
| **최종** | **75.06%** | **70.15%** | FavoriteRepositoryImplTest로 마지막 1줄 커버 |

---

## 자동 Detekt 오류 수정

커밋 시 pre-commit Detekt 오류 발생 → 자동 수정 후 재커밋:

```
fully-qualified FavoriteSortOrder 참조로 라인 길이 초과
    ↓
import 추가 + 람다 블록 분리로 자동 수정
    ↓
Detekt 통과 ✅
```

---



| 항목 | /sciomc (050) | /autoresearch (051) |
|------|:------------:|:------------------:|
| 동작 방식 | 1회 분석 후 리포트 | 목표 달성까지 반복 루프 |
| 사용자 개입 | 주제 선택 | 미션 + 목표 + evaluator 설정 |
| 결과물 | 분석 리포트 | 실제 코드 변경 + 커밋 |
| 측정 기준 | 정성적 | 정량적 (JaCoCo 수치) |
| 언제 쓰나 | "뭐가 문제인지 알고 싶다" | "목표까지 자동으로 개선해줘" |

---

## 느낀 점

목표(Line 75%, Branch 70%)와 평가 기준(JaCoCo XML)만 설정했더니 Claude가 알아서 어느 파일에 테스트가 부족한지 파악하고, 15개 파일을 새로 만들고, 9개를 확장해서 목표를 달성했다.

`/sciomc`가 "무엇이 문제인지 보여주는 도구"라면, `/autoresearch`는 "문제를 직접 해결하는 도구"다.

**"목표를 주면 알아서 달성하는 AI" — 개발자의 역할이 구현에서 목표 설정으로 바뀌고 있다.**

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-06-08*
