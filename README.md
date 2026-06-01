# AI Dev Notes
**47일, 47개 실험 — Claude Code / MCP / GitHub Actions로 Android 개발 워크플로우를 자동화한 실험 기록**

AI 도구를 활용한 개발 실험과 학습 기록 저장소  
Claude Code 등 AI 기반 개발 도구를 활용한 실험 내용을 정리합니다.

---

## 포트폴리오

→ [1차 실험 (001~017) — AI 도구 활용 및 자동화 구축](./PORTFOLIO.md)  
→ [2차 실험 (018~030) — AI를 더 잘 쓰는 방법 탐구](./PORTFOLIO_v2.md)  
→ [3차 실험 (031~045) — 실제 개발 문제를 AI로 해결](./PORTFOLIO_v3.md)

---

## 📊 핵심 성과 요약

| 지표 | 수치 |
|------|------|
| 총 실험 기간 | 47일 |
| 실험 수 | 47개 |
| JaCoCo 테스트 커버리지 달성 | 74.8% |
| 유닛 테스트 수 | 0 → 465개 (Claude Code 자동 작성) |
| ultra think 자동 발견 이슈 | 30개+ (4개 에이전트 병렬 분석) |
| 자동화 구축 항목 | PR 코드 리뷰, 릴리즈 노트, 취업 지원, 보안 취약점 분석 |
| 실험 대상 | MovieFinder (Android), MediScan (Android) |

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
| 022 | [System Prompt 설계 실험](./022-system-prompt-engineering/) | 역할+제약+형식 설계 시 발견 이슈 0→3개 |
| 023 | [Few-shot Prompting 실험](./023-few-shot-prompting/) | 예시 제공은 형식 통제 도구, 버그 발견 수는 0-shot이 최다 |
| 024 | [Chain of Thought 프롬프팅 실험](./024-chain-of-thought/) | 단계별 사고 강제로 일반 요청 대비 문제 2개 추가 발견 |
| 025 | [System Prompt + Few-shot + CoT 조합 실험](./025-combined-prompting/) | 3기법 조합 시 발견 이슈 2배, Before/After 코드 자동 생성 |
| 026 | [Claude vs GPT 동일 프롬프트 비교 실험](./026-claude-vs-gpt/) | GPT는 이슈 더 많이 발견, Claude는 트레이드오프 분석 강점 |
| 027 | [Claude Code로 XML → Jetpack Compose 마이그레이션](./027-xml-to-compose/) | 점진적 마이그레이션으로 선언형 UI 핵심 개념 체험 |
| 028 | [Filesystem MCP로 프로젝트 건강도 리포트 자동 생성](./028-filesystem-mcp-health-report/) | 340파일 분석 → PROJECT_HEALTH.md 자동 생성 |
| 029 | [Claude Code vs Codex CLI 동일 요청 비교 실험](./029-claude-code-vs-codex-cli/) | 빠른 핵심 분석은 Codex, 상세 코드 예시는 Claude Code 강점 |
| 030 | [2차 실험 총정리 및 포트폴리오 v2 작성](./030-summary-v2/) | 018~030 종합 정리, 도구 활용 → AI 제어 방법 탐구로 발전 |
| 031 | [멀티모달 활용 — 스크린샷 → Compose 코드 자동 생성](./031-multimodal-screenshot-to-compose/) | 이미지 입력만으로 UI 구조 분석 + Compose 코드 생성 |
| 032 | [멀티모달 활용 — 스크린샷 → 접근성 점검 자동화](./032-multimodal-accessibility-check/) | 이미지 분석은 시각적 스크리닝, 코드 검증은 Claude Code로 |
| 033 | [ADB + Claude Code로 실기기 UI 자동 테스트](./033-adb-real-device-ui-testing/) | 실기기 자동 조작 + UI dump 분석으로 버그 3개 자동 발견 |
| 034 | [ADB 심화 — 텍스트 입력 + 스크롤 + 프레임 성능 측정](./034-adb-advanced-automation/) | Samsung 한글 입력 차단 해결 + gfxinfo 프레임 측정 (Janky 0.93%) |
| 035 | [ADB + Claude Code로 UI Visual Regression Test 자동화](./035-visual-regression-test/) | before/after 자동 캡처 + Claude 이미지 비교 분석 + 리포트 자동 생성 |
| 036 | [Claude Code로 기술 문서 자동 생성](./036-technical-docs-automation/) | MovieFinder 1,187줄 / MediScan 754줄 기술문서 자동 생성, ADR 역추적 포함 |
| 037 | [MovieFinder 전용 커스텀 MCP 서버 직접 제작](./037-custom-mcp-server/) | ADB 자동화 툴 4종 MCP화, 보안 취약점 자동 수정 포함 |
| 038 | [CLAUDE.md 경량화 + 품질 비교 실험](./038-claudemd-optimization/) | 26.6% 경량화 후 코드리뷰 점수 8.2→8.6, 블로킹 이슈 4→0개 |
| 039 | [모델별 코드리뷰 비용 vs 품질 트레이드오프](./039-model-comparison/) | Sonnet이 회귀 버그 탐지 최강, "비싼 모델 = 좋은 리뷰" 가설 반증 |
| 040 | [Claude Code vs Gemini CLI 코드리뷰 비교](./040-claude-vs-gemini-cli/) | 버그 탐지는 Claude Code, 아키텍처 개선 제안은 Gemini CLI 강점 |
| 041 | [GEMINI.md 설정 전후 비교 실험](./041-gemini-md-impact/) | 이슈 수 동일, 분석 깊이와 탐색 방식 개선 — "AI 도구는 컨텍스트 설정이 절반" |
| 042 | [멀티 리뷰 통합 → 버그 분류 → GitHub Issues 자동 등록](./042-bug-classification-pipeline/) | 52건→36건 중복 제거, HIGH 9건 병렬 자동 등록 (#62~#70) |
| 043 | [Claude Code로 앱 스토어 설명문 자동 생성](./043-store-listing-automation/) | strings.xml 분석으로 숨겨진 기능 발굴, MovieFinder·MediScan 즉시 등록 가능 설명문 생성 |
| 044 | [Claude Code로 신규 개발자 온보딩 문서 자동 생성](./044-onboarding-doc-automation/) | 563줄 온보딩 문서 자동 생성, "자주 하는 실수" 8가지·핵심 패턴 9가지 포함 |
| 045 | [Claude Code로 코드 기술 부채 자동 측정](./045-tech-debt-measurement/) | 14,305줄 분석, 부채 점수 28/100 🟢, TODO 0건, Presentation God Class 집중 확인 |
| 046 | [3차 실험 총정리 및 포트폴리오 v3 작성](./046-summary-v3/) | 031~045 종합 정리, 실제 개발 문제를 AI로 해결하는 파이프라인 완성 |
| 047 | [Claude Code Auto Mode 실험](./047-auto-mode/) | 승인 0회로 코드 수정→컴파일→Detekt→커밋 완주, 6분 29초 |

---

## 목적

AI 도구를 단순히 "써보는" 것이 아니라, 실제 Android 프로젝트에 어떻게 통합하면 생산성이 올라가는지 직접 실험하고 수치로 증명합니다.

비전공자 출신으로 Android 개발을 시작했고, AI 도구를 활용해 코드 품질·테스트·자동화 전반을 빠르게 성장시킨 과정을 기록합니다.

각 실험은 동일한 프로젝트(MovieFinder)를 대상으로 반복하여 실험 간 비교가 가능합니다.

---

## 🛠️ 사용 도구

**AI 도구**
- Claude Code (CLI) — 코드 생성, 리팩토링, 테스트 자동화, 멀티 에이전트
- Claude.ai — 멀티모달 활용 (이미지 → 코드/분석)
- Codex CLI — OpenAI 터미널 기반 코딩 에이전트 (비교 실험)
- Claude API — GitHub Actions 연동, 커스텀 코드 리뷰어 제작
- oh-my-claudecode — ulw / team / ralph 모드 실험

**MCP 서버**
- GitHub MCP — 이슈 생성, PR 자동화, 릴리즈 노트
- Context7 MCP — 실시간 공식 문서 참조
- Exa MCP — 라이브러리 보안 취약점 실시간 검색
- Figma MCP — 디자인 → Android 코드 자동 변환
- Filesystem MCP — 프로젝트 파일 분석 및 리포트 자동 생성
- playwright-cli — 브라우저 자동화, 취업 지원 자동화

**기기 자동화**
- ADB (Android Debug Bridge) — 실기기 자동 조작, 텍스트 입력, 성능 측정

**인프라**
- GitHub Actions — CI/CD, PR 자동 리뷰, 인증서 핀 검증
