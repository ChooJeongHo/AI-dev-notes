# 060 - 4차 실험 총정리 및 포트폴리오 v4 작성

**047~059일차 13개 실험 종합 정리 — Claude Code 자율 에이전트 시스템 구축 완성**

---

## 목적

047~059일차 실험을 돌아보며 4차 포트폴리오 작성.  
3차(실제 개발 문제 해결)에서 4차는 **AI 에이전트 시스템을 직접 구축하는 단계**로 발전.

---

## 4차 실험 핵심 성과

| 지표 | 수치 |
|------|------|
| 실험 수 | 13개 (047~059) |
| Claude Code 모드 비교 | 4가지 완전 비교 (Auto/Plan/Accept Edits/Default) |
| 병렬 에이전트 실험 | 3가지 (/sciomc, /ultrawork, /autoresearch) |
| 테스트 커버리지 자율 달성 | Line 66%→75%, Branch 63%→70% (~1,500줄 자동 추가) |
| @claude PR 자동 수정 | 37초 만에 CI 실패 자동 수정 |
| Claude Code Hooks | PreToolUse + PostToolUse 3단계 품질 게이트 완성 |
| /loop 자동 수정 | 3회차 자율 반복으로 14개 파일 수정 |
| 에뮬레이터 vs 실기기 | Janky 31%→5.41%, 최대 11배 차이 수치 증명 |
| 클라우드 스케줄 루틴 | 매주 월요일 GitHub 이슈 리포트 자동 생성 |

---

## 047~059 실험 목록

| # | 실험 | 핵심 결과 |
|---|------|---------|
| 047 | Claude Code Auto Mode | 승인 0회, 6분 29초 완주 |
| 048 | Claude Code Plan Mode | 4분 41초, Detekt 오류 0건 |
| 049 | Claude Code Accept Edits Mode | 12파일 순감소 58줄, 4가지 모드 완전 비교 |
| 050 | /sciomc 병렬 에이전트 | 이전 5번 감사에서 놓친 치명 버그 2개 신규 발견 |
| 051 | /autoresearch 자율 커버리지 향상 | Line 66%→75%, Branch 63%→70% 자율 달성 |
| 052 | @claude PR 자동 수정 | CI 실패 PR → 37초 만에 자동 수정 + 커밋 |
| 053 | PostToolUse 훅 | .kt 수정 즉시 ~800ms Detekt 실행, pre-commit 대비 37배 빠름 |
| 054 | PreToolUse 훅 | Domain 레이어 Android import 원천 차단, 3단계 품질 게이트 완성 |
| 055 | /ultrawork 병렬 3작업 | 8분 31초, UI 버그 8개 발견 + 테스트 30개 추가 |
| 056 | 앱 성능 프로파일링 자동화 | 에뮬레이터 Janky 31% → 실기기 5.41%, 최대 11배 차이 |
| 057 | /ultraplan | 클라우드 심층 계획으로 아키텍처 개선 + detekt 버그 발견 |
| 058 | /loop | 3회차 자율 반복으로 14개 파일 수정, PostToolUse 훅과 시너지 |
| 059 | /schedule | 매주 월요일 오전 9시 GitHub 이슈 리포트 자동 생성 |

---

## 4차에서 배운 핵심 교훈

**"모드를 상황에 맞게 선택하는 것이 Claude Code를 잘 쓰는 능력이다"**  
Auto/Plan/Accept Edits — 각각 최적인 상황이 다르다.

**"단일 에이전트와 병렬 에이전트는 다른 버그를 잡는다"**  
시스템 관점에서만 보이는 버그가 존재한다.

**"훅이 있으면 AI가 스스로 품질을 지킨다"**  
PreToolUse + PostToolUse 조합으로 AI가 수정 전/후에 자동으로 규칙을 확인한다.

**"목표를 주면 달성하고, 루틴을 주면 지속한다"**  
/autoresearch는 수치 목표를 달성하고, /schedule은 세션 없이도 매주 돌아간다.

---

→ [PORTFOLIO_v4.md 전체 보기](../PORTFOLIO_v4.md)

---

*작성일: 2026-06-19*
