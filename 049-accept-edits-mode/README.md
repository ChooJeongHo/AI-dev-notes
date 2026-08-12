# 049 - Claude Code Accept Edits Mode 실험

**파일 수정은 자동, bash는 수동 승인 — Auto/Plan Mode와 비교해서 4가지 모드 완전 비교 완성**

---

## 목적

047 Auto Mode, 048 Plan Mode에 이어 Accept Edits Mode 실험.  
동일한 요청으로 4가지 모드를 모두 비교해서 어떤 상황에 어떤 모드가 적합한지 결론 도출.

---

## Accept Edits Mode란?

- 파일 읽기/수정: **자동 승인**
- bash 명령(컴파일, 테스트, 커밋 등): **수동 승인 필요**

Shift+Tab 한 번으로 활성화 (`accept edits on` 표시).

---

## 실험 결과

**요청:**
```
MovieFinder 앱의 코드 품질 개선점 3가지를 찾아서 직접 수정하고 커밋해줘
```

**총 소요 시간: 7분 0초**

---

## 자동 수정된 내용

| # | 개선 | 파일 | 효과 |
|---|------|------|------|
| 1 | 4개 PagingSource 중복 제거 → `BaseMoviePagingSource` (Template Method 패턴) | data/paging/ 5개 파일 | 각 ~40줄 → ~15줄, CancellationException 처리 단일화 |
| 2 | TMDB API 매직 스트링 상수화 ("Director", "YouTube", "Trailer", "KR", "US") | Constants.kt, MovieRepositoryImpl.kt, MovieApiService.kt | 오타가 컴파일 에러로 잡힘 |
| 3 | `ImageView.loadPoster()` 확장 함수 — 3곳 Coil 로딩 중복 통합 | PosterImageViewExt.kt 신규, ViewHolder 2개 + HorizontalMovieAdapter | crossfade/placeholder 정책 단일 관리 |

**결과: 12개 파일, +107/-165 (순감소 58줄)**  
**검증: Detekt ✅ · 유닛 테스트 509개 전체 통과 ✅ · Pre-commit 훅 통과 ✅**

---

## 3가지 모드 최종 비교

| 항목 | Auto Mode (047) | Plan Mode (048) | Accept Edits (049) |
|------|:--------------:|:---------------:|:-----------------:|
| 소요 시간 | 6분 29초 | **4분 41초** | 7분 0초 |
| 계획 확인 | ❌ | ✅ | ❌ |
| 파일 수정 승인 | 자동 | 자동 | 자동 |
| bash 승인 | 자동 | 자동 | **수동** |
| Detekt 오류 | 발생 후 수정 | **없음** | **없음** |
| 변경 규모 | 4파일 | 3파일 | **12파일** |
| 순 코드 변화 | - | - | **-58줄** |

---

## 4가지 모드 상황별 추천 가이드

| 상황 | 추천 모드 | 이유 |
|------|----------|------|
| 복잡한 멀티파일 리팩토링 | **Plan** | 계획 확인으로 방향 오류 사전 방지 |
| 장시간 자율 작업 | **Auto** | 개입 없이 완주 |
| bash 명령이 걱정될 때 | **Accept Edits** | 파일은 자동, 명령은 내가 확인 |
| 중요한 코드 (실수 방지) | **Plan** | 실행 전 전체 계획 검토 |
| 단순 파일 수정 | **Default** | 가볍게 수동 확인 |

---

## ★ Insight (Claude Code 발언)

> "품질 리팩토링의 좋은 신호: 코드가 줄면서(−58줄) 기능은 그대로입니다. 509개 유닛 테스트가 수정 없이 전부 통과한 것이 동작 보존의 증거입니다 — 테스트가 구현이 아닌 public API 계약을 검증하도록 작성돼 있었기에 가능했습니다."

---

## 느낀 점

Accept Edits Mode는 "파일은 믿지만 bash는 내가 확인하고 싶다"는 상황에 딱 맞는 모드였다.

3개 실험을 통해 모드별 특성이 명확해졌다:
- **빠름**: Plan Mode (4분 41초)
- **무인**: Auto Mode (개입 0회)
- **안전**: Plan Mode (Detekt 오류 0건)
- **대규모**: Accept Edits (12파일, -58줄)

**"모드를 상황에 맞게 선택할 줄 아는 것" 자체가 Claude Code를 잘 쓰는 능력이다.**

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-06-04*
