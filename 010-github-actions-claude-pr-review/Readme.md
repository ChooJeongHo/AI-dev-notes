# AI 활용 기록 010 - GitHub Actions + Claude API PR 자동 코드 리뷰

> PR이 올라오면 Claude API가 자동으로 코드 리뷰 코멘트를 달아주는 워크플로우 구축

## 목적

GitHub Actions와 Claude API를 연동하여 PR 생성/업데이트 시 자동으로 android-code-review 스킬 기준 코드 리뷰를 수행하고, 결과를 PR 코멘트로 자동 게시하는 CI 워크플로우 구축

## 사용한 도구

- GitHub Actions (`.github/workflows/claude-pr-review.yml`)
- Claude API (`claude-sonnet-4-20250514`)
- android-code-review SKILL.md (3일차 작성)
- 대상 레포: [MovieFinder](https://github.com/ChooJeongHo/MovieFinder)

---

## 워크플로우 구조

```
PR 생성/업데이트
    ↓
.kt / .xml 변경 파일 감지
    ↓ (변경 없으면 스킵)
PR diff 추출 (gh pr diff)
    ↓
Claude API 호출 (android-code-review 6개 카테고리 기준)
    ↓
PR 코멘트 자동 게시 (이전 리뷰 삭제 후 갱신)
```

### 주요 설정

| 항목 | 내용 |
|------|------|
| 트리거 | PR opened / synchronize → main |
| 리뷰 대상 | `.kt`, `.xml` 파일만 |
| Claude 모델 | claude-sonnet-4-20250514 |
| 중복 방지 | PR 업데이트마다 이전 리뷰 코멘트 삭제 후 새로 게시 |
| 동시 실행 방지 | concurrency로 동일 PR 중복 실행 자동 취소 |

### 필요한 GitHub Secrets

- `ANTHROPIC_API_KEY` - Claude API 키

---

## 구축 과정

### 1단계 - ANTHROPIC_API_KEY 등록

GitHub Settings 탭이 브라우저 창 너비에 따라 잘려서 보이지 않는 경우 URL로 직접 접근:

```
https://github.com/{owner}/{repo}/settings/secrets/actions/new
```

### 2단계 - 워크플로우 파일 생성

Claude Code가 android-code-review SKILL.md를 자동으로 읽고 6개 카테고리 기준으로 리뷰 프롬프트를 구성하여 `.github/workflows/claude-pr-review.yml` 생성 (192줄)

### 3단계 - 테스트 PR 생성

```bash
git checkout -b test/claude-pr-review
# DetailViewModel.kt에 주석 한 줄 추가
git push origin test/claude-pr-review
gh pr create --title "test: Claude PR 코드 리뷰 워크플로우 테스트"
```

---

## 에러 수정 과정

### 에러 1 - gh pr diff 인자 오류

```
accepts at most 1 arg(s), received 2
```

`gh pr diff`에 파일 목록을 인자로 넘기는 방식이 지원되지 않음  
→ 전체 diff를 가져온 후 awk로 `.kt/.xml` 파일만 필터링하는 방식으로 수정

### 에러 2 - 셸 파싱 오류

```
heredoc 안의 ()가 셸 $() 파싱과 충돌
```

리뷰 프롬프트 heredoc 내부의 괄호가 셸 변수 확장과 충돌  
→ 프롬프트를 별도 파일(`review_prompt.txt`)로 분리하고 `jq --rawfile`로 읽는 방식으로 수정

---

## 실행 결과

### PR #1 코드 리뷰 자동 생성 확인

`github-actions bot`이 PR에 자동으로 달아준 코드 리뷰:

```
🤖 Claude AI Code Review

Category          | Grade | Notes
Architecture      | N/A   | No architectural changes made
Coroutine Safety  | N/A   | No coroutine-related changes
Memory Management | N/A   | No memory management changes
Error Handling    | N/A   | No error handling changes
Android Best Practices | N/A | No functional changes to evaluate
Code Quality      | B     | Comment placement violates Kotlin style conventions

Issues Found:
DetailViewModel.kt:3 - Comment should be placed after imports, not before them.

Overall Grade: B

Automated review by Claude API | android-code-review criteria
```

- **Claude PR Code Review / review**: ✅ Successful in 17s
- **GitGuardian Security Checks**: ✅ No secrets detected

---

## 3일차 스킬과의 연결

| 구분 | 3일차 | 10일차 |
|------|-------|--------|
| 방식 | Claude Code CLI에서 수동 리뷰 요청 | GitHub Actions에서 자동 리뷰 |
| 트리거 | 개발자가 직접 요청 | PR 생성/업데이트 시 자동 |
| 결과 위치 | 터미널 출력 | PR 코멘트 자동 게시 |
| 기준 | android-code-review SKILL.md | 동일 |

3일차에 만든 스킬이 10일차 자동화의 기반이 됨

---

## 느낀 점

Claude Code가 SKILL.md를 읽고 리뷰 기준을 자동으로 프롬프트에 반영했다.  
에러도 스스로 로그를 분석하고 수정 방향을 잡아서 재시도까지 했다.

GitHub Actions + Claude API 조합은 코드 품질 관리를 자동화하는 데 실용적이다.  
PR마다 사람이 직접 리뷰하기 어려운 환경에서 1차 필터 역할로 충분히 활용 가능하다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*PR #1: https://github.com/ChooJeongHo/MovieFinder/pull/1*  
*작성일: 2026-03-30*
