# 070 - Claude Code 커스텀 Subagent 직접 설계 — "AI 이용"에서 "AI 설계"로

**069개 실험이 전부 "이미 있는 AI 기능을 호출"하는 것이었다면, 070일차는 처음으로 "AI의 역할과 판단 기준 자체를 내가 정의"한 실험**

---

## 목적

지금까지(001~069일차)는 `/ultrawork`, `/sciomc`, Exa MCP처럼 **이미 만들어진 AI 기능을 호출**하는 실험이었다.  
070일차는 처음으로 **나만의 커스텀 서브에이전트를 직접 설계**해서, "AI에게 역할·규칙·출력 형식을 부여하는 법"을 배운다.

---

## 핵심 개념 — "AI를 이용" vs "AI를 설계"

| 구분 | 정의 | 예시 |
|---|---|---|
| **AI를 이용 (Using AI)** | 이미 존재하는 AI 기능에게 작업을 시키고 결과를 받음 | `/ultrawork`로 병렬 작업, Exa MCP로 리서치 — 지금까지 069개 실험 대부분 |
| **AI를 설계 (Building AI)** | AI가 어떻게 판단할지(역할, 체크리스트, 출력 형식)를 내가 직접 정의 | **오늘 만든 `kotlin-architecture-reviewer` 서브에이전트** |

> 비유하면: 이용은 "이미 있는 직원에게 업무 지시", 설계는 "새 직원을 채용하고 업무 매뉴얼을 처음부터 작성"하는 것.

---

## Subagent 정의 파일의 구조

`.claude/agents/kotlin-architecture-reviewer.md`

```markdown
---
name: kotlin-architecture-reviewer
description: MovieFinder 프로젝트 전담 리뷰어. Kotlin 코루틴/Flow 안전성과
  Clean Architecture 레이어 규칙만 리뷰한다 — 코드 스타일, 네이밍,
  일반 보안(OWASP), UX/접근성, 테스트 커버리지는 다루지 않는다.
tools: Read, Grep, Glob, Bash
model: opus
---
# 역할
당신은 MovieFinder 프로젝트 전담 아키텍처 리뷰어입니다.
이 프로젝트의 `CLAUDE.md`에 문서화된 규칙만 기준으로 삼습니다.
...
```

| 필드 | 역할 |
|---|---|
| `name` | 이 서브에이전트를 호출할 때 쓰는 식별자 |
| `description` | Claude Code가 "언제 이 에이전트를 자동으로 골라야 하는지" 판단하는 기준 |
| `tools` | 이 에이전트가 쓸 수 있는 도구 화이트리스트 (여기선 읽기/검색/bash만 — 수정 권한 없음) |
| `model` | 이 에이전트 전용 모델 지정 가능 (여기선 opus) |
| 본문 (frontmatter 아래) | 시스템 프롬프트 — 역할, 체크리스트, 출력 형식을 정의 |

---

## 트러블슈팅 — 파일을 만들었는데 왜 바로 안 되나

```
Error: Agent type 'kotlin-architecture-reviewer' not found
```

**원인:** `.claude/agents/`의 프로젝트 서브에이전트는 **세션 시작 시점에 한 번만 스캔**된다. 세션 도중에 새로 만든 파일은 자동으로 인식되지 않는다.

**해결:**
```
/reload-plugins
```
→ `Reloaded: 19 plugins · 17 skills · 42 agents · 41 hooks · ...`  
→ 이후 새 에이전트가 목록에 등록되어 정상 호출됨.

> **배운 점:** Claude Code의 확장 요소(agents, hooks, skills)는 대부분 "세션 시작 시 로드"되는 구조다. 런타임 중 추가한 건 리로드 명령이 필요하다.

---

## 설계 원칙 4가지 (왜 이렇게 만들었나)

### 1. 스코프를 의도적으로 좁혔다

일반 `code-reviewer` 계열 에이전트는 보안·스타일·SOLID·성능·테스트를 전부 다루도록 설계되어 있다. 그러면 "레이어를 깼는지", "Mutex를 스코프 밖에서 잠갔는지" 같은 **이 프로젝트 고유의 규칙**은 20개 체크리스트 중 하나로 묻혀서 놓치기 쉽다.

→ 프롬프트 맨 위에 **"하지 않는 것" 목록**을 명시하고, 스코프 밖 관찰은 "언급도 하지 말라"고 지시했다. 넓은 리뷰의 노이즈를 없애는 게 목적.

### 2. 체크리스트를 CLAUDE.md 원문에서 그대로 뽑아 넣었다

```
- launchWithErrorHandler, Channel.CONFLATED, Mutex.tryLock()은
  반드시 viewModelScope.launch{} 안에서
- SharingStarted.Lazily vs WhileSubscribed5s 구분
```

이 프로젝트 문서에만 있는 관례를 항목화했고, **에이전트가 실행 시점에 CLAUDE.md를 다시 Read하도록 지시**해서 문서가 나중에 갱신돼도 체크리스트가 자동으로 최신을 따라가게 만들었다.

### 3. "자매 코드와 대조"를 강제했다 — 그리고 실제로 실패를 발견했다

문서 문구만 기계적으로 대조하면 "예전엔 맞았지만 지금은 아닌" 규칙을 오탐으로 잡을 위험이 있다. (아래 실제 사례 참고)

### 4. 출력 형식을 강제했다

심각도 3단계(Critical/Major/Minor) + 근거 코드 라인 인용을 **"생략 불가"**로 못박았다. "전반적으로 좋아 보입니다" 같은 내용 없는 응답을 원천 차단.  
통과한 체크리스트 항목도 표에 남겨서 **"뭘 확인했는지"가 "뭘 놓쳤는지"만큼 보이게** 했다 — 위반이 없을 때 침묵하는 리뷰어와, 확인 후 통과시킨 리뷰어를 구분하기 위해서.

---

## 실전 실행 — HomeViewModel.kt 리뷰 결과

| 체크 항목 | 결과 |
|---|---|
| A1: Presentation→Data 직접 참조 금지 | ✅ ViewModel/Fragment 모두 UseCase·core.util만 참조 |
| B1: CancellationException 재throw | ✅ MovieRemoteMediator:95 |
| B6: SharingStarted 선택 | ✅ watchHistory에 WhileSubscribed5s — 시청기록 리스트 특성상 적절 |
| C1: RemoteMediator 오프라인 즉시 반환 | ✅ MovieRemoteMediator:33, 52 |
| B8: withExponentialBackoff 재사용 | ✅ MovieRemoteMediator:103 |

**최초 리포트:** Major 1건 — `flatMapLatest`(line 49)에 `@OptIn(ExperimentalCoroutinesApi::class)` 누락, 자매 ViewModel 4개는 전부 붙어있음.

---

## 발견된 한계 — "자매 코드 대조"만으로는 부족했다

에이전트가 지적한 근거(자매 파일 대조) 자체는 정확했다. 하지만 **직접 컴파일해서 검증**해보니:

```bash
./gradlew :app:compileDebugKotlin  # opt-in 없이도 정상 컴파일됨
```

**진짜 원인:** `flatMapLatest`는 `kotlinx.coroutines` 1.7.0부터 experimental 딱지가 떨어졌는데, 이 프로젝트는 1.11.0을 쓴다. 즉 자매 ViewModel들의 `@OptIn`은 **옛날 coroutines 버전 시절의 유물**이 남아있는 것이고, HomeViewModel.kt는 오히려 불필요한 보일러플레이트 없이 최신 상태였다.

**최종 결과 (검증 후): Critical 0 / Major 0 / Minor 0**

> **핵심 교훈:** "자매 코드와 비교"는 유용한 휴리스틱이지만, **자매 코드 자체가 레거시일 가능성**을 놓칠 수 있다. 다음 개선으로 "experimental API 관련 지적은 실제 컴파일로 확인 후 보고"라는 안전장치를 체크리스트에 추가하는 게 필요하다.

---

## 일반 리뷰 요청 vs 커스텀 서브에이전트 비교

| 항목 | 일반 리뷰 요청 ("이 코드 리뷰해줘") | 커스텀 Subagent |
|---|---|---|
| 질문의 성격 | "이 코드 괜찮아?"에 대한 폭넓은 의견 | "이 프로젝트가 스스로 정한 규칙을 지키는가"에 대한 좁고 기계적인 감사(audit) |
| 스코프 | 보안/스타일/성능/테스트 전반 | CLAUDE.md에 명시된 규칙만 |
| 재사용성 | 매번 프롬프트를 새로 씀 | 한 번 정의하면 계속 호출 가능한 "도구"가 됨 |
| 출력 일관성 | 매번 다를 수 있음 | 형식(심각도+근거+통과표)이 강제됨 |
| 한계 | 광범위해서 프로젝트 고유 규칙을 놓치기 쉬움 | 좁아서 "규칙 자체가 낡았는지"는 놓칠 수 있음 (오늘 실제로 확인) |

---

## 오늘 배운 것 — 한 줄 정리

1. Subagent는 `.claude/agents/*.md`에 **frontmatter(name/description/tools/model) + 시스템 프롬프트**로 정의한다.
2. 세션 도중 새로 만든 에이전트는 `/reload-plugins`로 리로드해야 인식된다.
3. 좋은 서브에이전트 설계는 **"무엇을 하지 않는지"를 먼저 못박는 것**에서 시작한다 (스코프 좁히기).
4. 체크리스트를 프로젝트 문서에서 그대로 가져오고, **실행 시점에 문서를 다시 읽게** 하면 최신성이 유지된다.
5. "과거 코드와 비교"라는 휴리스틱은 **과거 코드가 이미 낡았을 가능성**을 놓칠 수 있다 — 자동화된 판단은 항상 실제 검증(컴파일, 테스트)으로 이중 확인이 필요하다.
6. 출력 형식을 강제하면 "침묵하는 리뷰어"와 "확인하고 통과시킨 리뷰어"를 구분할 수 있다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-07-02*
