# AI를 활용한 Android 개발 — 2차 실험 기록

> 도구 활용에서 **AI를 더 잘 쓰는 방법**을 탐구하는 단계로의 발전

> 📅 **2차 실험: 2026년 4월 ~ 5월 (018~030일차)**  
> 1차 실험: [PORTFOLIO.md](./PORTFOLIO.md) (001~017일차)

---

## 1차에서 2차로 — 무엇이 달라졌나

| 구분 | 1차 (001~017) | 2차 (018~030) |
|------|-------------|-------------|
| 핵심 주제 | AI 도구 활용 및 자동화 구축 | AI를 더 잘 쓰는 방법 탐구 |
| 주요 실험 | GitHub Actions, MCP, AFK 모드 | 프롬프트 엔지니어링, 모델 비교 |
| 새로 배운 것 | 도구 설치/활용 | 프롬프트 설계, 도구 간 차이 판단 |

---

## 2차 실험 핵심 성과

| 지표 | 수치 |
|------|------|
| 실험 수 | 13개 (018~030) |
| 프롬프트 엔지니어링 실험 | 4개 (System Prompt / Few-shot / CoT / 조합) |
| 비교 실험 | 3개 (Claude vs GPT / Claude Code vs Codex CLI / think vs ultra think) |
| 새로 설치한 MCP | Figma MCP, Filesystem MCP |
| 새로 배운 기술 | Jetpack Compose, 프롬프트 엔지니어링 3기법 |

---

## 핵심 실험 요약

### 🧠 프롬프트 엔지니어링 4부작 (022~025일차)

AI 답변이 마음에 안 들 때 어떤 기법을 쓸지 아는 것이 AI 인재의 핵심 역량.

| 기법 | 효과 | 언제 쓸까 |
|------|------|---------|
| System Prompt | 분석 깊이 + 형식 제어 | 일관된 품질 반복 생성 |
| Few-shot | 출력 형식 통제 | 팀 내 표준화 |
| Chain of Thought | 근거 있는 깊은 분석 | 복잡한 판단, 아키텍처 결정 |
| **3기법 조합** | **발견 이슈 2배, Before/After 코드 자동** | 복잡한 코드 리뷰 |

### 🔍 모델/도구 비교 실험 (018, 026, 029일차)

| 실험 | 결과 |
|------|------|
| think vs ultra think | ultra think = 4개 에이전트 병렬, 30개+ 이슈 발견 |
| Claude vs GPT | GPT 이슈 더 많이 발견, Claude 트레이드오프 분석 강점 |
| Claude Code vs Codex CLI | 빠른 핵심은 Codex, 상세 코드 예시는 Claude Code |

### 🎨 Figma MCP — 디자인 → Android 코드 자동 변환 (021일차)

국비 수업 때 Figma로 디자인만 하고 코드로 못 뽑아냈던 것을 Figma MCP + Claude Code로 자동 변환.

- 116개 프레임 Figma 파일 분석
- `activity_main.xml` (567줄) 자동 생성
- Before/After: ViewBinding 5줄 → ComposeView 2줄

### 🚀 Jetpack Compose 마이그레이션 (027일차)

Compose를 처음 접하는 상태에서 Claude Code의 가이드만으로 XML → Compose 변환.

```
XML 패턴          →  Compose 패턴
ViewPager2        →  HorizontalPager
dot_0/dot_1/dot_2 →  repeat(pages.size) {} 한 줄
setBackgroundResource() → animateColorAsState()
```

dots 하드코딩 → `repeat` 한 줄로 바뀌는 순간이 선언형 UI의 핵심을 직접 느끼게 해줬다.

### 📊 Filesystem MCP — 프로젝트 건강도 리포트 (028일차)

Filesystem MCP로 MovieFinder 340개 파일 전체를 분석해서 `PROJECT_HEALTH.md` 자동 생성.

- 총 20,900줄 코드 분석
- 레이어별 분포, TOP 10 파일, 건강도 평가 자동화
- 예상치 못한 발견: Compose 도입 시 Detekt 설정도 업데이트 필요

---

## 2차 실험에서 배운 것

### "AI 답변이 마음에 안 들 때 어떻게 할지 아는 것"

도구를 쓸 줄 아는 것에서, AI를 더 잘 제어하는 방법을 아는 것으로 레벨업했다.

- 피상적인 답 → CoT 적용
- 형식이 안 맞으면 → Few-shot 적용  
- 전문성이 부족하면 → System Prompt 적용

### "어떤 도구를 언제 쓸지 판단하는 능력"

Claude Code, Codex CLI, GPT, Claude API — 각각의 강점이 다르다. 상황에 맞게 선택하는 판단력이 AI 인재의 역량이다.

### "모르는 기술도 AI와 함께 배울 수 있다"

Compose를 한 번도 안 써봤는데 Claude Code가 각 변환마다 설명해줘서 이해하면서 진행할 수 있었다.

---

## 앞으로의 방향

1차에서 도구 활용 능력을 쌓았고, 2차에서 AI를 더 잘 쓰는 방법을 배웠다.

다음 단계는 이 두 가지를 결합해서 **실제 업무 문제를 AI로 해결하는 능력**을 키우는 것이다.

---

*실험 프로젝트: [AI-dev-notes](https://github.com/ChooJeongHo/AI-dev-notes)*  
*대상 앱: [MovieFinder](https://github.com/ChooJeongHo/MovieFinder), MediScan*  
*기간: 2026년 4월 ~ 5월 (018~030일차)*
