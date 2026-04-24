# AI Dev Notes
**22일, 22개 실험 — Claude Code / MCP / GitHub Actions로 Android 개발 워크플로우를 자동화한 실험 기록**

AI 도구를 활용한 개발 실험과 학습 기록 저장소  
Claude Code 등 AI 기반 개발 도구를 활용한 실험 내용을 정리합니다.

---

## 포트폴리오

→ [AI를 활용한 Android 개발 워크플로우 자동화 구축기](./PORTFOLIO.md)

---

## 📊 핵심 성과 요약

| 지표 | 수치 |
|------|------|
| 총 실험 기간 | 22일 |
| 실험 수 | 22개 |
| JaCoCo 테스트 커버리지 달성 | 74.8% |
| 유닛 테스트 수 | 0 → 465개 (Claude Code 자동 작성) |
| ultra think 자동 발견 이슈 | 30개+ (4개 에이전트 병렬 분석) |
| 자동화 구축 항목 | PR 코드 리뷰, 릴리즈 노트, 취업 지원, 보안 취약점 분석 |
| 실험 대상 | MovieFinder (Android) |

---

## 기록 목록

| # | 실험 | 핵심 결과 |
|---|------|---------|
| 001 | [MovieFinder AI Workflow](./001-moviefinder-ai-workflow/) | AI 기반 첫 앱 구현 기준점 |
| 002 | [Claude Code Skill Analysis](./002-claude-code-skill-analysis/) | 스킬 유무에 따른 코드 품질 차이 확인 |
| 003 | [Android 코드리뷰 스킬 실전 검증](./003-android-codereview-skill-validation/) | 스킬 적용 시 리뷰 깊이 향상 |
| 004 | [MovieFinder 시청 통계 기능 추가](./004-moviefinder-stats-feature/) | 1일차 대비 AI 워크플로우 성장 측정 |
| 005 | [oh-my-claudecode ulw 모드 실험](./005-ohmyclaudecode-ulw-experiment/) | 일반 모드 vs ulw 접근 방식 비교 |
| 006 | [oh-my-claudecode /team 멀티 에이전트 실험](./006-ohmyclaudecode-team-experiment/) | 3개 에이전트 레이어별 병렬 작업 |
| 007 | [oh-my-claudecode ralph 모드 실험](./007-ohmyclaudecode-ralph-experiment/) | ulw / team / ralph 3부작 종합 비교 |
| 008 | [CLAUDE.md 활용 실험](./008-claudemd-experiment/) | CLAUDE.md 유무에 따른 코드 생성 차이 |
| 009 | [playwright-cli 사람인 자동 지원 시스템](./009-saramin-auto-apply/) | cron 기반 매일 자동 지원, 누적 54건 지원 완료 |
| 010 | [GitHub Actions + Claude API PR 자동 코드 리뷰](./010-github-actions-claude-pr-review/) | PR 생성 → 자동 리뷰 코멘트 |
| 011 | [Context7 MCP + ML Kit 영화 태그 AI 추천](./011-context7-mlkit-tag-suggestion/) | 실시간 공식 문서 참조로 구현 |
| 012 | [Claude Code로 테스트 커버리지 분석 및 개선](./012-jacoco-coverage-improvement/) | JaCoCo 74.8% 달성 |
| 013 | [GitHub MCP로 코드 분석 및 이슈 자동 등록](./013-github-mcp-issue-automation/) | 코드 품질 분석 → 이슈 자동 생성 |
| 014 | [GitHub MCP로 이슈 → 코드 수정 → PR 자동화](./014-github-mcp-issue-to-pr/) | 이슈 읽기부터 PR 생성까지 전체 자동화 |
| 015 | [GitHub MCP + gh CLI로 릴리즈 노트 자동 생성](./015-release-notes-automation/) | 커밋 분석 → v1.0.0 릴리즈 노트 자동 작성 |
| 016 | [Exa MCP로 라이브러리 보안 취약점 분석](./016-exa-mcp-library-security-check/) | 실시간 웹 검색으로 실제 버그 발견 |
| 017 | [16일 실험 총정리 및 포트폴리오 문서 작성](./017-summary-and-portfolio/) | AI 활용 인재 포트폴리오 구성 |
| 018 | [think / ultra think 키워드 실험](./018-think-ultrathink-experiment/) | ultra think: 4에이전트 병렬, 30개+ 이슈 발견 |
| 019 | [Claude Code AFK 자율 실행 모드 실험](./019-afk-autonomous-mode/) | 이슈 목록만 전달 후 Claude 자율 처리 |
| 020 | [Claude API로 커스텀 코드 리뷰어 만들기](./020-claude-api-code-reviewer/) | 로컬 HTML 코드 리뷰어 직접 제작 |
| 021 | [Figma MCP로 디자인 → Android XML 자동 변환](./021-figma-mcp-android-xml/) | Figma 파일 → Android 코드 변환 |
| 022 | [System Prompt 설계 실험](./022-system-prompt-engineering/) | 역할+제약+형식 설계 시 발견 이슈 0→3개, 전문가 수준 응답 |

---

## 목적

AI 도구를 단순히 "써보는" 것이 아니라, 실제 Android 프로젝트에 어떻게 통합하면 생산성이 올라가는지 직접 실험하고 수치로 증명합니다.

비전공자 출신으로 Android 개발을 시작했고, AI 도구를 활용해 코드 품질·테스트·자동화 전반을 빠르게 성장시킨 과정을 기록합니다.

각 실험은 동일한 프로젝트(MovieFinder)를 대상으로 반복하여 실험 간 비교가 가능합니다.

---

## 🛠️ 사용 도구

**AI 도구**
- Claude Code (CLI) — 코드 생성, 리팩토링, 테스트 자동화, 멀티 에이전트
- Claude API — GitHub Actions 연동, 커스텀 코드 리뷰어 제작
- oh-my-claudecode — ulw / team / ralph 모드 실험

**MCP 서버**
- GitHub MCP — 이슈 생성, PR 자동화, 릴리즈 노트
- Context7 MCP — 실시간 공식 문서 참조
- Exa MCP — 라이브러리 보안 취약점 실시간 검색
- Figma MCP — 디자인 → Android 코드 자동 변환
- playwright-cli — 브라우저 자동화, 취업 지원 자동화

**인프라**
- GitHub Actions — CI/CD, PR 자동 리뷰, 인증서 핀 검증
