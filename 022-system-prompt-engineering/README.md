# 022 - System Prompt 설계 실험

**같은 요청, 다른 System Prompt — 결과가 얼마나 달라지는가**

---

## 목적

System Prompt의 설계 수준에 따라 Claude Code의 응답 품질이 어떻게 달라지는지 실험.  
프롬프트 엔지니어링은 특정 도구에 종속되지 않는 범용 AI 활용 역량이라는 관점에서 접근.

---

## 실험 조건

**동일한 요청:** `MovieFinder의 HomeViewModel을 리뷰해줘.`

| 단계 | System Prompt |
|------|--------------|
| 1단계 | 없음 |
| 2단계 | `당신은 10년 경력의 시니어 Android 개발자입니다.` |
| 3단계 | 역할 + 분석 관점 + 심각도 표시 + 코드 예시 + 테스트 관점 5가지 제약 조건 |

---

## 실험 결과 비교

| 구분 | System Prompt 없음 | 역할만 부여 | 역할 + 제약 + 형식 |
|------|------------------|------------|------------------|
| 발견 이슈 수 | 0개 | 0개 | **3개** |
| 심각도 분류 | 없음 | 없음 | Critical/Major/Minor |
| 개선 코드 예시 | 없음 | 없음 | **있음** |
| 테스트 관점 | 없음 | 없음 | **있음** |
| 트레이드오프 분석 | 없음 | 없음 | **있음** |
| 응답 깊이 | 표면적 | 약간 구체적 | 전문가 수준 |

---

## 3단계 System Prompt 전문

```
당신은 10년 경력의 시니어 Android 개발자이자 코드 리뷰 전문가입니다.
다음 규칙을 반드시 따르세요:
1. Kotlin Coroutines, Flow, Paging3 관점에서 분석
2. 실제 프로덕션 환경에서 발생할 수 있는 문제 위주로
3. 각 이슈에 심각도(Critical/Major/Minor)를 반드시 표시
4. 개선 코드 예시를 반드시 포함
5. 테스트 가능성 관점도 포함
```

---

## 3단계에서만 발견된 것들

**Minor — lazy 스레드 안전 모드 불필요**
```kotlin
// 현재
val nowPlayingMovies by lazy {
    getNowPlayingMoviesUseCase().cachedIn(viewModelScope)
}

// 개선 — ViewModel은 UI 스레드 전용이므로 동기화 오버헤드 불필요
val nowPlayingMovies by lazy(LazyThreadSafetyMode.NONE) {
    getNowPlayingMoviesUseCase().cachedIn(viewModelScope)
}
```

**Minor — watchHistory 초기 Loading 상태 미구분**
```kotlin
// 개선
sealed interface WatchHistoryState {
    data object Loading : WatchHistoryState
    data class Success(val history: List<Movie>) : WatchHistoryState
}
```

**Major — 3개 Paging Flow 동시 캐시 메모리 상주 (설계 트레이드오프)**
- 현재: 탭 3개 Flow가 항상 메모리에 상주 → 빠른 탭 전환, 캐시 유지
- 대안: flatMapLatest로 단일 Flow → 탭 전환마다 캐시 소멸
- **결론: 현재 방식이 UX상 더 좋음, 의도적 트레이드오프**

---

## 핵심 발견

**역할만 부여해서는 충분하지 않다.**

"시니어 Android 개발자" 역할을 부여해도 결과가 System Prompt 없을 때와 거의 동일했다. 진짜 차이는 **제약 조건과 출력 형식을 명확하게 지정했을 때** 나타났다.

**System Prompt 설계의 3요소:**
1. **역할(Role)** — 누구의 관점으로 볼 것인가
2. **제약 조건(Constraint)** — 무엇을 반드시 포함할 것인가
3. **출력 형식(Format)** — 어떤 구조로 응답할 것인가

세 가지가 모두 있을 때 응답 품질이 가장 높아진다.

---

## 언제 어떤 System Prompt를 쓰면 좋을까

| 상황 | 추천 |
|------|------|
| 빠른 질문, 간단한 작업 | System Prompt 없이 |
| 특정 도메인 전문가 관점 필요 | 역할만 부여 |
| 일관된 품질의 결과물 반복 생성 | 역할 + 제약 + 형식 모두 설계 |
| 팀 내 AI 활용 표준화 | System Prompt 템플릿화 |

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-04-23*
