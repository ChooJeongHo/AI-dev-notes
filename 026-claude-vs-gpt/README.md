# 026 - Claude vs GPT 동일 프롬프트 비교 실험

**같은 조합 프롬프트를 두 모델에 보냈을 때 결과가 어떻게 다른가**

---

## 목적

25일차 조합 프롬프트(System Prompt + Few-shot + CoT)를 Claude와 GPT에 동일하게 적용해서 두 모델의 차이를 비교.  
"왜 이 모델을 쓰는가"를 직접 실험으로 확인.

---

## 실험 조건

**동일한 조합 프롬프트:** System Prompt + Few-shot + CoT  
**동일한 코드:** `HomeViewModel.kt` (MovieFinder)

| 구분 | Claude Sonnet 4.6 | GPT (Auto → 5.3) |
|------|------------------|-----------------|
| 분석 단계 | 5단계 | 7단계 |
| 발견 이슈 수 | 3개 | **5개** |
| 코드 예시 | 있음 | 있음 |
| 테스트 관점 | 있음 | 있음 |

---

## 가장 중요한 차이 — flatMapLatest 관점

**GPT:**
> "탭 상태와 데이터 흐름이 분리됨 — 무조건 flatMapLatest로 통합해야 한다. 이게 핵심."

**Claude:**
> "현재 3개 독립 Flow 방식은 탭 전환 시 캐시가 유지된다는 장점이 있다. flatMapLatest 방식은 탭 전환마다 이전 캐시가 날아간다. UX 관점에서 현재 방식이 더 나을 수 있다."

GPT는 일반적인 정답을 제시했고, Claude는 **프로젝트 맥락을 고려한 트레이드오프 분석**을 했다.

---

## 각 모델이 발견한 것

### GPT만 발견한 것
- UseCase를 `private val`로 보관하지 않음 → 재사용/테스트 불편
- `watchHistory`에 `.catch` 에러처리 누락 → Flow 예외 시 UI 멈춤 가능

### Claude만 발견한 것
- 3개 Paging Flow 동시 캐시 메모리 상주 트레이드오프 분석 (설계 의도 확인 필요)

### 둘 다 발견한 것
- `lazy` 스레드 안전 모드 불필요 (SYNCHRONIZED → NONE)
- `watchHistory` 초기 Loading 상태 미구분

---

## GPT가 제안한 개선 코드

```kotlin
val movies = selectedTab
    .flatMapLatest { tab ->
        when (tab) {
            HomeTab.NOW_PLAYING -> getNowPlayingMoviesUseCase()
            HomeTab.POPULAR -> getPopularMoviesUseCase()
            HomeTab.TRENDING -> getTrendingMoviesUseCase()
            HomeTab.UPCOMING -> getUpcomingMoviesUseCase()
        }
    }
    .cachedIn(viewModelScope)

val watchHistory = getWatchHistoryUseCase()
    .catch { emit(emptyList()) }
    .stateIn(viewModelScope, WhileSubscribed5s, emptyList())
```

---

## 두 모델의 성격 차이

| 관점 | Claude | GPT |
|------|--------|-----|
| 접근 방식 | 맥락 고려, 트레이드오프 분석 | 일반적인 Best Practice 제시 |
| 발견 이슈 수 | 적음 | 많음 |
| 응답 스타일 | 신중하고 보수적 | 단호하고 직접적 |
| 실용성 | 현재 코드 맥락 반영 | 교과서적 정답 |

---

## 느낀 점

GPT가 이슈를 더 많이 발견했지만 Claude가 더 쓸모없다는 뜻이 아니다.

GPT는 "이렇게 해야 한다"고 단호하게 말하는 반면, Claude는 "이렇게 하면 이런 트레이드오프가 있다. 현재 방식이 오히려 나을 수 있다"고 판단해줬다.

실무에서는 둘 다 필요하다:
- **GPT** → 빠르게 Best Practice 확인할 때
- **Claude** → 현재 코드 맥락에서 트레이드오프를 고려한 판단이 필요할 때

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*비교 모델: Claude Sonnet 4.6 vs GPT Auto(5.3)*  
*작성일: 2026-04-29*
