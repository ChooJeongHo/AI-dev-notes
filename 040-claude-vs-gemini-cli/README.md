# 040 - Claude Code vs Gemini CLI 코드리뷰 비교 실험

**동일한 MovieFinder 프로젝트에 Claude Code(Sonnet)와 Gemini CLI를 돌려서 품질·속도·접근 방식 비교**

---

## 목적

Claude Code만 써오다가 Google의 Gemini CLI를 직접 비교.  
"어떤 도구가 Android 개발 코드리뷰에 더 적합한가" 를 수치와 실제 탐지 이슈로 결론 도출.

---

## 실험 조건

- 동일한 MovieFinder 프로젝트
- 동일한 요청: "최근 커밋 기준 High/Medium/Low 이슈 분류"
- Claude Code: Sonnet 모델 + oh-my-claudecode code-reviewer
- Gemini CLI: v0.42.0 + gemini-2.5-flash-preview (무료 플랜)

---

## 결과 비교

| 항목 | Claude Code (Sonnet) | Gemini CLI |
|------|---------------------|-----------|
| High 이슈 | 2개 | 2개 |
| Medium 이슈 | 5개 | 3개 |
| Low 이슈 | 5개 | 2개 |
| 총 이슈 | 12개 | 7개 |
| 향후 로드맵 제안 | 없음 | 있음 |
| 파일 저장 | 자동 | 승인 필요 |

---

## 각 도구가 발견한 이슈

### 공통 발견
- `DetailViewModel` stale state 참조 문제 (WhileSubscribed + .value 레이스 컨디션)
- 대형 Fragment 비대화 (SearchFragment 777줄, DetailFragment 688줄)

### Claude Code만 발견
- `FavoriteFragment` retry 버튼 미연결 + 구독 영구 종료 (실제 기능 버그)
- `SavedStateHandle` 미적용 → 프로세스 사망 시 탭 상태 리셋
- `WorkManager` ViewModel 직접 주입 → SRP 위반
- `stateIn` 정책 의미 차이
- `collectCurrentMovies` 데드코드 catch 블록

### Gemini CLI만 발견
- `HomeFragment` 탭/Enum 하드코딩 결합도 문제
- `DatabaseModule` 333줄 비대화 → 도메인별 분리 제안
- `SearchViewModel` God Object 조짐 (영화/배우/기록 모든 책임 혼재)
- Compose 전환 로드맵 제안 (향후 방향성 포함)

---

## 도구별 특징 비교

| 특징 | Claude Code | Gemini CLI |
|------|------------|-----------|
| 접근 방식 | 버그/회귀 탐지 중심 | 아키텍처 개선 중심 |
| 강점 | 실제 런타임 버그, 코루틴/Flow 심층 분석 | 구조적 문제, 장기 로드맵 제안 |
| 파일 작업 | 자동 저장 | 매번 승인 필요 |
| CLAUDE.md/GEMINI.md | CLAUDE.md로 컨텍스트 최적화 가능 | GEMINI.md 지원하나 설정 미적용 |
| 서비스 안정성 | 안정적 | 6월 18일 Antigravity CLI로 전환 예정 |

---

## 핵심 발견

### "두 도구는 경쟁이 아니라 보완 관계"

Claude Code는 **"지금 당장 고쳐야 할 버그"** 를 잘 잡고,  
Gemini CLI는 **"장기적으로 개선해야 할 구조"** 를 잘 짚는다.

실제로 Gemini CLI가 발견한 `DatabaseModule` 분리, `SearchViewModel` 분리 제안은  
Claude Code가 놓쳤지만 장기 유지보수 관점에서 유효한 지적이었다.

반면 Claude Code가 발견한 retry 버튼 미연결, SavedStateHandle 미적용은  
실제 사용자가 경험할 수 있는 기능 버그였다.

---

## 작업별 도구 선택 가이드 (실험 결론)

| 상황 | 추천 도구 | 이유 |
|------|----------|------|
| PR 머지 전 버그 검수 | Claude Code | 런타임 버그 탐지율 높음 |
| 아키텍처 리팩토링 계획 | Gemini CLI | 구조적 개선 제안 강점 |
| 빠른 코드 품질 체크 | Gemini CLI | 설치 간단, 무료 플랜 |
| 코루틴/Flow 심층 분석 | Claude Code | 동시성 패턴 이해도 높음 |

---

## 느낀 점

두 도구를 써보니 "어떤 게 더 좋다"가 아니라 **"목적에 따라 골라 쓰는 것"** 이 맞다.

Claude Code는 현재 코드의 버그를 잡는 데 강하고,  
Gemini CLI는 코드를 어떻게 발전시킬지 방향을 잡는 데 강하다.

둘 다 쓸 줄 안다는 게 AI 인재로서의 차별점이 된다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-05-20*
