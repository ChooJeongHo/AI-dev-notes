# AI 활용 기록 003 - Android 코드리뷰 스킬 실전 검증

> 2일차에서 이론적으로 예측한 스킬의 효과를 실제 실험으로 검증한 기록

## 목적

2일차에 직접 설계한 `android-code-review` 스킬이 실제로 유효한지 검증  
동일한 코드를 **스킬 없이** → **스킬 적용 후** 순서로 리뷰하여 차이를 직접 확인

## 사용한 도구

- Claude Code
- 직접 제작한 `android-code-review` 스킬 ([2일차 산출물](../002-claude-code-skill-analysis/))
- 실험 대상 프로젝트: [MovieFinder](https://github.com/ChooJeongHo/MovieFinder)

---

## 실험 방법

1. `DetailViewModel.kt` 파일 선택
2. **스킬 없이** → "이 코드 리뷰해줘" 요청
3. **스킬 적용 후** → "이 코드 android-code-review 스킬을 사용해서 리뷰해줘" 요청
4. 두 결과 비교

실험 대상으로 `DetailViewModel.kt`를 선택한 이유:  
2일차 분석에서 잘된 패턴(Mutex, 병렬 API 호출 등)이 집중된 파일이라  
스킬 유무에 따른 차이가 가장 명확하게 드러날 것으로 예상했기 때문

---

## 결과 비교

### 스킬 없이 리뷰했을 때

| 항목 | 평가 |
|------|------|
| 에러 처리 | 우수 (CancellationException rethrow, 부분 실패, Channel 이벤트) |
| 동시성 | 우수 (Mutex, coroutineScope + async 병렬) |
| 코드 중복 | 우수 (launchWithSnackbar, loadOptional 헬퍼) |
| 책임 분리 | 보통 (UseCase 15개 → Facade 고려 가능) |
| 확장성 | 보통 (toMovie()를 도메인 레이어로 이동 고려) |

> 전체적으로 잘 설계된 ViewModel이고, 개선점은 "있으면 좋은" 수준

---

### 스킬 적용 후 리뷰했을 때

| 카테고리 | 등급 | 비고 |
|----------|------|------|
| 아키텍처 | A | 순수 도메인 레이어 의존성, 적절한 UseCase 패턴, sealed class UI 상태 |
| 코루틴 안전성 | A | Mutex 가드, CancellationException rethrow, 구조적 동시성 |
| 메모리 관리 | A | Fragment에서 binding null 처리, 어댑터 해제, viewLifecycleOwner 사용 |
| 에러 처리 | A | loadOptional로 부분 실패 처리, Channel 기반 일회성 이벤트, ErrorType 추상화 |
| Android 모범 사례 | A | SavedStateHandle, repeatOnLifecycle, Safe Args |
| 코드 품질 | A- | 깔끔한 헬퍼, 좋은 네이밍; 사소한 FQN 사용 |

**종합 등급: A** — 프로덕션 수준의 코드

> 스킬이 추가로 발견한 이슈:
> - `143행, 203행` FQN 문제 (import 누락으로 풀 패키지명 사용)
> - `68행` Channel 버퍼 과잉 할당 (BUFFERED → CONFLATED 권장)
> - `161행` toggleWatchlist 경쟁 조건 가능성 (실용적으로는 무시 가능)

---

## 핵심 차이 요약

| 비교 항목 | 스킬 없음 | 스킬 있음 |
|-----------|----------|----------|
| 출력 형식 | 자유 형식 (잘된 점 / 개선점) | A~F 등급 + 카테고리별 구조화 |
| 체크 항목 | 눈에 띄는 것 위주 | 5개 카테고리 빠짐없이 체크 |
| 세밀도 | 큰 패턴 위주 | FQN, 버퍼 크기 등 세밀한 이슈까지 발견 |
| 일관성 | 매번 다를 수 있음 | 항상 동일한 기준으로 리뷰 |

---

## 스킬 개선 사항

실험을 통해 발견한 스킬 보완점

- 등급 기준(A/B/C)이 SKILL.md에 명시되어 있지 않아 Claude가 자체 판단으로 부여함 → 기준 명시 필요
- Fragment 관련 체크리스트가 SKILL.md에 없었는데 스킬이 자동으로 확인함 → 명시적으로 추가하면 더 일관성 있을 것

---

## 느낀 점

스킬 없이도 Claude Code의 리뷰 품질은 이미 높은 편이었다.  
하지만 스킬을 적용하면 **체크 누락 없이 일관된 기준**으로 리뷰가 나온다는 점이 명확히 달랐다.

특히 스킬 없이는 발견하지 못한 `FQN import 문제`나 `Channel 버퍼 과잉 할당` 같은  
사소하지만 실질적인 개선점을 추가로 찾아낸 것이 인상적이었다.

스킬은 AI의 능력을 높이는 게 아니라, **빠짐없이 체크하도록 강제하는 체크리스트** 역할을 한다는 것을 직접 확인했다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-03-12*
