# 041 - GEMINI.md 설정 전후 비교 실험

**CLAUDE.md처럼 GEMINI.md도 컨텍스트 설정이 리뷰 품질에 영향을 주는가**

---

## 목적

38일차 CLAUDE.md 경량화 실험의 Gemini CLI 버전.  
GEMINI.md 설정 전후 동일한 코드리뷰 요청으로 품질 변화를 측정.

---

## GEMINI.md 생성 방법

Claude Code가 CLAUDE.md를 참고해서 Gemini CLI 전용으로 자동 변환.

| 항목 | CLAUDE.md | GEMINI.md |
|------|-----------|-----------|
| OMC/skills/hooks | 포함 | 제거 |
| 라이브러리 테이블 | 3개 분리 | 버전 포함 단일 테이블 |
| 섹션 순서 | 개요→스택→아키텍처→명령 | 명령 먼저 (즉시 실행 컨텍스트 우선) |
| 분량 | ~300줄 | ~170줄 (약 43% 압축) |

---

## before vs after 비교 (GEMINI.md 설정 전후)

| 항목 | before (GEMINI.md 없음) | after (GEMINI.md 적용) |
|------|------------------------|----------------------|
| High 이슈 | 2개 | 2개 |
| Medium 이슈 | 3개 | 3개 |
| Low 이슈 | 2개 | 2개 |
| 파일 읽기 전략 | PROJECT_HEALTH.md 위주 | GEMINI.md → 핵심 파일 직접 탐색 |
| 이슈 설명 깊이 | 표면적 | 코드 라인 수 직접 확인 후 구체적 설명 |
| 강점 섹션 | 없음 | 아키텍처 강점 별도 섹션 추가 |

---

## 핵심 발견

### 이슈 수는 동일, 분석 깊이는 향상

GEMINI.md 적용 전후 이슈 수는 동일했지만 분석 방식이 달라졌다.

**before**: PROJECT_HEALTH.md, TECHNICAL_DOCS.md 등 이미 생성된 문서를 주로 참조  
**after**: GEMINI.md를 베이스로 실제 소스 파일을 직접 읽고 라인 수까지 확인하며 분석

예시:
```
// before
SearchFragment.kt (777줄) → 대형 프래그먼트 문제

// after
wc -l SearchFragment.kt → 785줄 직접 확인 후 언급
git show 1fd1d0f → 최근 커밋 실제 diff 분석
```

### Claude Code vs Gemini CLI GEMINI.md 적용 후 최종 비교

| 항목 | Claude Code (Sonnet) | Gemini CLI (GEMINI.md 적용) |
|------|---------------------|---------------------------|
| High 이슈 | 2개 | 2개 |
| Medium 이슈 | 5개 | 3개 |
| Low 이슈 | 5개 | 2개 |
| 총 이슈 | 12개 | 7개 |
| 공통 발견 | stale state, Fragment 비대화 | stale state, Fragment 비대화 |
| 단독 발견 | retry 버그, SavedStateHandle | DatabaseModule 분리, SearchViewModel God Object |
| 아키텍처 강점 섹션 | 없음 | 있음 |

---

## GEMINI.md 작성 가이드라인 (실험 결론)

| 넣어야 할 것 | 빼도 되는 것 |
|------------|------------|
| 빌드 명령 (상단 배치) | Claude 전용 지시문 |
| 아키텍처 규칙 | OMC/skills/hooks |
| 패키지 구조 | 중복 섹션 |
| API 설정 | 버전 정보 (toml에 있음) |

---

## 3일차 실험 종합 결론 (038~041)

| 실험 | 핵심 결론 |
|------|---------|
| 038 CLAUDE.md 경량화 | 컨텍스트 26% 줄이니 품질 8.2→8.6 향상 |
| 039 모델별 비교 | Sonnet이 회귀 버그 탐지 최강, 비싼 모델 ≠ 좋은 리뷰 |
| 040 Claude vs Gemini | 버그 탐지는 Claude, 아키텍처 개선은 Gemini |
| 041 GEMINI.md 효과 | 이슈 수 동일, 분석 깊이와 탐색 방식 개선 |

**공통 교훈: AI 도구는 컨텍스트 설정이 절반이다.**

---

## 느낀 점

GEMINI.md를 설정하니 Gemini CLI가 이미 생성된 문서(PROJECT_HEALTH.md)에 의존하지 않고 실제 소스 파일을 직접 탐색했다.

38일차 CLAUDE.md 경량화 실험과 같은 패턴 — **"AI에게 주는 컨텍스트도 설계가 필요하다"**.

CLAUDE.md든 GEMINI.md든 "많이 넣는다고 좋은 게 아니라, 필요한 것만 잘 정리해서 넣는 것이 핵심"이라는 걸 두 도구 모두에서 확인했다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-05-21*
