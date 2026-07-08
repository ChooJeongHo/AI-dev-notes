# 071 - Subagent 개선 + PreToolUse 훅 자동 연동 — "차단"과 "정보 제공"은 다른 문제다

**070일차에서 발견한 오탐을 실제로 고치고, ViewModel 파일 수정 시 커스텀 리뷰어가 자동으로 미리 검토해주는 시스템을 완성했다**

---

## 목적

070일차 리뷰 결과에서 "다음 개선으로 권장드립니다"라고 남겨둔 것을 **바로 다음 날 실제로 구현**한다.
1. 오탐(false positive) 방지 안전장치를 서브에이전트 자체에 내장
2. 그 서브에이전트를 PreToolUse 훅에 연결해서, 사람이 매번 호출 안 해도 자동으로 사전 검토가 돌게 만든다

---

## 작업 1 — 서브에이전트에 자기 검증 규칙 추가

070일차의 오탐 원인: `@OptIn` 관련 지적을 **실제 컴파일 없이** "자매 코드와 다르다"는 이유만으로 보고했다.

**추가한 규칙 (`.claude/agents/kotlin-architecture-reviewer.md`):**

```markdown
6. 오탐 방지 안전장치 (experimental API / @OptIn 관련 지적 전용):
   - @OptIn 관련 지적을 하려면 반드시 ./gradlew :app:compileDebugKotlin 먼저 실행
   - 컴파일 성공 → 그 지적은 보고서에서 완전히 제외 (침묵 = 정상)
   - 컴파일 실패 + 실제 연관 있음 → 지적 포함, 컴파일러 에러 메시지 인용
   - 이 검증은 @OptIn류에만 적용 (레이어 위반, 코루틴 안전성 등은 그대로 보고)
   - 매번 무조건 컴파일하지 않음 — 해당 지적이 없으면 이 단계 자체를 건너뜀
```

> **왜 "전면 재검증"이 아니라 "해당 유형에만" 적용했나?** 레이어 위반(`Presentation→Data 직접 참조`)이나 `CancellationException` 재throw 누락 같은 건 **컴파일 여부와 무관한 설계/런타임 결함**이다. 컴파일이 성공해도 여전히 나쁜 코드다. 반대로 `@OptIn` 문제는 "컴파일러가 요구하는가 아닌가"로 100% 결정되는 성질이라, 이 유형만 컴파일 검증으로 자동 필터링할 수 있다.

---

## 작업 2 — PreToolUse 훅으로 서브에이전트 자동 호출

### 만든 파일 3개

| 파일 | 역할 |
|---|---|
| `.claude/hooks/pre-edit-architecture-review.sh` | 메인 훅 — 조건 확인 + 서브에이전트 헤드리스 호출 |
| `.claude/hooks/lib/reconstruct_edit.py` | Edit/Write의 `tool_input`을 파싱해서 "적용될 예정인 내용"을 임시 파일로 재구성 |
| `.claude/settings.json` | PreToolUse 훅 등록 (matcher: `Edit\|Write`) |

### 동작 흐름

```
domain/*.kt 또는 presentation/**/ViewModel.kt 파일에 Edit/Write 시도
    ↓
reconstruct_edit.py가 "적용 예정 내용"을 임시 파일로 재구성
    ↓
claude --agent kotlin-architecture-reviewer -p 로 헤드리스 호출
    ↓
리뷰 결과를 hookSpecificOutput.additionalContext로 Claude Code에 전달
    ↓
permissionDecision: "allow" (항상 허용 — 편집 자체는 막지 않음)
```

### 핵심 기법 — `claude --agent` 플래그로 서브에이전트 재사용

훅은 셸 스크립트일 뿐이라 서브에이전트를 코드로 직접 "호출"할 수 없다. 대신:

```bash
claude --agent kotlin-architecture-reviewer -p \
    --allowedTools "Read,Grep,Glob,Bash(./gradlew :app:compileDebugKotlin:*)" \
    --add-dir "$tmp_dir" \
    < "$prompt_file"
```

`.md` 파일에 정의해둔 **같은 에이전트(tools/model/프롬프트)를 헤드리스 세션에 그대로 로드**해서 재사용한다.

### 왜 `--allowedTools`로 권한을 좁혔나 (이중 안전장치)

```bash
--allowedTools "Read,Grep,Glob,Bash(./gradlew :app:compileDebugKotlin:*)"
```

작업 1에서 "compileDebugKotlin만 실행하라"는 규칙을 **프롬프트로** 적어뒀지만, `--permission-mode bypassPermissions`를 쓰면 서브에이전트가 임의의 bash 명령을 승인 없이 실행할 수 있게 된다. 즉 프롬프트 규칙이 실수로 깨져도, **툴 권한 레벨에서 한 번 더 막아주는 구조**다.

> **교훈:** "AI에게 규칙을 프롬프트로 알려주는 것"과 "AI가 실제로 그 규칙 밖으로 못 나가게 시스템으로 강제하는 것"은 다른 안전장치다. 둘 다 있어야 진짜 안전하다.

---

## 트러블슈팅 — macOS엔 GNU `timeout`이 없다

```bash
# 처음 구현: 즉시 exit 127 (command not found)
printf '%s' "$prompt" | timeout 240 claude ...
```

macOS 기본 환경엔 GNU coreutils의 `timeout`이 없다. 순수 bash로 재구현:

```bash
( claude --agent ... < "$prompt_file" ) > "$review_file" 2>&1 &
claude_pid=$!

( sleep 240 && kill -TERM "$claude_pid" 2>/dev/null ) &
watcher_pid=$!

wait "$claude_pid" 2>/dev/null
kill "$watcher_pid" 2>/dev/null   # claude가 먼저 끝나면 워처도 정리
```

**"메인 프로세스를 백그라운드로 띄우고, 워처가 시간 초과 시 죽인다"**는 패턴. 실제로 이 버그는 작업 3의 실전 테스트 중에 잡혔다.

---

## 작업 3 — 검증에서 발견한 예상 밖 제약

### 제약: 훅은 세션 시작 시에만 로드된다 (070일차와 동일한 문제)

지금 세션에서 방금 등록한 훅은, 지금 세션에서 실제로 Edit 도구를 써도 **발동하지 않는다.** (070일차에서 `/reload-plugins`가 필요했던 것과 같은 근본 원인)

**해결(우회):** 훅이 받을 실제 PreToolUse JSON 페이로드를 직접 구성해서 스크립트에 파이프로 흘려보내는 방식으로 검증.

```
DetailViewModel.submitTmdbRating()의 ratingMutex.tryLock()을
viewModelScope.launch 바깥으로 옮기는 — 실제 CLAUDE.md 위반(B2)을 
일부러 주입한 가짜 diff를 구성해서 테스트
```

### 검증 결과 (2분 16초, 실제 opus 헤드리스 호출)

| 항목 | 결과 |
|---|---|
| 대상 파일 매칭 | ✅ `presentation/detail/DetailViewModel.kt` → `ViewModel.kt` 패턴 정확히 매치 |
| diff 재구성 | ✅ `tryLock`이 `launch` 밖으로 나온 버전이 임시 파일에 정확히 반영됨 |
| 주입한 B2 위반 탐지 | ✅ Critical로 정확히 잡음 — 같은 파일의 `loadMovieDetail()`, `PersonDetailViewModel`과 대조하며 근거 제시 |
| **작업 1 규칙 적용 확인** | ✅ "No @OptIn concerns present, so per procedure step 6 I skip the Gradle compile gate" — 새로 추가한 규칙을 정확히 인지하고 스스로 건너뜀 |
| 미주입 이슈 자체 발견 | ✅ 주입하지 않은 기존 이슈(B7, `saveWatchHistory` 격리 미흡)도 스스로 찾아 Minor 보고 — 파일 전체를 다시 읽고 판단했다는 증거 |

---

## 작업 4 — 왜 "차단"이 아니라 "정보 제공"인가 (핵심 개념)

054일차와 071일차 훅은 겉모습(PreToolUse, Edit/Write 매칭)은 똑같지만 **성격이 근본적으로 다르다.**

| | 054일차 (`domain-android-import-guard.sh`) | 071일차 (이번 훅) |
|---|---|---|
| 판단 방식 | `grep -E '^import (android\|androidx)\.'` — 정규식 | opus 모델이 코드를 읽고 판단 |
| 판단 속도 | 몇 밀리초, 결정론적 | 2분 16초, 확률적 |
| 오탐 가능성 | **구조적으로 불가능** (import 문 존재 여부는 100% 확실) | **항상 존재** (작업 1의 안전장치도 "줄이는" 것이지 "0으로 만드는" 게 아님) |
| 성격 | **검사** (test) | **의견** (opinion) |
| 설계 | `exit 2` 즉시 차단 | `permissionDecision: allow` + `additionalContext` |
| 비유 | 컴파일러 에러 | 옆자리 시니어의 훈수 |

### 설계 원칙

> **결정론적이고 항상 참인 규칙만 차단하고, 판단이 필요한 리뷰는 정보 제공으로 남긴다.**

정보 제공 방식의 장점: 리뷰가 틀려도 실제 손해가 없다. 메인 세션의 Claude가 그 내용을 참고해서 **고칠지 무시할지 최종 판단**을 내릴 수 있다. 이 훅은 "게이트"가 아니라 "훈수 두는 옆자리 시니어"에 가깝게 설계한 것.

만약 이 훅도 `exit 2`로 차단하게 만들었다면? opus가 오판했을 때(070일차 실제로 있었던 일) 정당한 코드 수정까지 매번 막혀서 개발 흐름 자체가 망가진다.

---

## 070일차 → 071일차 발전 요약

```
070일차: Subagent를 "만들기"만 함 → 오탐 1건 발견 (수동 검증으로 확인)
    ↓
071일차: (1) 오탐을 막는 규칙을 Subagent 자체에 내장
         (2) Subagent를 Hook에 연결해서 "자동으로 매번 도는" 시스템 완성
         (3) 검증 중 예상 못한 제약(세션 로드 시점) 발견 + 우회
         (4) "차단"과 "정보 제공"이라는 훅 설계의 두 축을 명확히 구분
```

---

## 오늘 배운 것 — 한 줄 정리

1. AI 산출물의 신뢰도를 높이는 방법은 **"AI가 스스로 자기 지적을 재검증하는 규칙"을 프롬프트에 넣는 것**이다.
2. 프롬프트 규칙만으로는 부족하다 — **`--allowedTools`로 실제 권한을 좁히는 이중 안전장치**가 필요하다.
3. 훅에서 서브에이전트를 재사용하려면 `claude --agent <name> -p` 헤드리스 호출로 **같은 정의를 그대로 로드**한다.
4. macOS엔 GNU `timeout`이 없다 — `sleep N && kill` 워처 패턴으로 직접 구현해야 한다.
5. **결정론적 판단(정규식 등)은 차단해도 안전**하지만, **AI의 판단(코드 리뷰 등)은 정보 제공으로 남기는 게 안전**하다 — 이 구분이 훅 설계의 핵심 기준이다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-07-03*
