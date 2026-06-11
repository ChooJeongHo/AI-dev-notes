# 054 - Claude Code PreToolUse 훅 — Domain 레이어 Android 의존성 차단

**파일 수정 전에 Domain 레이어에 Android import가 추가되는 것을 차단 — Clean Architecture 자동 수호**

---

## 목적

053일차 PostToolUse 훅(수정 후 Detekt 검사)의 짝꿍.  
PreToolUse 훅으로 **수정 전에** Domain 레이어 위반을 원천 차단.

---

## PreToolUse vs PostToolUse 비교

| 항목 | PostToolUse (053) | PreToolUse (054) |
|------|:-----------------:|:----------------:|
| 실행 시점 | 파일 수정 **후** | 파일 수정 **전** |
| 역할 | 코드 품질 검사 | 위반 원천 차단 |
| exit 2 효과 | 경고 주입 | **수정 자체 차단** |
| 대상 | 모든 .kt 파일 | Domain 레이어만 |

---

## 만든 훅: domain-android-import-guard.sh

```
Edit/Write 도구로 domain/*.kt 파일 수정 시도
    ↓
PreToolUse 훅 자동 트리거
    ↓
추가되는 텍스트에 android.* / androidx.* import 감지
    ↓
있으면 exit 2 → 수정 차단 + Claude Code에 경고
없으면 exit 0 → 정상 통과
```

---

## settings.json 설정

```json
"PreToolUse": [
  {
    "matcher": "Edit|Write",
    "hooks": [
      {
        "type": "command",
        "command": "/Users/serveace/.claude/hooks/domain-android-import-guard.sh"
      }
    ]
  }
]
```

---

## 검증 결과

| 케이스 | 결과 |
|--------|------|
| domain/*.kt + `import androidx.*` | ⛔ exit 2 (차단) |
| domain/*.kt + `import android.*` | ⛔ exit 2 (차단) |
| domain/*.kt + 순수 Kotlin import | ✅ exit 0 (허용) |
| data/*.kt + Android import | ✅ exit 0 (허용) |
| domain/README.md | ✅ exit 0 (허용) |

---

## 차단 메시지 예시

```
⛔ [Domain Layer Guard] Android 의존성 위반 — 수정이 차단되었습니다.

파일: domain/usecase/GetMovieUseCase.kt
위반 import:
  import androidx.lifecycle.ViewModel

Domain 레이어는 순수 Kotlin/Java만 허용됩니다.
Android 프레임워크 의존성은 Presentation/Data 레이어로 이동하세요.
```

---

## 핵심 발견 — set -euo pipefail 제거

```bash
# 위험: pipefail + grep 조합
set -euo pipefail
violations=$(echo "$content" | grep -E '^import' || true)
# grep이 매치 없으면 exit 1 → pipefail로 스크립트 비정상 종료

# 안전: 명시적 if/exit 패턴
violations=$(printf '%s' "$content" | grep -E '^import' || true)
if [[ -n "$violations" ]]; then ...
```

훅처럼 다양한 입력을 받아 조건 분기하는 스크립트에서는 **명시적 if/exit 패턴이 더 안전**하다.

---

## 053 + 054 훅 조합 — 완전한 품질 게이트

```
파일 수정 시도
    ↓
PreToolUse: Domain 레이어 Android import? → 차단
    ↓
파일 수정 실행
    ↓
PostToolUse: Detekt 오류? → 경고 주입
    ↓
커밋
    ↓
pre-commit: Detekt 전체 프로젝트 검사
```

**수정 전 → 수정 후 → 커밋 전** 3단계 품질 게이트 완성.

---

## 느낀 점

PostToolUse가 "이미 일어난 일을 검사"라면, PreToolUse는 "일어나기 전에 막는다"는 개념이다.

050일차 sciomc에서 레이어 준수율 99.2%를 확인했는데, 이 훅이 있었다면 처음부터 100%를 유지할 수 있었을 것이다.

**"AI가 코드를 수정하기 전에 규칙을 확인하고 스스로 멈추는 구조" — 개발자가 설계한 규칙을 AI가 자율적으로 지킨다.**

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-06-11*
