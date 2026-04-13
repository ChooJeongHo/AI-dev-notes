# AI 활용 기록 016 - Exa MCP로 라이브러리 보안 취약점 및 버전 분석

> 실시간 웹 검색으로 MovieFinder 라이브러리 전체 보안 점검 및 버그 발견

## 목적

Exa MCP를 활용하여 프로젝트에서 사용 중인 라이브러리의 최신 버전과 보안 취약점을 실시간으로 검색하고 분석하는 실험

## 사용한 도구

- Claude Code CLI (터미널)
- Exa MCP (`exa-mcp-server`) — 실시간 웹 검색
- 대상 프로젝트: [MovieFinder](https://github.com/ChooJeongHo/MovieFinder)

---

## Context7 vs Exa 차이

| 구분 | Context7 | Exa |
|------|----------|-----|
| 검색 범위 | 공식 문서 | 웹 전체 (GitHub 이슈, 블로그, 뉴스 등) |
| 용도 | 라이브러리 API/사용법 | 최신 버그/이슈/버전 정보 |
| 실시간성 | 공식 문서 기준 | 실시간 웹 검색 |

---

## 실험 프롬프트

```
exa로 웹 검색해서 MovieFinder에서 사용 중인 라이브러리들 중
최신 버전이 나왔거나 보안 취약점이 있는 게 있는지 찾아줘.
build.gradle.kts 읽고 현재 버전 확인해서 비교해줘.
```

---

## 분석 결과 (2026-04-13 기준)

### ✅ 최신 상태 (업데이트 불필요)

| 라이브러리 | 현재 버전 | 최신 버전 |
|-----------|---------|---------|
| AGP | 9.1.0 | 9.1.0 |
| Hilt | 2.59.2 | 2.59.2 |
| OkHttp | 5.3.2 | 5.3.2 |
| Retrofit | 3.0.0 | 3.0.0 |
| Coil | 3.4.0 | 3.4.0 |
| Room | 2.8.4 | 2.8.4 |
| Navigation | 2.9.7 | 2.9.7 |
| WorkManager | 2.11.2 | 2.11.2 |

**보안 취약점(CVE): 없음** ✅

---

### ⚠️ 주의가 필요한 항목

**1. Detekt 2.0.0-alpha.2 — 직접 영향 버그 발견!**

`parallel = true` 설정 시 랜덤 컴파일러 실패 발생
([detekt#9121](https://github.com/detekt/detekt/issues/9121), 2026-03-06 오픈)

MovieFinder `app/build.gradle.kts`에 바로 이 설정이 있었음:
```kotlin
detekt {
    parallel = true  // ← 문제 설정
}
```

**수정:** `parallel = false`로 변경 → Detekt 정상 실행 확인

**2. Facebook Shimmer 0.5.0 — 사실상 방치된 라이브러리**

마지막 릴리즈가 2021년, Android 15 이상에서 예상치 못한 동작 가능성

**3. LeakCanary 2.14 — UI 버그 존재**

`debugImplementation` 전용이라 프로덕션 영향 없음

---

### 📋 CLAUDE.md 버전 불일치 발견

Exa 검색 과정에서 CLAUDE.md가 실제 버전과 다르다는 것도 발견됨:

| 항목 | CLAUDE.md (잘못됨) | 실제 버전 |
|------|-----------------|---------|
| OkHttp | 5.0.0-alpha.14 | 5.3.2 |
| kotlinx-serialization | 1.8.1 | 1.10.0 |
| kotlinx-datetime | 0.6.2 | 0.7.1 |
| Paging | 3.4.0 | 3.4.2 |
| WorkManager | 2.10.1 | 2.11.2 |

→ CLAUDE.md 직접 업데이트 완료

---

## 수행한 조치

### Detekt parallel 버그 수정

```kotlin
// 수정 전
detekt {
    parallel = true
}

// 수정 후
detekt {
    parallel = false
}
```

커밋 후 확인:
- Detekt 통과 ✅
- 유닛 테스트 전체 통과 ✅
- main 푸시 완료 ✅

---

## 느낀 점

Exa MCP는 Context7과 다르게 **GitHub 이슈, 릴리즈 노트, 커뮤니티 글**까지 실시간으로 검색한다.

공식 문서에는 없는 **버그 리포트나 알려진 이슈**를 찾는 데 특히 유용했다. Detekt `parallel = true` 버그처럼 공식 문서에는 나오지 않지만 실제로 CI를 불안정하게 만들 수 있는 문제를 찾아낸 것이 좋은 예시다.

**Exa MCP가 유용한 상황:**
- 라이브러리 업그레이드 전 이슈 사전 조사
- 보안 취약점 정기 점검
- 특정 버전에서 알려진 버그 확인

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-04-13*
