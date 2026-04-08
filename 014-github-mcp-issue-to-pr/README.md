# AI 활용 기록 014 - GitHub MCP로 이슈 → 코드 수정 → PR 자동화

> 13일차에 생성한 이슈를 Claude Code가 직접 읽고, 버그를 수정한 후 PR까지 자동으로 올린 기록

## 목적

GitHub MCP + Claude Code 조합으로 **이슈 → 코드 수정 → PR 생성** 전체 흐름 자동화  
그리고 PR 생성 시 10일차/12일차 워크플로우가 자동으로 연동되는 것 확인

## 사용한 도구

- Claude Code CLI (터미널)
- GitHub MCP (이슈 조회 + PR 생성)
- 대상 이슈: [#19 WatchHistoryGenreEntity 중복 삽입](https://github.com/ChooJeongHo/MovieFinder/issues/19)
- 생성된 PR: [#34](https://github.com/ChooJeongHo/MovieFinder/pull/34)

---

## 실험 프롬프트

```
GitHub MCP로 MovieFinder 이슈 #19를 읽고,
해당 버그를 수정한 다음 새 브랜치 만들어서 PR 자동으로 올려줘.
```

딱 한 줄 요청으로 전체 흐름이 자동으로 처리됨

---

## Claude Code가 수행한 전체 흐름

```
1. GitHub MCP로 이슈 #19 내용 읽기
        ↓
2. 버그 원인 분석
   - WatchHistoryGenreEntity에 OnConflictStrategy 없어서 중복 삽입 발생
   - 동일 영화 N번 시청 시 장르 카운트가 N배로 부풀어 통계 왜곡
        ↓
3. 브랜치 생성
   git checkout -b fix/watch-history-genre-duplicate
        ↓
4. 코드 수정 (4개 파일)
        ↓
5. 커밋 + 푸시
        ↓
6. GitHub MCP로 PR #34 자동 생성
   - 제목, 본문, Closes #19 링크까지 자동 작성
```

---

## 수정된 코드

### WatchHistoryGenreEntity.kt
```kotlin
// 수정 전
indices = [Index("watch_history_id"), Index("genre_name")]

// 수정 후 — (watch_history_id, genre_name) UNIQUE 복합 인덱스 추가
indices = [
    Index("watch_history_id"),
    Index("genre_name"),
    Index(value = ["watch_history_id", "genre_name"], unique = true)
]
```

### WatchHistoryDao.kt
```kotlin
// 수정 전
@Insert
suspend fun insertGenres(genres: List<WatchHistoryGenreEntity>)

// 수정 후 — 중복 시 무시
@Insert(onConflict = OnConflictStrategy.IGNORE)
suspend fun insertGenres(genres: List<WatchHistoryGenreEntity>)
```

### MovieDatabase.kt
```kotlin
// DB 버전 13 → 14
```

### DatabaseModule.kt
```kotlin
// MIGRATION_13_14 추가
// 기존 테이블 재생성 + 중복 데이터 정리 + 인덱스 재생성
```

---

## PR 생성 후 자동 연동된 것들

PR #34가 생성되자마자 기존 워크플로우들이 자동으로 실행됨:

| 워크플로우 | 결과 | 출처 |
|-----------|------|------|
| Claude AI Code Review | **Overall Grade: A** | 10일차 |
| JaCoCo 커버리지 리포트 | Line 74%, Branch 48% | 12일차 |
| GitGuardian Security | No secrets detected ✅ | 자동 |
| Android CI | Build passed ✅ | 기존 |

### Claude 자동 코드 리뷰 결과

| 카테고리 | 등급 |
|---------|------|
| Architecture | A |
| Coroutine Safety | A |
| Memory Management | A |
| Error Handling | B |
| Android Best Practices | A |
| Code Quality | A |
| **Overall** | **A** |

---

## 1~14일차 연결 구조

```
001~003 — Claude Code 기본 + 스킬
    ↓
004~007 — oh-my-claudecode (ulw/team/ralph)
    ↓
008     — CLAUDE.md 실험
    ↓
009     — playwright-cli 자동화
    ↓
010     — GitHub Actions + Claude API 코드 리뷰 ←──┐
    ↓                                              │ PR #34에서 자동 연동
011     — Context7 MCP + ML Kit                    │
    ↓                                              │
012     — JaCoCo 커버리지 개선 ←───────────────────┤
    ↓                                              │
013     — GitHub MCP 이슈 자동 생성                │
    ↓                                              │
014     — 이슈 → 코드 수정 → PR 자동화 ────────────┘
```

---

## 느낀 점

한 줄 프롬프트로 이슈를 읽고, 원인을 분석하고, 코드를 수정하고, PR을 올리기까지 전부 자동으로 처리됐다.

더 인상적인 건 PR이 올라가자마자 10일차에 만든 코드 리뷰와 12일차에 개선한 커버리지 체크가 자동으로 실행된 것이다. 14일 동안 만든 것들이 하나의 워크플로우로 연결되는 순간이었다.

AI가 단순히 코드를 짜주는 게 아니라, **개발 프로세스 전체를 자동화**할 수 있다는 걸 직접 확인했다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*PR #34: https://github.com/ChooJeongHo/MovieFinder/pull/34*  
*작성일: 2026-04-08*
