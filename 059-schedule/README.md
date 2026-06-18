# 059 - /schedule 클라우드 에이전트 스케줄 실험

**세션 종료 후에도 살아있는 클라우드 에이전트 — 매주 월요일 오전 9시 GitHub 이슈 리포트 자동 생성**

---

## 목적

058일차 `/loop`의 클라우드 지속 버전인 `/schedule` 실험.  
로컬 세션이 종료돼도 클라우드에서 지속적으로 실행되는 자동화 루틴 구축.

---

## /loop vs /schedule 차이

| 항목 | /loop | /schedule |
|------|:-----:|:---------:|
| 실행 환경 | 로컬 세션 | **클라우드** |
| 세션 종료 시 | 사라짐 | **지속** |
| 실행 방식 | 시간 간격 반복 | **cron 스케줄** |
| 만료 | 세션 종료 시 | **7일 후 자동 만료** |

---

## 생성한 루틴

| 항목 | 값 |
|------|---|
| 이름 | moviefinder-issue-triage |
| 스케줄 | 매주 월요일 오전 9시 KST (UTC 00:00) |
| 모델 | claude-sonnet-4-6 |
| 저장소 | ChooJeongHo/MovieFinder |
| 결과물 | 새 GitHub Issue 생성 |

**에이전트 프롬프트:**
```
gh issue list로 14일 이상 경과된 미해결 이슈를 조회하고,
이슈 번호·제목·생성일·라벨·담당자·댓글 수를 정리한 주간 리포트를
새 GitHub Issue로 생성한다.
제목은 📋 주간 미해결 이슈 리포트 - YYYY-MM-DD
```

---

## Run now 실행 결과 (Issue #89)

**생성된 이슈**: [📋 주간 미해결 이슈 리포트 - 2026-06-18](https://github.com/ChooJeongHo/MovieFinder/issues/89)

| 라벨 | 건수 |
|------|------|
| 🔴 bug + priority-high | 9건 |
| 🔒 bug + security | 5건 |
| 🐛 bug | 9건 |
| ✨ enhancement | 4건 |
| ⚡ performance | 1건 |
| 🧪 test | 2건 |
| **총계** | **30건** |

- 가장 오래된 이슈: #16 ML Kit 태그 추천 정확도 개선 (73일 경과)
- 미배정 이슈: 30건 (100%)

---

## 핵심 발견 — schedule이 의미 있는 상황

처음엔 CI, Dependabot, Detekt이 이미 다 자동화돼있어서 schedule이 중복인가 싶었는데, **"주기적 요약 리포트"** 는 별개의 가치가 있었다:

- CI는 코드 품질 게이트
- schedule은 **전체 현황 모니터링**

개발자가 이슈를 일일이 확인 안 해도 매주 월요일에 현황이 자동으로 정리돼서 올라오는 것.

---

## /schedule 루틴 관리

```bash
# 목록 확인
/schedule → List

# 즉시 실행
/schedule → Run now

# 루틴 수정
/schedule → Update

# 웹에서 관리
https://claude.ai/code/routines/trig_01DVNgcMCeySp4sqqYmcTPEP
```

---

## 느낀 점

/loop가 "지금 이 세션에서만 반복"이라면, /schedule은 "내가 없어도 알아서 돌아가는 루틴"이다.

매주 월요일 아침에 GitHub에 이슈 현황이 자동으로 정리돼서 올라온다는 게 실제 팀 협업에서도 쓸 수 있는 수준의 자동화다.

**"AI가 개발자 대신 주간 회의 자료를 준비해둔다" — 진짜 AI 비서의 역할.**

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-06-18*
