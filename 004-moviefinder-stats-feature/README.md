# AI 활용 기록 004 - MovieFinder 시청 통계 기능 추가

> 1일차와 동일한 개발 환경에서, 3일차까지 익힌 AI 워크플로우를 적용하여 얼마나 달라졌는지 비교한 기록

## 목적

1일차에 MovieFinder 앱을 처음 만들 때와 비교하여, Claude Code + 스킬을 활용한 개발 방식이 어떻게 발전했는지 직접 확인  
기존 Room DB 시청 기록 데이터를 활용한 **통계 화면**을 새로 구현

## 사용한 도구

- Claude Code
- `android-code-review` 스킬 (3일차에서 검증 완료)
- 실험 대상 프로젝트: [MovieFinder](https://github.com/ChooJeongHo/MovieFinder)

---

## 구현한 기능

### 시청 통계 화면 (StatsFragment)

설정 화면에서 **시청 통계** 항목을 누르면 진입하는 화면  
기존 Room DB에 쌓인 시청 기록 + 사용자 평점 데이터를 활용하여 아래 항목을 표시

| 항목 | 내용 |
|------|------|
| 총 시청 편수 | 지금까지 본 영화 총 개수 |
| 가장 많이 본 장르 Top 3 | 시청한 영화의 장르 분포 |
| 평균 별점 | 내가 매긴 별점 평균 |
| 이번 달 시청 편수 | 월별 시청 현황 |

---

## AI 활용 방식

### Claude Code에게 요청한 프롬프트

```
현재 프로젝트 구조와 기존 코드 패턴을 먼저 파악해줘.
특히 아래 파일들을 읽어봐:
- WatchHistoryEntity.kt
- UserRatingEntity.kt
- DetailViewModel.kt (기존 패턴 참고용)
- HomeFragment.kt (화면 구조 참고용)

파악이 끝나면 MovieFinder 프로젝트의 기존 Clean Architecture + MVVM 패턴을 유지하면서
시청 통계 화면을 추가해줘.

기존 WatchHistoryEntity, UserRatingEntity 데이터를 활용하고
아래 항목을 표시하는 StatsFragment를 만들어줘:
- 총 시청 편수
- 가장 많이 본 장르 Top 3
- 내가 매긴 별점 평균
- 이번 달 시청 편수

기존 코드의 패턴(ErrorType, launchWithSnackbar, CancellationException rethrow 등)을
그대로 따라줘.
```

1일차와 달리 **기존 파일을 먼저 읽히는 과정**을 추가했더니 Claude Code가 프로젝트 패턴을 훨씬 잘 따라왔다.

---

## 코드 리뷰 (android-code-review 스킬 적용)

### StatsViewModel 리뷰 결과

| 카테고리 | 등급 | 비고 |
|----------|------|------|
| 아키텍처 | A | Presentation → Domain만 의존, UseCase 패턴 준수, sealed class UI state |
| 코루틴 안전성 | C | loadStats() 재호출 시 이전 코루틴 미취소 — 레이스 컨디션 발생 |
| 메모리 관리 | A | Fragment에서 정상 cleanup |
| 에러 처리 | B | CancellationException rethrow 정상, ErrorType 매핑 정상 |
| Android 모범 사례 | B | 기능은 적절하나 프로젝트 내 stateIn(WhileSubscribed) 패턴과 불일치 |
| 코드 품질 | A | 간결하고 명확, 불필요한 코드 없음 |

**종합 등급: B** → 수정 후 A 가능

---

## 발생한 문제와 해결 과정

### 문제: 레이스 컨디션 (코루틴 안전성 C)

`loadStats()`를 재호출할 때마다 새 코루틴이 생성되지만 이전 코루틴이 취소되지 않아  
동시에 여러 코루틴이 같은 StateFlow에 값을 쓰는 레이스 컨디션이 발생

```kotlin
// 문제가 된 코드
fun loadStats() {
    viewModelScope.launch {       // retry 시 새 코루틴 추가 생성
        getWatchStatsUseCase().collect { ... }  // 이전 collect는 계속 살아있음
    }
}
```

### 해결: stateIn 패턴으로 전환

Claude Code가 제안한 `stateIn` 방식으로 수정 — 프로젝트 내 다른 ViewModel과도 패턴 일치

```kotlin
// 수정 후
@HiltViewModel
class StatsViewModel @Inject constructor(
    getWatchStatsUseCase: GetWatchStatsUseCase
) : ViewModel() {

    val uiState: StateFlow<StatsUiState> = getWatchStatsUseCase()
        .map<WatchStats, StatsUiState> { StatsUiState.Success(it) }
        .catch { e -> emit(StatsUiState.Error(ErrorMessageProvider.getErrorType(e))) }
        .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), StatsUiState.Loading)
}
```

Room Flow는 DB 변경 시 자동으로 갱신되므로 retry 로직 자체가 불필요해졌다.

---

## 1일차와 비교

| 항목 | 1일차 | 4일차 |
|------|-------|-------|
| 요청 방식 | 기능 설명 후 코드 생성 요청 | 기존 파일 먼저 읽힌 후 구현 요청 |
| 코드 검증 | 실행 여부만 확인 | 스킬로 아키텍처/코루틴/메모리 체크 |
| 문제 발견 | 실행 오류 위주 | 레이스 컨디션 같은 잠재적 버그까지 발견 |
| 문제 해결 | 오류 메시지 그대로 전달 | 스킬 리뷰 결과 보고 구체적 수정 방향 적용 |
| 전반적인 속도 | 코드 생성 → 오류 → 수정 반복 | 확실히 더 빠르고 수월했다 |

---

## 느낀 점

1일차보다 확실히 빠르고 수월했다.

가장 큰 차이는 **프롬프트 방식**이었다. 기존 파일을 먼저 읽히고 패턴을 파악하게 한 뒤 요청하니,  
Claude Code가 프로젝트 스타일에 맞는 코드를 처음부터 잘 만들어줬다.

스킬 덕분에 눈에 보이지 않는 **레이스 컨디션 버그**도 잡을 수 있었다.  
1일차였다면 실행이 되니까 그냥 넘어갔을 문제인데, 스킬이 코루틴 패턴까지 체크해줬다.

AI를 잘 쓰는 것은 단순히 코드를 생성하는 게 아니라,  
**무엇을 먼저 알려줘야 하는지 아는 것**이라는 걸 느꼈다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-03-16*
