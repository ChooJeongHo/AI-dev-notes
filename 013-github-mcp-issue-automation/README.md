# AI 활용 기록 013 - GitHub MCP로 코드 분석 및 이슈 자동 등록

> Claude Code가 GitHub MCP를 통해 코드를 직접 분석하고 개선점을 이슈로 자동 등록

## 목적

GitHub MCP를 Claude Code CLI에 연동하여 코드 품질 분석 자동화 실험  
사람이 직접 코드 리뷰하던 과정을 AI가 대신해서 GitHub 이슈까지 자동 생성

## 사용한 도구

- Claude Code CLI (터미널)
- GitHub MCP (`@modelcontextprotocol/server-github`, HTTP 방식)
- oh-my-claudecode mcp-setup 스킬
- 대상 프로젝트: [MovieFinder](https://github.com/ChooJeongHo/MovieFinder)

---

## GitHub MCP 설치 과정

### HTTP 방식 (GitHub Copilot 구독 필요)

```bash
claude mcp add --transport http -s user github \
  https://api.githubcopilot.com/mcp/ \
  --header "Authorization: Bearer $(gh auth token)"
```

oh-my-claudecode의 `/oh-my-claudecode:mcp-setup` 스킬로 대화형 설치도 가능

### 연결 확인

```
/mcp
```

```
github · ✔ connected   ← 이게 나오면 성공
```

---

## 실험 내용

### 1단계 - 기본 기능 확인

**이슈 목록 조회:**
```
GitHub MCP로 ChooJeongHo/MovieFinder 레포의 이슈 목록을 보여줘.
```

**커밋 목록 조회:**
```
GitHub MCP로 ChooJeongHo/MovieFinder 레포의 최근 커밋 목록 보여줘.
```

### 2단계 - 코드 분석 + 이슈 자동 등록

```
GitHub MCP로 ChooJeongHo/MovieFinder 레포의 코드를 분석해서
개선이 필요한 부분을 찾아 이슈로 자동 등록해줘.
최근 커밋 내용도 참고해서 아직 남은 개선점을 찾아줘.
```

---

## 결과

### 자동 등록된 이슈 16개

| 이슈 | 제목 | 심각도 | 라벨 |
|------|------|--------|------|
| #17 | Widget API 키가 URL 문자열에 직접 노출 | 🔴 High | bug |
| #18 | 백업 import에 입력값 유효성 검사 없음 | 🔴 High | bug |
| #19 | WatchHistoryGenreEntity 중복 삽입으로 통계 오염 | 🔴 High | bug |
| #20 | WatchHistory 장르 삽입이 원자적이지 않음 | 🟡 Medium | bug |
| #21 | 위젯 API 요청 언어 ko-KR 하드코딩 | 🟡 Medium | bug |
| #22 | CalendarHeatmapView 주 경계 판정 오류 | 🟡 Medium | bug |
| #23 | FavoriteFragment 태그 추천 코루틴 미취소 | 🟡 Medium | bug |
| #24 | SearchViewModel 인물 검색 레이스 컨디션 | 🟡 Medium | bug |
| #25 | FavoriteFragment WindowSizeClass 미적용 | 🟡 Medium | bug |
| #26 | PosterTagSuggester Bitmap recycle 누락 | 🟡 Medium | performance |
| #27 | 백업에 태그 및 시청 기록 미포함 | 🟡 Medium | enhancement |
| #28 | CertificatePinner 설정 3곳에 중복 | 🟡 Medium | enhancement |
| #29 | Fragment/CustomView/Adapter 테스트 없음 | 🟡 Medium | test |
| #30 | WatchHistoryRepositoryImpl 테스트 없음 | 🟡 Medium | test |
| #31 | SettingsFragment repeatOnLifecycle 7개 중복 | 🔵 Low | enhancement |
| #32 | StatsFragment 공유 이미지 px 하드코딩 | 🔵 Low | bug |

**분석 소요 시간:** 4분 (50 tool uses, 132.9k tokens)

---

## GitHub MCP로 할 수 있는 것들

| 기능 | 명령 예시 |
|------|----------|
| 이슈 조회 | 이슈 목록 보여줘 |
| 이슈 생성 | 제목/내용/라벨 지정해서 이슈 만들어줘 |
| PR 조회 | 열린 PR 목록 보여줘 |
| 커밋 조회 | 최근 커밋 20개 보여줘 |
| 파일 읽기 | 특정 파일 내용 읽어줘 |
| 브랜치 관리 | 브랜치 목록 보여줘 |

---

## 10일차와의 차이

| 구분 | 10일차 (GitHub Actions) | 13일차 (GitHub MCP) |
|------|------------------------|-------------------|
| 트리거 | PR 생성 시 자동 | 수동 요청 |
| 실행 환경 | GitHub 서버 | 로컬 Claude Code |
| 결과 | PR 코멘트 | GitHub 이슈 등록 |
| 활용 | CI/CD 자동화 | 코드 품질 관리 |

---

## 느낀 점

GitHub MCP가 단순히 이슈를 조회하는 수준이 아니라, 코드를 직접 분석해서 개선점을 찾고 이슈까지 자동으로 등록해주는 것이 인상적이었다.

4분 동안 코드 전체를 분석해서 보안 취약점(API 키 노출), 데이터 무결성 문제, 성능 이슈까지 세밀하게 잡아냈다. 특히 `WatchHistoryGenreEntity` 중복 삽입 버그처럼 사람이 놓치기 쉬운 사일런트 버그도 발견했다.

**GitHub MCP가 유용한 상황:**
- 새로운 기능 추가 후 사이드 이펙트 검토
- 코드 리뷰 전 사전 분석
- 기술 부채 정기 점검

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*등록된 이슈: #17 ~ #32*  
*작성일: 2026-04-06*
