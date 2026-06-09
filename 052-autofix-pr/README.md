# 052 - Claude GitHub App + @claude PR 자동 수정 실험

**CI 실패한 PR에 @claude 코멘트 하나로 37초 만에 자동 수정 + 커밋 완료**

---

## 목적

`/autofix-pr` CLI 명령어와 `@claude` PR 코멘트 방식으로 PR 자동 수정 실험.  
CI 파이프라인과 AI 자동 수정이 연결되는 완전한 자동화 흐름 검증.

---

## 실험 흐름

```
PR #81 — WatchHistoryDao 배치 삽입 (실용 변경)
    ↓
PR #82 — 의도적 테스트 실패 (autofix 실험용)
    ↓
@claude fix the failing test 코멘트
    ↓
37초 만에 자동 수정 + 커밋 완료 ✅
```

---

## 설정 과정

| 단계 | 작업 | 결과 |
|------|------|------|
| 1 | GitHub Apps에서 Claude App 설치 | ✅ |
| 2 | claude.ai에서 GitHub 계정 연결 | ✅ |
| 3 | `gh auth refresh -s repo,workflow` | ✅ |
| 4 | `/install-github-app` 실행 → `CLAUDE_CODE_OAUTH_TOKEN` 시크릿 생성 | ✅ |
| 5 | `.github/workflows/claude.yml` 수동 생성 | ✅ |

---

## @claude 작동 결과

**코멘트:**
```
@claude fix the failing test
```

**Claude 응답 (37초):**
```
Fixed BackupRepositoryImplTest.kt:88
— restored the correct expected value 1000L
  (was intentionally set to 9999L)

Commit: dd19a2f
```

테스트 실패 원인 분석 → 파일 읽기 → 수정 → 커밋까지 **37초 만에 자동 완료**.

---

## /autofix-pr CLI vs @claude 코멘트 비교

| 항목 | /autofix-pr CLI | @claude 코멘트 |
|------|:--------------:|:--------------:|
| 작동 여부 | ❌ 버그 | ✅ 정상 |
| 트리거 방법 | 로컬 터미널 | PR 코멘트 |
| 소요 시간 | - | **37초** |
| 권한 제한 | - | OWNER/MEMBER/COLLABORATOR만 가능 |

---

## 핵심 발견

### pre-push 훅과 CI 자동화의 충돌

```
pre-push 훅: 테스트 실패 시 push 차단
    ↓
CI 실패를 만들려면 --no-verify 필요
    ↓
결론: 철저한 pre-push 환경에서 CI 실패는 사실상 발생하지 않음
```

**"보안이 너무 좋아서 실험하기 어려웠다"** — 역설적으로 좋은 개발 환경의 증거.

### /autofix-pr CLI 버그

`/autofix-pr`은 `claude.ai/code/repos/{owner}/{repo}` API로 GitHub App 설치 여부를 확인하는데, 실제 설치됐음에도 `github_app_not_installed`를 반환하는 서버 측 버그가 있음.

→ Anthropic 버그 리포트 대상

---

## claude.yml 워크플로우 보안 설정

```yaml
if: |
  github.event.comment.author_association == 'OWNER' ||
  github.event.comment.author_association == 'MEMBER' ||
  github.event.comment.author_association == 'COLLABORATOR'
```

외부 유저가 `@claude` 코멘트로 코드 실행하는 것을 차단.

---

## 느낀 점

설정 과정이 복잡했지만, 한 번 세팅하면 PR에서 `@claude` 코멘트 하나로 AI가 직접 코드를 고쳐준다.

010일차 GitHub Actions PR 코드 리뷰가 "문제를 발견하고 코멘트"였다면, 이번 실험은 "문제를 발견하고 직접 수정까지" 한 단계 더 나아간 것이다.

**"AI가 코드 리뷰어에서 코드 수정자로 진화했다."**

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-06-09*
