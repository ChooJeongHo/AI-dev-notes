# AI Dev Notes

> **21일, 21개 실험** — Claude Code / MCP / GitHub Actions로 Android 개발 워크플로우를 자동화한 실험 기록

AI 도구를 활용한 개발 실험과 학습 기록 저장소

Claude Code 등 AI 기반 개발 도구를 활용한 실험 내용을 정리합니다.

---

## 포트폴리오

→ [AI를 활용한 Android 개발 워크플로우 자동화 구축기](./PORTFOLIO.md)

---

## 📊 핵심 성과 요약

| 지표 | 수치 |
|---|---|
| 총 실험 기간 | 21일 |
| 실험 수 | 21개 |
| JaCoCo 테스트 커버리지 달성 | **74.8%** |
| ultra think 자동 발견 이슈 | **30개+** (4개 에이전트 병렬 분석) |
| 자동화 구축 항목 | PR 코드 리뷰, 릴리즈 노트, 취업 지원, 보안 취약점 분석 |
| 실험 대상 | [MovieFinder](https://github.com/ChooJeongHo/MovieFinder) (Android) |

---

## 기록 목록

| # | 실험 | 핵심 결과 |
|---|---|---|
| [001](./001-moviefinder-ai-workflow/) | MovieFinder AI Workflow — Claude Code로 영화 추천 앱 만들기 | AI 기반 첫 앱 구현 기준점 |
| [002](./002-claude-code-skill-analysis/) | Claude Code Skill Analysis — 스킬 분석 실험 | 스킬 유무에 따른 코드 품질 차이 확인 |
| [003](./003-android-codereview-skill-validation/) | Android 코드리뷰 스킬 실전 검증 | 스킬 적용 시 리뷰 깊이 향상 |
| [004](./004-moviefinder-stats-feature/) | MovieFinder 시청 통계 기능 추가 | 1일차 대비 AI 워크플로우 성장 측정 |
| [005](./005-ohmyclaudecode-ulw-experiment/) | oh-my-claudecode ulw 모드 실험 | 일반 모드 vs ulw 접근 방식 비교 |
| [006](./006-ohmyclaudecode-team-experiment/) | oh-my-claudecode /team 멀티 에이전트 실험 | 3개 에이전트 레이어별 병렬 작업 |
| [007](./007-ohmyclaudecode-ralph-experiment/) | oh-my-claudecode ralph 모드 실험 | ulw / team / ralph 3부작 종합 비교 |
| [008](./008-claudemd-experiment/) | CLAUDE.md 활용 실험 | CLAUDE.md 유무에 따른 코드 생성 차이 |
| [009](./009-saramin-auto-apply/) | playwright-cli 사람인 자동 지원 시스템 | **취업 지원 자동화 파이프라인 구축** |
| [010](./010-github-actions-claude-pr-review/) | GitHub Actions + Claude API PR 자동 코드 리뷰 | PR 생성 → 자동 리뷰 코멘트 |
| [011](./011-context7-mlkit-tag-suggestion/) | Context7 MCP + ML Kit 영화 태그 AI 추천 | 실시간 공식 문서 참조로 구현 |
| [012](./012-jacoco-coverage-improvement/) | Claude Code로 테스트 커버리지 분석 및 개선 | **JaCoCo 74.8% 달성** |
| [013](./013-github-mcp-issue-automation/) | GitHub MCP로 코드 분석 및 이슈 자동 등록 | 코드 품질 분석 → 이슈 자동 생성 |
| [014](./014-github-mcp-issue-to-pr/) | GitHub MCP로 이슈 → 코드 수정 → PR 자동화 | 이슈 읽기부터 PR 생성까지 전체 자동화 |
| [015](./015-release-notes-automation/) | GitHub MCP + gh CLI로 릴리즈 노트 자동 생성 | 커밋 분석 → v1.0.0 릴리즈 노트 자동 작성 |
| [016](./016-exa-mcp-library-security-check/) | Exa MCP로 라이브러리 보안 취약점 분석 | 실시간 웹 검색으로 실제 버그 발견 |
| [017](./017-summary-and-portfolio/) | 16일 실험 총정리 및 포트폴리오 문서 작성 | AI 활용 인재 포트폴리오 구성 |
| [018](./018-think-ultrathink-experiment/) | think / ultra think 키워드 실험 | **ultra think: 4에이전트 병렬, 30개+ 이슈 발견** |
| [019](./019-afk-autonomous-mode/) | Claude Code AFK 자율 실행 모드 실험 | 이슈 목록만 전달 후 Claude 자율 처리 |
| [020](./020-claude-api-code-reviewer/) | Claude API로 커스텀 코드 리뷰어 만들기 | 로컬 HTML 코드 리뷰어 직접 제작 |
| [021](./021-figma-mcp-android-xml/) | Figma MCP로 디자인 → Android XML 자동 변환 | Figma 파일 → Android 코드 변환 |
---

## 목적

- AI 활용 개발 학습 기록
- AI 기반 개발 workflow 실험
