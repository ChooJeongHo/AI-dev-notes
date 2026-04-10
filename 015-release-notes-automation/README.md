# AI 활용 기록 015 - GitHub MCP + gh CLI로 릴리즈 노트 자동 생성

> 커밋 히스토리를 분석해서 릴리즈 노트를 자동 작성하고 GitHub Release에 등록

## 목적

GitHub MCP로 커밋 전체를 분석하여 카테고리별 릴리즈 노트를 자동 작성하고,
`v1.0.0` GitHub Release로 등록하는 실험

## 사용한 도구

- Claude Code CLI (터미널)
- GitHub MCP (커밋 조회)
- gh CLI (Release 생성 — GitHub MCP 미지원 기능 우회)
- 대상 프로젝트: [MovieFinder](https://github.com/ChooJeongHo/MovieFinder)

---

## 실험 프롬프트

```
GitHub MCP로 ChooJeongHo/MovieFinder 레포의 최근 커밋들을 분석해서
릴리즈 노트를 자동으로 작성해줘.
버전은 v1.0.0으로 하고, 카테고리별로 정리해줘.
```

---

## 흥미로운 발견 — 도구 부재 시 자동 우회

```
GitHub MCP → Release 생성 툴 없음
    ↓
Claude Code가 스스로 gh CLI로 우회
    ↓
gh release create v1.0.0 --target main ...
    ↓
Release 정상 등록
```

GitHub MCP에 Release 생성 기능이 없자 Claude Code가 스스로 `gh CLI`로 방법을 바꿔서 처리했다.
막히면 다른 도구로 우회하는 자율적인 문제 해결 방식이 인상적이었다.

---

## 자동 생성된 릴리즈 노트 구조

### 🎬 New Features
- 홈 화면 3탭 (현재 상영작 / 인기 영화 / 일별 트렌딩)
- 오프라인 캐시 (RemoteMediator 기반 1시간 캐시)
- 실시간 검색 + 연도/장르/정렬 필터
- 인물 검색 → 상세 프로필 / 필모그래피
- 7개 API 병렬 로딩 (영화 상세)
- 통계 화면 9개 카드 (파이차트, 바차트, 히스토그램, 캘린더 히트맵)
- ML Kit 태그 추천, 홈 화면 위젯, 개봉일 알림, 데이터 백업 등

### 🐛 Bug Fixes
- WatchHistoryGenreEntity 중복 삽입 통계 오염 수정 (#19)
- Bitmap.Config.HARDWARE API 26 미만 크래시 수정
- 딥링크 유효하지 않은 movieId 가드 추가 등

### ⚡ Performance
- Room 장르 정규화 테이블 (v13)
- Coil ViewSizeResolver 다운샘플링
- Baseline Profile 주요 경로 확장 등

### 🏗 Architecture & Refactoring
- Repository ISP 분리 (God Class → 9개 인터페이스)
- launchWithErrorHandler 공통 패턴 추출 등

### 🔒 Security & Resilience
- Certificate Pinning + 주간 CI 자동 검증
- ExponentialBackoff, RateLimiter 등

### 🧪 Tests
- 유닛 테스트 0 → 389개
- JaCoCo 커버리지 26.3% → 74.8%

### 🔧 CI/CD & Tooling
- GitHub Actions 3개 워크플로우
- Pre-commit / Pre-push Hook
- Dependabot 자동 버전 업데이트

---

## 결과

- GitHub Release v1.0.0 등록 완료
- URL: https://github.com/ChooJeongHo/MovieFinder/releases/tag/v1.0.0

---

## 느낀 점

커밋 메시지만 잘 써놓으면 Claude Code가 전체 히스토리를 읽고 카테고리별로 깔끔하게 정리해준다.

GitHub MCP에 Release 생성 기능이 없었는데 Claude Code가 스스로 `gh CLI`로 우회한 부분이 인상적이었다. 하나의 도구가 막혀도 다른 도구를 찾아서 목적을 달성하는 방식이 실무에서도 유용할 것 같다.

**릴리즈 노트 자동화가 유용한 상황:**
- 스프린트 마무리 시 변경사항 정리
- 오픈소스 프로젝트 버전 릴리즈
- 팀 내 개발 진행상황 공유

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*릴리즈: https://github.com/ChooJeongHo/MovieFinder/releases/tag/v1.0.0*  
*작성일: 2026-04-10*
