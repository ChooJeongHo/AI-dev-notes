# 042 - 멀티 리뷰 통합 → 버그 분류 → GitHub Issues 자동 등록 파이프라인

**5개 코드리뷰 결과를 통합 분석해서 중복 제거 후 우선순위 분류, GitHub Issues까지 자동 등록**

---

## 목적

038~041일차 4일간의 코드리뷰 실험에서 나온 이슈들을 하나로 통합.  
중복 제거 + 우선순위 분류 + GitHub Issues 자동 등록까지 풀 파이프라인 완성.

---

## 파이프라인 흐름

```
5개 리뷰 파일 병렬 읽기
    ↓
52건 원시 이슈 추출
    ↓
중복 제거 (같은 이슈를 다른 리뷰어가 발견한 경우 통합)
    ↓
36건 고유 이슈 분류 (High/Medium/Low × 카테고리)
    ↓
HIGH 9건 GitHub Issues 병렬 자동 등록 (#62~#70)
```

---

## 분석 소스

| 파일 | 리뷰어 | 설명 |
|------|--------|------|
| claude_md_test_before.md | Opus (CLAUDE.md 경량화 전) | 038일차 |
| claude_md_test_after.md | Opus (CLAUDE.md 경량화 후) | 038일차 |
| model_test_haiku.md | Haiku | 039일차 |
| model_test_sonnet.md | Sonnet | 039일차 |
| gemini_review.md | Gemini CLI | 040~041일차 |

---

## 분류 결과

| 심각도 | 건수 | 주요 카테고리 |
|--------|------|-------------|
| HIGH | 9건 | 동시성 3, UI 2, 아키텍처 4 |
| MEDIUM | 15건 | 아키텍처 9, 성능 3, 동시성 2, UI 1 |
| LOW | 12건 | 아키텍처 6, 동시성 3, 성능 2, UI 1 |
| **합계** | **36건** | 원시 52건 → 중복 제거 후 |

---

## 등록된 HIGH 이슈 (#62~#70)

| 이슈 | 제목 | URL |
|------|------|-----|
| H-C1 | DetailViewModel loadingMutex 영구 잠금 | #62 |
| H-C2 | StatsViewModel retryTrigger race | #63 |
| H-C3 | DetailViewModel toggleWatchlist stale read | #64 |
| H-U1 | FavoriteFragment 강제 scroll-to-top 회귀 | #65 |
| H-U2 | FavoriteFragment retry 버튼 미연결 | #66 |
| H-A1 | FavoriteViewModel SavedStateHandle 미적용 | #67 |
| H-A2 | SearchHistoryRepository 정규화 불일치 | #68 |
| H-A3 | RemoteMediator initialize() 오프라인 가드 | #69 |
| H-A4 | HomeFragment 탭/Enum 매핑 취약 구조 | #70 |

각 이슈에 **파일 위치 + 재현 시나리오 + 수정 방향 + 발견 리뷰어** 자동 포함.

---

## 핵심 발견 — 중복 발견 빈도 = 위험도 신호

```
WHITESPACE_REGEX 상수화 → 4개 리뷰어 독립 발견 (가장 높은 중복 빈도)
loadingMutex tryLock 위치 → 3개 리뷰어 발견 (설명이 모두 달라 — 복잡성 반증)
stale read 문제 → 3개 리뷰어 발견
```

**중복 빈도가 높을수록 실제 위험도가 높다** — 여러 AI가 독립적으로 같은 이슈를 발견했다는 건 그만큼 코드에서 뚜렷하게 보인다는 의미다.

---

## 즉시 수정 권장 TOP 3

| 순위 | 이슈 | 이유 |
|------|------|------|
| 1 | H-C1 loadingMutex | 코드 3줄 이동으로 해결, 위험도 최고 |
| 2 | H-U2 retry 버튼 미연결 | 사용자가 에러 후 복구 방법 없는 상태 |
| 3 | H-A1 SavedStateHandle | 프로세스 종료 시 탭 상태 초기화 회귀 |

---

## 013일차 vs 042일차 비교

| 항목 | 013일차 | 042일차 |
|------|---------|---------|
| 이슈 소스 | 코드 분석 1회 | 5개 리뷰 통합 |
| 중복 제거 | 없음 | 52→36건 자동 제거 |
| 우선순위 분류 | 없음 | High/Medium/Low × 카테고리 |
| 등록 방식 | 순차 | 병렬 (9건 동시) |
| 이슈 내용 | 제목+설명 | 위치+재현+수정방향+발견자 |

---

## 느낀 점

코드리뷰를 여러 번 돌리다 보면 이슈가 쌓이는데, 사람이 직접 정리하면 시간이 걸린다.  
Claude Code가 5개 파일을 병렬로 읽고 중복을 제거해서 우선순위까지 자동 분류한 게 인상적이었다.

특히 **"중복 발견 빈도 = 위험도 신호"** 라는 인사이트가 새로웠다.  
여러 AI가 독립적으로 같은 이슈를 발견했다는 건, 그 이슈가 코드에서 뚜렷하게 보인다는 의미다.

**발견 → 분류 → 등록 파이프라인이 완성됐다. 앞으로는 코드리뷰 후 이 파이프라인을 자동으로 돌리면 된다.**

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-05-22*
