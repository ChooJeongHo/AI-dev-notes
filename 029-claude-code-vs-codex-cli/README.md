# 029 - Claude Code vs Codex CLI 동일 요청 비교 실험

**같은 코드 리뷰 요청을 두 터미널 AI 에이전트에 보냈을 때 결과가 어떻게 다른가**

---

## 목적

Claude Code와 OpenAI Codex CLI — 둘 다 터미널 기반 AI 코딩 에이전트.  
동일한 요청으로 결과를 비교해서 각 도구의 강점과 성격 차이를 확인.

---

## 실험 환경

| 구분 | Claude Code | Codex CLI |
|------|------------|---------|
| 모델 | Claude Sonnet 4.6 | GPT-5.5 |
| 방식 | 터미널 기반 로컬 에이전트 | 터미널 기반 로컬 에이전트 |
| 설치 | Claude Max 구독 | `npm i -g @openai/codex` (ChatGPT Plus 포함) |

---

## 동일한 요청

```
MovieFinder 프로젝트의 HomeViewModel.kt를 리뷰해줘.
문제점과 개선 방향을 알려줘.
```

---

## 결과 비교

| 구분 | Codex CLI (GPT-5.5) | Claude Code |
|------|-------------------|-------------|
| 발견 이슈 수 | 4개 | **5개** |
| 분석 깊이 | 중간 | 깊음 |
| Fragment 코드 함께 분석 | ✅ | ✅ |
| 개선 코드 예시 | 일부 | **전 이슈 포함** |
| 심각도 분류 | 없음 | **높음/중간/낮음** |
| 응답 속도 | 빠름 | 중간 |

---

## 각 도구가 발견한 것

### 둘 다 발견
- `selectedTab` StateFlow가 Fragment에서 사용 안 됨 (Dead State)
- `lazy` 스레드 안전성 불일치 (3개 SYNCHRONIZED, 1개 NONE)

### Codex CLI만 발견
- `selectedTab`이 `SavedStateHandle`에 보존되지 않음
- 테스트에서 UPCOMING 경로 누락

### Claude Code만 발견
- `watchHistory` 개수 제한 없음 → `.take(20)` 권장
- Fragment의 명령형 `collectJob` 수동 관리 → `flatMapLatest`로 대체 가능

---

## 응답 성격 차이

**Codex CLI:**
- 간결하고 핵심만 짚음
- 관련 파일(Fragment, UseCase)을 자동으로 탐색해서 같이 분석
- `SavedStateHandle` 미사용 같은 Android 아키텍처 관점 강점

**Claude Code:**
- 더 상세하고 코드 예시 풍부
- 심각도 분류 + 우선순위 제시
- `watchHistory.take(20)` 같은 실용적인 개선안 추가 발견

---

## Codex CLI 설치 방법

```bash
# 설치
npm i -g @openai/codex

# 실행 (프로젝트 폴더에서)
cd /프로젝트/폴더
codex

# 종료
/exit 또는 Ctrl+D
```

ChatGPT Plus, Pro, Business 플랜에 포함 (추가 비용 없음)

---

## 느낀 점

두 도구 모두 코드를 직접 읽고 관련 파일을 탐색하는 능력이 있어서 기본적인 리뷰 품질은 비슷했다.

차이는 **성격**에 있었다. Codex는 빠르고 간결하게 핵심을 짚었고, Claude Code는 더 상세하고 개선 코드 예시까지 풍부하게 제공했다.

**실용적인 결론:**
- 빠르게 핵심 이슈만 확인하고 싶을 때 → Codex CLI
- 상세한 분석 + 개선 코드 예시가 필요할 때 → Claude Code
- 둘 다 무료(플랜 포함)라 상황에 따라 번갈아 쓰는 게 현실적

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-05-03*
