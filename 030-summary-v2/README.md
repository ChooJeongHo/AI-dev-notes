# 030 - 2차 실험 총정리 및 포트폴리오 v2 작성

**018~030일차 실험을 종합 정리하고 포트폴리오 v2로 재구성**

---

## 목적

17일차 1차 총정리 이후 18~30일차 실험들을 정리.  
1차(도구 활용)에서 2차(AI를 더 잘 쓰는 방법 탐구)로 발전한 과정을 포트폴리오로 재구성.

---

## 018~030 실험 전체 흐름

```
018일차  — think / ultra think 키워드 실험
019일차  — Claude Code AFK 자율 실행 모드
020일차  — Claude API로 커스텀 코드 리뷰어 제작
021일차  — Figma MCP → Android XML 자동 변환
022일차  — System Prompt 설계 실험
023일차  — Few-shot Prompting 실험
024일차  — Chain of Thought 프롬프팅 실험
025일차  — 3기법 조합 실험
026일차  — Claude vs GPT 동일 프롬프트 비교
027일차  — XML → Jetpack Compose 마이그레이션
028일차  — Filesystem MCP 프로젝트 건강도 리포트
029일차  — Claude Code vs Codex CLI 비교
030일차  — 2차 총정리 + 포트폴리오 v2
```

---

## 1차 vs 2차 비교

| 구분 | 1차 (001~017) | 2차 (018~030) |
|------|-------------|-------------|
| 핵심 주제 | AI 도구 활용 및 자동화 구축 | AI를 더 잘 쓰는 방법 탐구 |
| 주요 실험 | GitHub Actions, MCP, AFK 모드 | 프롬프트 엔지니어링, 모델 비교 |
| 새로 배운 것 | 도구 설치/활용 | 프롬프트 설계, 도구 간 차이 판단 |

---

## 2차 핵심 발견

### 프롬프트 엔지니어링 3기법 + 조합

| 기법 | 효과 |
|------|------|
| System Prompt | 발견 이슈 0 → 3개, 전문가 수준 응답 |
| Few-shot | 출력 형식 통제 (발견 수는 0-shot이 최다) |
| Chain of Thought | 일반 요청 대비 문제 2개 추가 발견 |
| **3기법 조합** | **발견 이슈 2배, Before/After 코드 자동 생성** |

### 모델/도구 비교

| 비교 | 결과 |
|------|------|
| think vs ultra think | ultra think = 병렬 에이전트, 질적으로 다름 |
| Claude vs GPT | 각각 강점이 다름, 상황에 따라 선택 필요 |
| Claude Code vs Codex CLI | 빠른 핵심은 Codex, 상세 분석은 Claude Code |

### 새로운 영역 개척

- **Figma MCP**: 디자인 → Android 코드 자동 변환
- **Jetpack Compose**: 처음 접했지만 Claude Code로 점진적 마이그레이션 성공
- **Filesystem MCP**: 340파일 프로젝트 건강도 리포트 자동 생성

---

## 2차 실험의 핵심 메시지

> **"AI 답변이 마음에 안 들 때 어떤 기법을 꺼내 쓸지 아는 것"**

1차가 "AI 도구를 어떻게 쓰는가"였다면,  
2차는 "AI를 더 잘 제어하는 방법을 아는 것"이었다.

---

## 포트폴리오 v2

→ [AI를 활용한 Android 개발 — 2차 실험 기록](../PORTFOLIO_v2.md)

---

*작성일: 2026-05-06*
