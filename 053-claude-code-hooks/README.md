# 053 - Claude Code Hooks 실험 — .kt 파일 수정 시 Detekt 자동 실행

**파일 수정할 때마다 Detekt가 자동으로 돌아서 커밋 전에 오류를 미리 잡는다 — ~800ms**

---

## 목적

Claude Code Hooks를 직접 만들어서 개발 워크플로우에 통합.  
기존 pre-commit 훅은 커밋 시점에만 Detekt를 실행하는데, **파일 수정 시점에 즉시** 실행하도록 개선.

---

## Claude Code Hooks란?

Claude Code가 특정 도구를 실행할 때 자동으로 트리거되는 스크립트.

| 훅 타입 | 트리거 시점 |
|---------|-----------|
| `UserPromptSubmit` | 사용자 프롬프트 제출 시 |
| `PostToolUse` | 도구 실행 후 |
| `PreToolUse` | 도구 실행 전 |

**exit 2로 종료하면 stdout이 Claude Code에 `system-reminder`로 주입** → 즉각 인식.

---

## 만든 훅: detekt-on-save.sh

```
Edit/Write 도구로 .kt 파일 수정
    ↓
PostToolUse 훅 자동 트리거
    ↓
Gradle init script로 해당 파일만 Detekt 분석
    ↓
오류 있으면 exit 2 → Claude Code에 즉시 알림
오류 없으면 exit 0 → 조용히 통과
```

---

## settings.json 설정

```json
"PostToolUse": [
  {
    "matcher": "Edit|Write",
    "hooks": [
      {
        "type": "command",
        "command": "/Users/serveace/.claude/hooks/detekt-on-save.sh"
      }
    ]
  }
]
```

---

## 핵심 기술 — Gradle init script

처음엔 Detekt CLI jar로 실행하려 했지만 dev.detekt 2.x는 fat jar(-all.jar)가 없어서 실패.  
Gradle init script 방식으로 해결:

```groovy
gradle.projectsEvaluated {
    rootProject.allprojects {
        tasks.matching { it.name == 'detekt' }.configureEach {
            setSource(files(System.getProperty('detekt.file')))
        }
    }
}
```

프로젝트 파일 수정 없이 **단일 파일만** 분석하도록 Detekt 소스를 오버라이드.

---

## 검증 결과

| 케이스 | 결과 |
|--------|------|
| 정상 .kt 파일 수정 | exit 0, 무출력 (조용히 통과) |
| 오류 있는 .kt 파일 수정 | exit 2 + 오류 라인 → Claude Code 즉시 알림 |
| .xml / 없는 파일 | exit 0, 조용히 스킵 |

**실행 속도: ~800ms** (Gradle 데몬 워밍 시)

---

## 기존 훅과 비교

| 항목 | pre-commit 훅 | PostToolUse 훅 (이번 실험) |
|------|:------------:|:-------------------------:|
| 트리거 시점 | 커밋 시 | **파일 수정 직후** |
| 분석 범위 | 전체 프로젝트 | **수정된 파일만** |
| 실행 속도 | ~30초 | **~800ms** |
| 오류 발견 시점 | 커밋 직전 | **수정 즉시** |

---

## macOS 호환 처리

macOS에는 `timeout` 명령이 없어서 자동 감지 래퍼 구현:

```bash
_timeout() {
  if command -v gtimeout &>/dev/null; then gtimeout "$secs" "$@"
  elif command -v timeout &>/dev/null; then timeout "$secs" "$@"
  else "$@"
  fi
}
```

---

## 느낀 점

pre-commit 훅은 "커밋 시 마지막 관문"이지만, PostToolUse 훅은 "수정할 때마다 즉시 피드백"이라 훨씬 빠른 개발 루프가 만들어진다.

특히 **800ms 만에 수정된 파일만 분석**한다는 게 인상적이었다. 전체 프로젝트 Detekt(~30초)와 비교하면 37배 빠르다.

**"AI가 코드를 수정할 때마다 품질 검사가 자동으로 따라온다" — 개발자가 설정만 하면 AI 스스로 품질을 지킨다.**

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-06-10*
