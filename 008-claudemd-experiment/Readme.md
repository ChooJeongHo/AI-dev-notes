# AI 활용 기록 008 - CLAUDE.md 활용 실험

> CLAUDE.md 유무에 따른 코드 생성 차이를 실험했으나, 예상과 다른 결과가 나온 기록

## 목적

프로젝트 전용 AI 지침서인 CLAUDE.md가 실제로 코드 생성 품질에 어떤 영향을 미치는지 검증  
동일한 기능을 CLAUDE.md 없이 vs 있을 때 구현하여 차이 비교

## 사용한 도구

- Claude Code CLI (터미널)
- MovieFinder CLAUDE.md (기존 작성본)
- 실험 대상 프로젝트: [MovieFinder](https://github.com/ChooJeongHo/MovieFinder)

---

## CLAUDE.md란?

프로젝트 루트에 두는 마크다운 파일로, Claude Code가 작업 시작 시 **자동으로 읽는 프로젝트 전용 지침서**

| 구분 | 스킬 (SKILL.md) | CLAUDE.md |
|------|----------------|-----------|
| 적용 범위 | Claude Code 전체 | 해당 프로젝트에서만 |
| 내용 | 특정 작업 절차/체크리스트 | 프로젝트 구조/패턴/규칙 |
| 트리거 | 키워드 감지 시 자동 로딩 | 프로젝트 진입 시 자동 로딩 |

---

## MovieFinder CLAUDE.md 구성

MovieFinder의 CLAUDE.md는 아래 내용을 담고 있다:

- 기술 스택 및 라이브러리 버전 전체 목록
- Clean Architecture 폴더 구조 상세 설명
- 레이어별 규칙 (Domain 순수 Kotlin, Presentation → Data 직접 참조 금지 등)
- 주요 기능별 구현 패턴 상세 기술
- 에러 처리 방식 (ErrorType, CancellationException rethrow 등)
- 테스트 패턴 (MockK, Turbine, StandardTestDispatcher)
- QA 완료 사항 체크리스트

---

## 실험: 즐겨찾기 화면 검색 기능 추가

### 실험 방법

1. `mv CLAUDE.md CLAUDE.md.bak` 으로 CLAUDE.md 비활성화
2. 동일 프롬프트로 기능 구현 요청
3. `git stash`로 결과물 초기화
4. `mv CLAUDE.md.bak CLAUDE.md` 으로 CLAUDE.md 복원
5. 동일 프롬프트 재요청
6. 두 결과 비교

### 요청한 프롬프트

```
MovieFinder에 즐겨찾기 화면에서 영화를 검색하는 기능을 추가해줘.
기존 패턴을 유지해줘.
```

---

## 결과 비교

### CLAUDE.md 없을 때

| 항목 | 내용 |
|------|------|
| FavoriteViewModel | _searchQuery StateFlow + filterByQuery 필터링 + onSearchQueryChanged() |
| fragment_favorite.xml | TextInputLayout 검색바 추가 (툴바 토글) |
| menu_favorite.xml | 검색 아이콘 메뉴 추가 |
| FavoriteFragment | 검색 토글, 텍스트 입력 연결, 빈 상태 메시지 분기 |
| strings.xml (ko/en) | 검색 관련 문자열 3개 추가 |
| 테스트 | 6개 추가 |
| 빌드/Detekt | 미기록 |

---

### CLAUDE.md 있을 때

| 항목 | 내용 |
|------|------|
| FavoriteViewModel | _searchQuery StateFlow + combine 필터링 + onSearchQueryChanged() |
| fragment_favorite.xml | TextInputLayout 검색바 추가 (툴바 토글) |
| menu_favorite.xml | 검색 아이콘 메뉴 추가 |
| FavoriteFragment | 검색 토글, 텍스트 연결, **탭 전환 시 검색어 초기화**, **키보드 제어** 추가 |
| strings.xml (ko/en) | 검색 관련 문자열 3개 추가 |
| 테스트 | 6개 추가 → 총 167개 |
| 빌드/Detekt | **All tests pass, Detekt passes 자동 검증** |

---

## 핵심 발견 — 차이가 거의 없었다

| 비교 항목 | CLAUDE.md 없음 | CLAUDE.md 있음 |
|-----------|---------------|---------------|
| 구현 내용 | 거의 동일 | 거의 동일 |
| 세밀한 UX | 기본 수준 | 탭 전환 시 초기화, 키보드 제어 추가 |
| 빌드/Detekt 검증 | 미기록 | 자동 명시 |
| 출력 형식 | 파일별 나열 | 컴포넌트별 구조화 |

---

## 왜 차이가 작았을까?

예상과 달리 결과물이 거의 비슷했던 이유를 분석했다.

**Claude Code는 CLAUDE.md가 없어도 기존 코드를 직접 읽는다**

Claude Code는 작업 전에 관련 파일들을 자동으로 탐색하고 읽는다.  
이미 코드가 쌓인 프로젝트에서는 CLAUDE.md 없이도 기존 패턴을 파악할 수 있다.

**CLAUDE.md가 진짜 빛나는 상황은 따로 있다**

| 상황 | CLAUDE.md 효과 |
|------|---------------|
| 완전 새 프로젝트 시작 | 매우 큼 (코드가 없어 참고할 패턴이 없음) |
| 여러 개발자 협업 | 매우 큼 (동일한 기준 공유) |
| 복잡한 아키텍처 규칙 강제 | 큼 (레이어 위반 방지) |
| 이미 코드가 많은 프로젝트 | 작음 (기존 코드가 사실상 CLAUDE.md 역할) |

**결론: MovieFinder는 이미 CLAUDE.md가 불필요할 만큼 코드가 잘 쌓여 있다**

역설적으로, 코드가 잘 정리된 프로젝트일수록 CLAUDE.md의 효과가 작아진다.  
CLAUDE.md는 **시작점이 없을 때 방향을 잡아주는 도구**라는 것을 직접 확인했다.

---

## 느낀 점

처음엔 CLAUDE.md 유무에 따라 코드 품질이 크게 달라질 거라고 예상했다.  
하지만 실제로는 차이가 거의 없었다.

이 결과 자체가 중요한 발견이다.  
Claude Code는 CLAUDE.md가 없어도 기존 코드를 읽고 패턴을 파악한다.  
즉, **좋은 코드가 쌓이면 CLAUDE.md 없이도 일관된 결과가 나온다.**

CLAUDE.md의 진짜 가치는 **새 프로젝트나 협업 환경에서 빠르게 컨텍스트를 공유하는 것**이라는 걸 알게 됐다.  
예상대로 되지 않은 실험도 배움이 있다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-03-25*
