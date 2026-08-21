# AI Dev Notes

**96일, 96개 실험 — Claude Code / MCP / GitHub Actions로 Android 개발 워크플로우를 자동화한 실험 기록**

AI 도구를 활용한 개발 실험과 학습 기록 저장소
Claude Code 등 AI 기반 개발 도구를 활용한 실험 내용을 정리합니다.

---

## 포트폴리오

→ [1차 실험 (001~017) — AI 도구 활용 및 자동화 구축](./PORTFOLIO.md)
→ [2차 실험 (018~030) — AI를 더 잘 쓰는 방법 탐구](./PORTFOLIO_v2.md)
→ [3차 실험 (031~045) — 실제 개발 문제를 AI로 해결](./PORTFOLIO_v3.md)
→ [4차 실험 (047~059) — AI 에이전트 시스템 구축](./PORTFOLIO_v4.md)
→ [5차 실험 (061~074) — AI를 이용에서 AI를 설계하는 것으로](./PORTFOLIO_v5.md)
→ [6차 실험 (076~089) — 무엇을 검증할 것인가](./PORTFOLIO_v6.md)

---

## 📊 핵심 성과 요약

| 지표 | 수치 |
|------|------|
| 총 실험 기간 | 96일 |
| 실험 수 | 96개 |
| JaCoCo 테스트 커버리지 달성 | 74.8% |
| 유닛 테스트 수 | 0 → 465개 (Claude Code 자동 작성) |
| ultra think 자동 발견 이슈 | 30개+ (4개 에이전트 병렬 분석) |
| 자동화 구축 항목 | PR 코드 리뷰, 릴리즈 노트, 취업 지원, 보안 취약점 분석 |
| 실험 대상 | MovieFinder (Android), MediScan (Android) |

---

## 기록 목록

| # | 실험 | 핵심 결과 |
|---|------|---------|
| 001 | [MovieFinder AI Workflow](./001-moviefinder-ai-workflow) | AI 기반 첫 앱 구현 기준점 |
| 002 | [Claude Code Skill Analysis](./002-claude-code-skill-analysis) | 스킬 유무에 따른 코드 품질 차이 확인 |
| 003 | [Android 코드리뷰 스킬 실전 검증](./003-android-codereview-skill-validation) | 스킬 적용 시 리뷰 깊이 향상 |
| 004 | [MovieFinder 시청 통계 기능 추가](./004-moviefinder-stats-feature) | 1일차 대비 AI 워크플로우 성장 측정 |
| 005 | [oh-my-claudecode ulw 모드 실험](./005-ohmyclaudecode-ulw-experiment) | 일반 모드 vs ulw 접근 방식 비교 |
| 006 | [oh-my-claudecode /team 멀티 에이전트 실험](./006-ohmyclaudecode-team-experiment) | 3개 에이전트 레이어별 병렬 작업 |
| 007 | [oh-my-claudecode ralph 모드 실험](./007-ohmyclaudecode-ralph-experiment) | ulw / team / ralph 3부작 종합 비교 |
| 008 | [CLAUDE.md 활용 실험](./008-claudemd-experiment) | CLAUDE.md 유무에 따른 코드 생성 차이 |
| 009 | [playwright-cli 사람인 자동 지원 시스템](./009-saramin-auto-apply) | cron 기반 매일 자동 지원, 누적 54건 지원 완료 |
| 010 | [GitHub Actions + Claude API PR 자동 코드 리뷰](./010-github-actions-claude-pr-review) | PR 생성 → 자동 리뷰 코멘트 |
| 011 | [Context7 MCP + ML Kit 영화 태그 AI 추천](./011-context7-mlkit-tag-suggestion) | 실시간 공식 문서 참조로 구현 |
| 012 | [Claude Code로 테스트 커버리지 분석 및 개선](./012-jacoco-coverage-improvement) | JaCoCo 74.8% 달성 |
| 013 | [GitHub MCP로 코드 분석 및 이슈 자동 등록](./013-github-mcp-issue-automation) | 코드 품질 분석 → 이슈 자동 생성 |
| 014 | [GitHub MCP로 이슈 → 코드 수정 → PR 자동화](./014-github-mcp-issue-to-pr) | 이슈 읽기부터 PR 생성까지 전체 자동화 |
| 015 | [GitHub MCP + gh CLI로 릴리즈 노트 자동 생성](./015-release-notes-automation) | 커밋 분석 → v1.0.0 릴리즈 노트 자동 작성 |
| 016 | [Exa MCP로 라이브러리 보안 취약점 분석](./016-exa-mcp-library-security-check) | 실시간 웹 검색으로 실제 버그 발견 |
| 017 | [16일 실험 총정리 및 포트폴리오 문서 작성](./017-summary-and-portfolio) | AI 활용 인재 포트폴리오 구성 |
| 018 | [think / ultra think 키워드 실험](./018-think-ultrathink-experiment) | ultra think: 4에이전트 병렬, 30개+ 이슈 발견 |
| 019 | [Claude Code AFK 자율 실행 모드 실험](./019-afk-autonomous-mode) | 이슈 목록만 전달 후 Claude 자율 처리 |
| 020 | [Claude API로 커스텀 코드 리뷰어 만들기](./020-claude-api-code-reviewer) | 로컬 HTML 코드 리뷰어 직접 제작 |
| 021 | [Figma MCP로 디자인 → Android XML 자동 변환](./021-figma-mcp-android-xml) | Figma 파일 → Android 코드 변환 |
| 022 | [System Prompt 설계 실험](./022-system-prompt-engineering) | 역할+제약+형식 설계 시 발견 이슈 0→3개 |
| 023 | [Few-shot Prompting 실험](./023-few-shot-prompting) | 예시 제공은 형식 통제 도구, 버그 발견 수는 0-shot이 최다 |
| 024 | [Chain of Thought 프롬프팅 실험](./024-chain-of-thought) | 단계별 사고 강제로 일반 요청 대비 문제 2개 추가 발견 |
| 025 | [System Prompt + Few-shot + CoT 조합 실험](./025-combined-prompting) | 3기법 조합 시 발견 이슈 2배, Before/After 코드 자동 생성 |
| 026 | [Claude vs GPT 동일 프롬프트 비교 실험](./026-claude-vs-gpt) | GPT는 이슈 더 많이 발견, Claude는 트레이드오프 분석 강점 |
| 027 | [Claude Code로 XML → Jetpack Compose 마이그레이션](./027-xml-to-compose) | 점진적 마이그레이션으로 선언형 UI 핵심 개념 체험 |
| 028 | [Filesystem MCP로 프로젝트 건강도 리포트 자동 생성](./028-filesystem-mcp-health-report) | 340파일 분석 → PROJECT_HEALTH.md 자동 생성 |
| 029 | [Claude Code vs Codex CLI 동일 요청 비교 실험](./029-claude-code-vs-codex-cli) | 빠른 핵심 분석은 Codex, 상세 코드 예시는 Claude Code 강점 |
| 030 | [2차 실험 총정리 및 포트폴리오 v2 작성](./030-summary-v2) | 018~030 종합 정리, 도구 활용 → AI 제어 방법 탐구로 발전 |
| 031 | [멀티모달 활용 — 스크린샷 → Compose 코드 자동 생성](./031-multimodal-screenshot-to-compose) | 이미지 입력만으로 UI 구조 분석 + Compose 코드 생성 |
| 032 | [멀티모달 활용 — 스크린샷 → 접근성 점검 자동화](./032-multimodal-accessibility-check) | 이미지 분석은 시각적 스크리닝, 코드 검증은 Claude Code로 |
| 033 | [ADB + Claude Code로 실기기 UI 자동 테스트](./033-adb-real-device-ui-testing) | 실기기 자동 조작 + UI dump 분석으로 버그 3개 자동 발견 |
| 034 | [ADB 심화 — 텍스트 입력 + 스크롤 + 프레임 성능 측정](./034-adb-advanced-automation) | Samsung 한글 입력 차단 해결 + gfxinfo 프레임 측정 (Janky 0.93%) |
| 035 | [ADB + Claude Code로 UI Visual Regression Test 자동화](./035-visual-regression-test) | before/after 자동 캡처 + Claude 이미지 비교 분석 + 리포트 자동 생성 |
| 036 | [Claude Code로 기술 문서 자동 생성](./036-technical-docs-automation) | MovieFinder 1,187줄 / MediScan 754줄 기술문서 자동 생성, ADR 역추적 포함 |
| 037 | [MovieFinder 전용 커스텀 MCP 서버 직접 제작](./037-custom-mcp-server) | ADB 자동화 툴 4종 MCP화, 보안 취약점 자동 수정 포함 |
| 038 | [CLAUDE.md 경량화 + 품질 비교 실험](./038-claudemd-optimization) | 26.6% 경량화 후 코드리뷰 점수 8.2→8.6, 블로킹 이슈 4→0개 |
| 039 | [모델별 코드리뷰 비용 vs 품질 트레이드오프](./039-model-comparison) | Sonnet이 회귀 버그 탐지 최강, "비싼 모델 = 좋은 리뷰" 가설 반증 |
| 040 | [Claude Code vs Gemini CLI 코드리뷰 비교](./040-claude-vs-gemini-cli) | 버그 탐지는 Claude Code, 아키텍처 개선 제안은 Gemini CLI 강점 |
| 041 | [GEMINI.md 설정 전후 비교 실험](./041-gemini-md-impact) | 이슈 수 동일, 분석 깊이와 탐색 방식 개선 — "AI 도구는 컨텍스트 설정이 절반" |
| 042 | [멀티 리뷰 통합 → 버그 분류 → GitHub Issues 자동 등록](./042-bug-classification-pipeline) | 52건→36건 중복 제거, HIGH 9건 병렬 자동 등록 (#62~#70) |
| 043 | [Claude Code로 앱 스토어 설명문 자동 생성](./043-store-listing-automation) | strings.xml 분석으로 숨겨진 기능 발굴, MovieFinder·MediScan 즉시 등록 가능 설명문 생성 |
| 044 | [Claude Code로 신규 개발자 온보딩 문서 자동 생성](./044-onboarding-doc-automation) | 563줄 온보딩 문서 자동 생성, "자주 하는 실수" 8가지·핵심 패턴 9가지 포함 |
| 045 | [Claude Code로 코드 기술 부채 자동 측정](./045-tech-debt-measurement) | 14,305줄 분석, 부채 점수 28/100 🟢, TODO 0건, Presentation God Class 집중 확인 |
| 046 | [3차 실험 총정리 및 포트폴리오 v3 작성](./046-summary-v3) | 031~045 종합 정리, 실제 개발 문제를 AI로 해결하는 파이프라인 완성 |
| 047 | [Claude Code Auto Mode 실험](./047-auto-mode) | 승인 0회로 코드 수정→컴파일→Detekt→커밋 완주, 6분 29초 |
| 048 | [Claude Code Plan Mode 실험](./048-plan-mode) | 계획 확인 후 실행, Auto Mode보다 빠른 4분 41초 + Detekt 오류 0건 |
| 049 | [Claude Code Accept Edits Mode 실험](./049-accept-edits-mode) | 12파일 순감소 58줄, 4가지 모드 완전 비교 완성 |
| 050 | [/sciomc 병렬 에이전트 아키텍처 건강도 분석](./050-sciomc-parallel-agents) | 5개 에이전트 병렬 분석, 이전 5번 감사에서 놓친 치명 버그 2개 신규 발견 |
| 051 | [/autoresearch 자율 테스트 커버리지 향상](./051-autoresearch) | Line 66%→75%, Branch 63%→70% 자율 달성, 테스트 ~1,500줄 자동 추가 |
| 052 | [Claude GitHub App + @claude PR 자동 수정](./052-autofix-pr) | CI 실패 PR에 @claude 코멘트로 37초 만에 자동 수정 + 커밋 완료 |
| 053 | [Claude Code Hooks — .kt 수정 시 Detekt 자동 실행](./053-claude-code-hooks) | PostToolUse 훅으로 파일 수정 즉시 ~800ms Detekt 실행, pre-commit 대비 37배 빠름 |
| 054 | [Claude Code PreToolUse 훅 — Domain 레이어 Android 의존성 차단](./054-pretooluse-hooks) | 수정 전 Android import 원천 차단, Pre+Post+pre-commit 3단계 품질 게이트 완성 |
| 055 | [/ultrawork 병렬 에이전트 실험 — 3가지 작업 동시 실행](./055-ultrawork) | 8분 31초 만에 UI 버그 8개 발견 + 테스트 30개 추가, worktree 격리 자동 적용 |
| 056 | [Claude Code로 앱 성능 프로파일링 자동화](./056-performance-profiling) | 에뮬레이터 Janky 31% → 실기기 5.41%, 에뮬레이터가 최대 11배 과장됨을 수치로 증명 |
| 057 | [/ultraplan 실험 — 웹 Plan + 로컬 실행](./057-ultraplan) | 클라우드 심층 계획으로 아키텍처 개선 + detekt 버전 버그·AGP 호환성 이슈 2개 추가 발견 |
| 058 | [/loop 명령어 실험 — 5분마다 자율 반복 코드 개선](./058-loop) | 3회차 자율 반복으로 14개 파일 수정, PostToolUse 훅과 시너지로 실시간 오류 감지 |
| 059 | [/schedule 클라우드 에이전트 스케줄 실험](./059-schedule) | 매주 월요일 오전 9시 GitHub 이슈 리포트 자동 생성, 세션 종료 후에도 지속 |
| 060 | [4차 실험 총정리 및 포트폴리오 v4 작성](./060-summary-v4) | 047~059 종합 정리, AI 에이전트 시스템 구축 단계 완성 |
| 061 | [Exa MCP 심화 — 라이브러리 최신 버전 및 보안 이슈 자동 리서치](./061-exa-library-research) | 4개 라이브러리 병렬 검색, 의존성 충돌 연결 고리 자동 발견, 업그레이드 가이드 209줄 생성 |
| 062 | [Baseline Profile 자동화 — 한 줄 명령으로 프로필 갱신](./062-baseline-profile-automation) | 307줄 스크립트로 에뮬레이터 시작→프로필 생성→결과 검증 4분 3초 완료 |
| 063 | [Claude Code로 접근성 자동 감사](./063-accessibility-audit) | MAJOR 4건 + MINOR 5건 발견, 색상 대비 WCAG 기준 자동 계산 |
| 064 | [Context7 MCP 심화 — 공식 문서 기반 라이브러리 개선점 발견](./064-context7-deep-dive) | 공식 문서와 코드 대조로 8개 개선점 발견, 훈련 데이터 컷오프 이후 최신 API 패턴 반영 |
| 065 | [Claude Code로 i18n 자동화 — strings.xml 영어 번역 감사 및 수정](./065-i18n-automation) | KO/EN 301개 키 완전 일치 달성, 누락 3개 추가 + plurals 문법 오류 발견 |
| 066 | [릴리즈 파이프라인 자동화 — 버전 관리부터 GitHub Release까지](./066-release-pipeline) | 한 줄 명령으로 버전 증가→CHANGELOG→태그→AAB 빌드→GitHub Release 자동 완성 |
| 067 | [네트워크 보안 강화 자동화](./067-network-security) | OS 레벨 + 앱 레벨 이중 피닝 전략 적용, Bearer 토큰 로그 마스킹 |
| 068 | [SearchFragment Compose 마이그레이션 (심화)](./068-search-compose-migration) | ViewModel+Paging3+RecyclerView 하이브리드 마이그레이션, 콜백 브릿지 패턴 적용 |
| 069 | [의존성 버전 자동 진단 — 안전/위험 판단 파이프라인](./069-dependency-update-pipeline) | 34개 항목 중 30개 최신 확인, 버전 불변이어도 위험한 케이스(archived·deprecated) 자동 탐지 |
| 070 | [Claude Code 커스텀 Subagent 직접 설계](./070-custom-subagent-design) | "AI 이용"에서 "AI 설계"로 전환, 서브에이전트가 잡은 오탐(false positive)을 실제 컴파일로 검증하며 한계까지 문서화 |
| 071 | [Subagent 개선 + PreToolUse 훅 자동 연동](./071-subagent-hook-integration) | 오탐 방지 규칙 내장 + Hook으로 자동 사전검토 시스템 완성, "결정론적 차단 vs 확률적 정보 제공" 설계 원칙 정립 |
| 072 | [멀티 에이전트 오케스트레이션 — 설계자→구현자→검증자 파이프라인](./072-multi-agent-pipeline) | 원샷 대비 파이프라인이 시간 18% 절약 + 검증자가 원샷에서만 응집도 문제 발견, 구조적 안전장치의 가치 확인 |
| 073 | [Stop Hook — 훅 3부작 완성](./073-stop-hook-trilogy) | 테스트 미작성 시 세션 종료 차단 구현, "에이전트 경계 = 상태 경계"라는 070/072/073 공통 원인 발견 |
| 074 | [사후 감시에서 사전 설계로 — 073 한계 재설계로 해소](./074-pipeline-redesign) | 구현자 에이전트에 테스트 필수 규칙 내장, 사전 설계도 완벽하지 않음을 실측 확인 → 계층 방어 결론 |
| 075 | [5차 실험 총정리 및 포트폴리오 v5 작성](./075-summary-v5) | 061~074 종합 정리, "AI 이용"에서 "AI 설계"로 전환한 단계 완성 |
| 076 | [MovieFinder 코드베이스 미니 RAG — Room DB를 벡터 저장소로](./076-codebase-rag) | API 키 없이 Room 기반 RAG 파이프라인 완성, 조용한 한글 토큰화 버그 실측으로 발견·수정 |
| 077 | [로컬 해싱 임베딩 vs 실제 Voyage AI 임베딩 비교](./077-voyage-embedding-comparison) | "1줄 교체" 설계 실제 검증, 어휘 불일치 질문에서 의미 기반 검색의 가치를 수치로 확인 |
| 078 | ["원리적으로 답 못한다"를 실제로 풀다 — RAG에 JaCoCo 통합](./078-jacoco-rag-integration) | 코퍼스 확장으로 076/077의 미해결 질문(Q3) 실제 해결, "임베딩 품질이 아니라 데이터 존재 여부" 교훈 |
| 079 | [RAG에 성능 데이터 통합 — 에뮬레이터 vs 실기기 재현](./079-performance-rag-integration) | 타입별 부분 재인덱싱 구현, 056일차 에뮬레이터 왜곡 재확인(Jank 61%→1.17%), "유사도≠랭킹" 한계 발견 |
| 080 | [LLM-as-judge — "AI의 판단을 어떻게 검증하는가"](./080-llm-as-judge) | 070/074일차 리뷰 재검증에서 순수 환각 발견, 3건 공통 결함(A5 누락) 즉시 원천 차단으로 보강 |
| 081 | [KOFIC 박스오피스 API 연동 — 실기기에서만 보이는 버그 2개](./081-kofic-api-integration) | 빌드/테스트/detekt 전부 통과했지만 실기기에서 크래시 2건 발견 및 수정, TMDB×KOFIC 데이터 매칭 설계 |
| 082 | [N+1 TMDB 검색 문제 해결 — "우연히 통과한 테스트" 발견](./082-n1-cache-optimization) | 로컬 캐시 우선 조회로 TOP50 기준 5배→거의 동일 성능 개선, MockK 예외 삼킴으로 인한 가짜 테스트 통과 발견 및 수정 |
| 083 | [KOFIC 주간 박스오피스 확장 — 재사용성 실증](./083-kofic-weekly-extension) | 081/082 설계의 재사용 가능성을 실기기 로그로 직접 증명, 082와 다른 형태의 "우연한 테스트 통과" 재발견 및 예방 |
| 084 | [KOFIC UI 접근성 재감사 — 063일차 교훈이 새 코드에 남았는가](./084-kofic-accessibility-reaudit) | 색상 단독 의존·발화 누락 패턴이 신규 UI에서 반복 확인, 아키텍처 수정이 접근성도 부수적으로 개선한 사례 발견 |
| 085 | [LLM-as-judge를 RAG에 적용 — 재사용성 실증](./085-rag-answer-judge) | 검증자 자신의 실수까지 judge가 적발, 079일차 함정(유사도≠랭킹) 재현한 오답을 정확히 감점 상한으로 판정, 재현성 검증 완료 |
| 086 | [Jetpack Glance 홈 화면 위젯](./086-glance-widget) | Domain 레이어 100% 재사용, XML→Compose→Glance 세 번의 UI 전환에도 비즈니스 로직 무변경 실증 |
| 087 | [실기기 검증을 Stop Hook으로 강제](./087-device-verification-hook) | 081~086 "실기기 검증 필수" 원칙을 시스템화, "파일 존재"와 "명령 실행"이 다른 신뢰도의 증거임을 정직하게 구분 |
| 088 | [Stop Hook 실전 검증 + "오래된 증거" 함정 — 3부작 완성](./088-stop-hook-trilogy-completion) | 진짜 세션 종료로 087일차 훅 실전 검증, 083일차와 같은 "정적 사실만 확인하면 시점을 놓친다" 함정 재발견 및 수정 |
| 089 | [기존 스킬로 검증하고 즉시 고치기 — /network-layer-validator](./089-network-layer-validator) | 081~083 KOFIC 연동 재검증, callTimeout 미설정 발견 즉시 수정 및 재검증, 검증 도구가 coroutineScope/supervisorScope 오탐 스스로 회피 |
| 090 | [6차 실험 총정리 및 포트폴리오 v6 작성](./090-summary-v6) | 076~089 종합 정리, "무엇을 검증할 것인가"라는 6차 사이클 완성 |
| 091 | [기존 스킬로 검증하기 — /material3-design-validator](./091-material3-design-validator) | 086일차 Glance 위젯 다크모드 대응 확인, 타이포그래피 하드코딩 수정 + 부수 발견 딥링크 크래시·인증서 핀 간헐 실패 처리 |
| 092 | [기존 스킬로 검증하기 — /proguard-validator (검증 스킬 3부작 완성)](./092-proguard-validator) | 릴리즈 빌드 전용 keep 규칙 누락 발견 및 수정, "테스트 환경과 배포 환경의 간극"이라는 패턴의 네 번째 변주 |
| 093 | [070~092일차 아키텍처 다이어그램 정리](./093-architecture-diagram) | Subagent·RAG·실제 기능·검증 스킬이 서로 어떻게 재사용되는지 그림 한 장으로 정리 |
| 094 | [인증서 핀 간헐적 실패 — 원인 재현 없이 안전장치 강화](./094-cert-pin-resilience) | 백업 핀 추가 + SSL 실패 재시도로 067일차 이중 피닝을 한 단계 더 강화, 원인 불명은 정직하게 인정 |
| 095 | [영상물등급위원회(KMRB) API 연동 — 잘못된 짐작을 사람이 바로잡다](./095-kmrb-rating-integration) | AI가 짐작한 엔드포인트가 틀려 사용자가 직접 curl로 실제 스펙 발견, XML 파싱 도입 + 스레드/동시성 버그 2건 검증자가 적발 |
| 096 | [KMRB 관람등급 필터 — 082일차 N+1 해법 재사용 한계 발견](./096-kmrb-rating-filter) | "캐시 우선 조회" 패턴은 재사용됐지만 "이미 채워진 캐시"라는 전제는 KMRB엔 없어서 새로 캐시 구축 필요 |

---

## 목적

AI 도구를 단순히 "써보는" 것이 아니라, 실제 Android 프로젝트에 어떻게 통합하면 생산성이 올라가는지 직접 실험하고 수치로 증명합니다.

비전공자 출신으로 Android 개발을 시작했고, AI 도구를 활용해 코드 품질·테스트·자동화 전반을 빠르게 성장시킨 과정을 기록합니다.

각 실험은 동일한 프로젝트(MovieFinder)를 대상으로 반복하여 실험 간 비교가 가능합니다.

---

## 🛠️ 사용 도구

**AI 도구**
- Claude Code (CLI) — 코드 생성, 리팩토링, 테스트 자동화, 멀티 에이전트, 커스텀 Subagent 설계
- Claude.ai — 멀티모달 활용 (이미지 → 코드/분석)
- Codex CLI — OpenAI 터미널 기반 코딩 에이전트 (비교 실험)
- Claude API — GitHub Actions 연동, 커스텀 코드 리뷰어 제작
- oh-my-claudecode — ulw / team / ralph / sciomc / ultrawork / autoresearch / loop / schedule 모드 실험

**MCP 서버**
- GitHub MCP — 이슈 생성, PR 자동화, 릴리즈 노트
- Context7 MCP — 실시간 공식 문서 참조
- Exa MCP — 라이브러리 보안 취약점·최신 버전 실시간 검색
- Figma MCP — 디자인 → Android 코드 자동 변환
- Filesystem MCP — 프로젝트 파일 분석 및 리포트 자동 생성
- playwright-cli — 브라우저 자동화, 취업 지원 자동화

**기기 자동화**
- ADB (Android Debug Bridge) — 실기기 자동 조작, 텍스트 입력, 성능 측정

**인프라**
- GitHub Actions — CI/CD, PR 자동 리뷰, 인증서 핀 검증, 릴리즈 파이프라인
- Claude Code Hooks — PreToolUse/PostToolUse/Stop 기반 자동 품질 게이트
