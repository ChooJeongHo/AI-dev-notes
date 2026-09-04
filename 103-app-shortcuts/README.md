# 103 - App Shortcuts — 정적분석을 전부 통과한 코드가 "롱프레스→탭→뒤로가기" 흐름에서만 무너졌다

**컴파일·유닛테스트 855개·detekt·아키텍처 리뷰까지 전부 통과했는데, 실기기에서 실제로 단축키를 눌러보고서야 popBackStack API 오용 버그를 발견했다 — 레이어 위반이나 코루틴 문제가 아니라 "API 의미를 잘못 이해한" 새로운 유형의 버그였다**

---

## 목적

086(Glance) → 101(Predictive Back) → 102(Splash Screen)로 이어온 "최신 Android UX 기능 조사 후 안전한 범위로 적용" 패턴을 App Shortcuts에도 적용한다.

---

## 작업 1 — 조사

- App Shortcuts 관련 코드/리소스가 전혀 없었다 (shortcuts.xml 없음, ShortcutManagerCompat 사용 이력 없음).
- nav_graph.xml에 searchFragment/favoriteFragment는 있지만 딥링크가 미설정 — moviefinder://stats, moviefinder://movie/{id} 패턴을 재사용 가능하다고 판단.
- WatchHistoryDao.getRecentHistory() + GetWatchHistoryUseCase 기존 인프라로, 새 DAO 쿼리 없이 동적 단축키 구현이 가능함을 확인.

---

## 작업 2 — 설계

- 새 Repository 계층 대신 기존 ReleaseNotificationScheduler 패턴(core 래퍼 + ViewModel 직접 주입)을 재사용하기로 결정.
- 정적 단축키는 moviefinder://search/moviefinder://favorite 딥링크만으로 충분, 동적 단축키는 기존 moviefinder://movie/{id} 그대로 재사용.
- Room/DI 모듈 변경 없이, 파일 12개(신규 8 + 수정 4)로 범위를 확정.

---

## 작업 3 — 구현 (~26분)

신규 8개(GetRecentShortcutMoviesUseCase, AppShortcutManager, 아이콘 3종, shortcuts.xml, 테스트 2개) + 기존 4개(DetailViewModel, MainActivity, AndroidManifest, nav_graph.xml) 수정. 컴파일/유닛테스트 855개/detekt 전부 1차 통과.

---

## 작업 4 — 테스트

GetRecentShortcutMoviesUseCaseTest(중복제거/limit/빈목록), AppShortcutManagerTest(개수상한/ID포맷/딥링크), DetailViewModelTest(단축키 갱신 실패 시 UI 영향 없음) 작성. 도중 Stop 훅(테스트 페어링 게이트)이 여러 차례 걸렸으나, 서브에이전트가 결국 전부 작성해서 통과했다.

---

## 작업 5 — 실기기 검증에서 잡힌 진짜 버그들

### 정적분석 단계에서 잡힌 것 (kotlin-architecture-reviewer 재검토)

- Major 1건: AppShortcutManager에 동시성 Mutex 누락
- Minor 1건: 메인스레드 블로킹

→ Mutex + withContext(Dispatchers.IO) 적용으로 수정.

### 실기기 롱프레스 UI 테스트에서만 잡힌 것

MainActivity의 popBackStack(onboarding, inclusive=true) 오용으로, 즐겨찾기/검색 단축키를 눌렀을 때 온보딩 화면으로 되돌아가는 버그가 있었다.

```kotlin
// Before: popBackStack만으로는 목적지 화면으로 정확히 이동 안 됨
popBackStack(R.id.onboardingFragment, inclusive = true)

// After: navigate + popUpTo로 명확한 목적지 지정
navigate(target, popUpTo(R.id.onboardingFragment) { inclusive = true })
```

수정 후 재검증: 정적 2개/동적 2개 단축키 롱프레스 메뉴 노출, 검색/즐겨찾기/영화 단축키 전부 정확한 화면 콜드 스타트 이동, 검색/즐겨찾기는 뒤로가기 시 정상 종료까지 전부 확인 완료.

동적 영화 단축키에서 뒤로가기 시 온보딩이 재등장하는 현상은 movie/stats 딥링크 공통의 기존 선재 결함(이번 작업 범위 밖, 회귀 아님)으로 확인했다.

---

## 101/102/103일차 비교 — 실기기 검증이 잡아내는 버그의 종류가 점점 다양해졌다

| | 검증 방법 | 잡아낸 문제의 성격 |
|---|---|---|
| 101일차 | 실기기 스와이프 | 코드가 아니라 리소스 타입(res/anim vs res/animator) |
| 102일차 | 조사만, 코드 변경 없음 | 애초에 처음부터 잘 되어 있었음 |
| 103일차 | 실기기 롱프레스→탭→뒤로가기 UI 흐름 | 레이어/코루틴 규칙 위반이 아니라 "API 의미를 잘못 이해한" 버그 |

popBackStack을 "그냥 이전 화면으로 돌아가는 API"로 오해하고 썼는데, 실제로는 "현재 백스택에서 지정한 목적지까지 pop만 하고, 새 화면으로 이동은 안 시키는" API였다. 이건 코드 리뷰 체크리스트(레이어 위반, 코루틴 안전성 등)가 애초에 다루지 않는 범주의 버그다 — 컴파일도 되고, 정적분석도 통과하고, 로직도 "그럴듯해 보이는" 코드가 실제 사용자 흐름에서만 무너진 사례.

---

## 오늘 배운 것 — 한 줄 정리

1. 정적분석(컴파일/테스트/detekt/아키텍처 리뷰)을 전부 통과해도, "API를 의도한 대로 정확히 썼는가"는 다른 차원의 검증이 필요하다.
2. popBackStack류의 네비게이션 API는 이름만 보고 짐작하면 오용하기 쉽다 — "pop만 하는가, 이동까지 하는가"를 정확히 구분해야 한다.
3. 롱프레스 메뉴 → 탭 → 뒤로가기까지 이어지는 전체 사용자 흐름을 실기기에서 끝까지 따라가야 이런 종류의 버그가 드러난다.
4. 우연히 발견한 다른 화면의 선재 결함은, 지금 작업과 무관하면 "회귀 아님"으로 명확히 구분해서 범위 밖에 남겨두는 것이 맞는 판단이다.
5. 기존 인프라(WatchHistoryDao, ReleaseNotificationScheduler 패턴)를 재사용하면, 새 기능도 새 Repository나 DAO 없이 안전하게 확장할 수 있다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*
*작성일: 2026-09-04*
