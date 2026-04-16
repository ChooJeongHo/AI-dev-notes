# AI 활용 기록 018 - think / ultra think 키워드 실험

> 동일한 요청에 키워드를 다르게 붙였을 때 Claude Code의 분석 깊이가 어떻게 달라지는지 비교

## 목적

`think`, `ultra think` 키워드가 Claude Code의 동작 방식에 어떤 차이를 만드는지 실험  
프롬프트 작성 역량이 AI 활용 능력의 핵심이라는 관점에서 접근

## 실험 조건

**동일한 요청:** MovieFinder 성능 개선점 찾기 (수정 없이 분석만)

| 단계 | 프롬프트 |
|------|---------|
| 1단계 | `MovieFinder에서 성능 개선할 수 있는 부분 찾아줘. 수정은 하지 말고 분석 결과만 알려줘.` |
| 2단계 | `think MovieFinder에서 성능 개선할 수 있는 부분 찾아줘. 수정은 하지 말고 분석 결과만 알려줘.` |
| 3단계 | `ultra think MovieFinder에서 성능 개선할 수 있는 부분 찾아줘. 수정은 하지 말고 분석 결과만 알려줘.` |

---

## 실험 결과 비교

| 구분 | 키워드 없음 | think | ultra think |
|------|-----------|-------|-------------|
| 분석 방식 | 단일 에이전트 | 단일 에이전트 | **4개 전문 에이전트 병렬 실행** |
| 발견 이슈 수 | ~7개 | ~7개 | **30개 이상** |
| 분석 깊이 | 표면적 | 표면적 | 코루틴/DB/UI/네트워크 전문 분석 |
| 우선순위 분류 | 없음 | 간단 | High/Medium/Low + 임팩트 순위표 |
| 소요 시간 | 빠름 | 빠름 | 느림 (47초 + 병렬 에이전트) |

---

## 핵심 발견

### think ≈ 키워드 없음

Claude Code에서 `think` 키워드는 일반 요청과 결과 차이가 거의 없었다.

이유: Claude Code는 코드를 읽고 분석하는 과정 자체가 이미 "thinking"이기 때문에, 단순 분석 요청에서는 `think` 키워드가 추가적인 효과를 내지 못했다.

`think` 키워드가 효과적인 상황:
- 수학/논리 문제 같은 순수한 추론이 필요한 경우
- 복잡한 트레이드오프 판단이 필요한 결정
- Claude.ai 채팅창에서 사용 시 (사고 과정이 텍스트로 표시됨)

### ultra think = 병렬 에이전트

`ultra think`는 단순히 더 오래 생각하는 게 아니었다.

```
ultra think 요청
    ↓
4개 전문 에이전트 병렬 실행
├── perf-coroutines: 코루틴/Flow/스레딩 분석
├── perf-db: Room DB/데이터 레이어 분석
├── perf-ui: UI 렌더링/메모리 분석
└── perf-network: 네트워크/캐싱 분석
    ↓
4개 결과 통합 → 우선순위 정렬
```

키워드 없음/think에서 못 찾은 것들:
- OkHttp `Cache-Control: no-cache` 때문에 캐시가 실제로 작동 안 하는 버그
- `strftime()` DB 쿼리에서 인덱스가 무용지물이 되는 문제
- `SearchViewModel` 인물 검색 레이스 컨디션
- `SettingsFragment` 백업/복원이 메인 스레드를 블로킹하는 문제

---

## ultra think 발견 이슈 요약

### 🔴 High
- SettingsFragment 백업/복원 메인 스레드 블로킹
- SearchViewModel 인물 검색 `flatMapLatest` 미사용
- OkHttp Cache-Control 재작성 필요 (캐시 사실상 무효)
- ExponentialBackoff 정의만 있고 미사용
- cached_movies 페이지 정렬 인덱스 누락
- Stats 월별 집계 `strftime()` 인덱스 무용지물

### 🟡 Medium (14개)
FavoriteViewModel 중첩 flatMapLatest, 이미지 OkHttpClient 캐시 없음, 장르 목록 API 재호출, MovieDto 불필요한 필드 파싱 등

### 🟢 Low (11개)
DetailViewModel StateFlow 재구독, HomeFragment 독립 repeatOnLifecycle 3개, 커스텀 뷰 오버드로우 등

---

## 언제 어떤 키워드를 쓰면 좋을까

| 상황 | 추천 |
|------|------|
| 빠른 코드 수정, 간단한 기능 추가 | 키워드 없이 |
| 복잡한 논리 추론, 트레이드오프 판단 | `think` |
| 대규모 코드 분석, 심층 버그 탐색 | `ultra think` |
| 시간/토큰이 중요할 때 | 키워드 없이 |

---

## 느낀 점

`think`와 키워드 없음의 차이가 예상보다 작았다. Claude Code를 많이 써왔기 때문에 이미 패턴을 알고 있어서 차이가 덜 느껴진 것일 수도 있다.

반면 `ultra think`는 명확하게 달랐다. 단순히 더 깊이 생각하는 게 아니라 **병렬 에이전트를 통해 각 도메인을 전문적으로 분석**하는 방식이 바뀐 것이다.

**실용적인 결론:** 빠른 작업에는 키워드 없이, 복잡한 분석이 필요할 때는 `ultra think`를 사용하는 게 효율적이다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-04-17*
