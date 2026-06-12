# 055 - /ultrawork 병렬 에이전트 실험 — 3가지 작업 동시 실행

**코드 품질 개선 + UI 버그 분석 + 테스트 추가를 8분 31초 만에 동시 완료**

---

## 목적

005일차 ulw 모드 실험의 현재 버전(/ultrawork) 재검증.  
3가지 독립적인 작업을 병렬로 동시에 실행해서 얼마나 효율적인지 확인.

---

## ultrawork란?

여러 executor 에이전트가 **독립적인 작업을 동시에** 실행하는 모드.  
코드 수정 작업은 **worktree 격리** 방식으로 충돌 방지.

---

## 실험 요청

```
1. 코드 품질 개선 — HIGH 이슈 중 아직 수정 안 된 것 3개 찾아서 수정
2. UI 버그 분석 — 주요 화면 스크린샷 찍고 레이아웃 이슈 찾기
3. 테스트 커버리지 — 커버리지 낮은 파일 찾아서 테스트 추가
```

**총 소요 시간: 8분 31초**

---

## 결과

### 작업 1 — 코드 품질 HIGH 이슈
이미 클린 상태 (Detekt 0 findings)  
→ 051일차 autoresearch, 047~049일차 모드 실험으로 다 정리된 상태

### 작업 2 — UI 버그 분석 (8개 이슈 발견)

| 화면 | 이슈 | 심각도 |
|------|------|--------|
| Home | 긴 제목 줄 잘림 (maxLines 부족) | 중 |
| Home | Recently Viewed 포스터 카드 과대 표시 | 중 |
| Search | 필터 칩 터치 영역 48dp 미달 가능성 | 하 |
| Search | 뷰 전환 아이콘 contentDescription 누락 | 하 |
| Favorite | 평점 필터 "5.0★" vs "3.0★+" 표기 불일치 | 하 |
| Settings | 루트 탭에 Back(←) 버튼 표시됨 | 중 |
| Settings | 마지막 항목이 bottom nav 뒤로 가려짐 | 중 |

### 작업 3 — 테스트 30개 추가 → main 머지 완료

| 파일 | 테스트 수 | 커버 내용 |
|------|---------|---------|
| SearchHistoryRepositoryImplTest | 13 | 공백 정규화, blank 체크, clearAll |
| PersonRepositoryImplCreditsTest | 9 | cast+crew 머지, 중복 제거, 날짜 정렬 |
| MovieRepositoryImplCollectionTest | 8 | 컬렉션 조회, 예외 래핑, 빈 목록 처리 |

---

## 005일차 ulw vs 055일차 ultrawork 비교

| 항목 | 005일차 ulw | 055일차 ultrawork |
|------|:-----------:|:----------------:|
| 작업 방식 | 순차 에이전트 | **병렬 동시 실행** |
| worktree 격리 | 없음 | **있음 (충돌 방지)** |
| 작업 수 | 1개 | **3개 동시** |
| 총 소요 시간 | - | **8분 31초** |
| 결과물 | 기능 추가 | 버그 발견 + 테스트 추가 |

---

## 핵심 발견

### worktree 격리 자동 적용

코드 수정 작업(1, 3번)이 충돌하지 않도록 자동으로 worktree 격리 모드로 실행됐다.  
개발자가 별도로 설정하지 않아도 Claude가 스스로 판단해서 격리 환경을 만든 것.

### 병렬 실행 토큰 사용량

| 에이전트 | tool uses | tokens |
|---------|-----------|--------|
| 코드 품질 개선 | 8 | 51.0k |
| UI 버그 분석 | 59 | 66.6k |
| 테스트 추가 | 63 | 107.2k |
| **합계** | **130** | **224.8k** |

---

## ★ Insight (Claude Code 발언)

> "Settings Back 버튼 이슈: NavigationUI.setupWithNavController()는 AppBarConfiguration의 top-level destination 목록을 기준으로 Up 버튼 표시 여부를 결정합니다. 모든 bottom nav destination을 AppBarConfiguration에 포함시켜야 루트 탭에서 ←가 사라집니다."

---

## 느낀 점

3가지 완전히 다른 작업을 8분 만에 동시에 처리했다.  
순차적으로 했다면 최소 30분 이상 걸렸을 작업이다.

특히 worktree 격리를 자동으로 적용한 게 인상적이었다. 두 에이전트가 같은 파일을 수정하면 충돌이 날 수 있는데, Claude가 스스로 판단해서 격리 환경을 만들었다.

**"병렬로 시키면 병렬로 한다" — ultrawork는 AI 개발 워크플로우의 멀티태스킹이다.**

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-06-12*
