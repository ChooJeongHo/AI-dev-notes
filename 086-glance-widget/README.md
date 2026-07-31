# 086 - Jetpack Glance 홈 화면 위젯 — "다른 프로세스"라는 제약이 코드에 어떻게 나타나는가

**081~083일차 KOFIC 박스오피스 도메인 로직은 100% 그대로 재사용됐지만, UI 레이어는 "홈 화면은 앱과 다른 프로세스"라는 근본 제약 때문에 완전히 새로 배워야 했다**

---

## 목적

070~085가 "AI 내부 로직/검증"에 집중했다면, 086일차는 Google I/O 2026에서 공식화된 **"Compose-first" 흐름**을 반영해 완전히 새로운 UI 기술(Glance)을 다룬다. 081~083일차에서 만든 KOFIC 박스오피스 데이터를 홈 화면 위젯으로 노출한다.

---

## 작업 1 — Glance란: 일반 Compose와 근본적으로 다른 이유

일반 Compose는 앱 자신의 프로세스 안에서 실제 View 트리를 직접 그린다. 하지만 **홈 화면(런처)은 앱과 완전히 다른 프로세스**라서, 앱이 그 화면에 View를 직접 그릴 수 없다.

```
[일반 Compose — 068/077일차]
@Composable 함수 → 실제 View 트리 → 같은 프로세스에서 직접 렌더링

[Glance — 086일차]
@Composable 함수 (제한된 컴포저블만) → RemoteViews로 변환 → 런처 프로세스에 전달 → 런처가 대신 그림
```

**이 제약이 코드에 실제로 나타나는 지점:**
- 사용 가능한 컴포저블이 `Text`/`Image`/`Column`/`Row`/`Box`/`LazyColumn` 등 **RemoteViews로 변환 가능한 것만으로 제한**됨 — 일반 Compose의 자유로운 `Modifier` 체인, 커스텀 `Canvas` 드로잉은 불가능
- 클릭 처리도 일반 `onClick` 람다가 아니라 `actionStartActivity()`/`actionRunCallback()` 같은 **인텐트/콜백 기반 액션**으로만 가능 — 클릭이 일어나는 시점에 앱 프로세스가 죽어있을 수도 있기 때문

---

## 작업 2 — 위젯 구현: 도메인 로직은 그대로, UI만 새로

```kotlin
class BoxOfficeGlanceWidget : GlanceAppWidget() {
    override suspend fun provideGlance(context: Context, id: GlanceId) {
        // 081/082/083일차에 이미 만든 UseCase를 그대로 주입해서 재사용
        val boxOffice = getDailyBoxOfficeWithTmdbMatchUseCase()

        provideContent {
            GlanceTheme {
                Column(
                    modifier = GlanceModifier
                        .fillMaxSize()
                        .clickable(actionStartActivity<MainActivity>())
                ) {
                    boxOffice.take(3).forEach { movie -> BoxOfficeGlanceRow(movie) }
                }
            }
        }
    }
}

class BoxOfficeWidgetReceiver : GlanceAppWidgetReceiver() {
    override val glanceAppWidget: GlanceAppWidget = BoxOfficeGlanceWidget()
}
```

**핵심 확인:** `BoxOfficeRepository`, `GetDailyBoxOfficeWithTmdbMatchUseCase`는 **새로 한 줄도 안 고치고 그대로 주입**됐다 — Clean Architecture에서 domain 레이어가 UI 기술(View/Compose/Glance)과 완전히 독립적으로 설계됐음을 실증하는 사례.

---

## 작업 3 — 데이터 갱신 전략: WorkManager 주기 실행

위젯은 화면이 항상 떠있는 상태가 아니므로, 앱을 열지 않아도 주기적으로 데이터를 갱신해야 한다.

```kotlin
val request = PeriodicWorkRequestBuilder<BoxOfficeWidgetUpdateWorker>(
    repeatInterval = 15, TimeUnit.MINUTES  // WorkManager 주기 작업의 시스템 최소 간격
).build()
```

Worker 안에서 위젯을 갱신한다:

```kotlin
override suspend fun doWork(): Result {
    BoxOfficeGlanceWidget().updateAll(applicationContext)
    return Result.success()
}
```

> `PeriodicWorkRequest`의 최소 주기가 15분으로 시스템에 고정돼 있어, 그보다 촘촘한 실시간 갱신은 애초에 불가능하다. 박스오피스 데이터는 하루 단위로 집계되므로 15분 주기도 충분히 촘촘하다.

---

## 작업 4 — 실기기 검증

081일차 이후 계속 지켜온 원칙대로, 컴파일/테스트 통과만 믿지 않고 **실제 홈 화면에 위젯을 배치**해서 확인했다.

| 확인 항목 | 결과 |
|---|---|
| 위젯 추가 (2x2 크기) | 정상 배치 |
| TOP 3 데이터 표시 | 실제 KOFIC 데이터와 일치 |
| 위젯 탭 → 앱 실행 | 정상 동작 |
| 다크모드 대응 | `GlanceTheme` 자동 적용 확인 |
| WorkManager 갱신 | 15분 후 데이터 갱신 확인 |

---

## 작업 5 — 정직한 재사용성 평가

| 레이어 | 재사용 여부 |
|---|---|
| Domain (Repository/UseCase) | 100% 그대로 재사용 — 새로 만든 코드 없음 |
| Data (캐시, 매칭 로직) | 100% 그대로 재사용 — 082/083일차 최적화 혜택 그대로 적용 |
| UI (화면 그리기) | 완전히 새로 작성 — 일반 Compose의 Modifier 체계, LazyColumn 스크롤 동작, 리컴포지션 최적화 지식이 그대로 통하지 않음 |
| 클릭 처리 | 완전히 새로 학습 — 람다 기반 onClick이 아니라 액션 기반 |

**정직한 결론:** 068/077일차에서 익힌 "선언형 UI로 생각하는 법" 자체(상태를 함수 파라미터로 흘려보내는 사고방식)는 도움이 됐지만, **구체적인 API와 제약은 거의 다 새로 배워야 했다.** 이는 070~085 시리즈에서 반복된 패턴과 같다 — 083일차(KOFIC 재사용성)에서도 "핵심 로직은 재사용, 엔드포인트 고유 부분은 새로 필요"라는 같은 구조의 결론이 나왔었다.

---

## Clean Architecture의 실질적 가치를 다시 확인

이번 실험에서 가장 인상 깊은 지점은, **UI 기술이 XML → Compose → Glance로 세 번 바뀌는 동안 domain 레이어는 단 한 번도 안 바뀌었다**는 것이다. 이게 바로 "Presentation 레이어를 UseCase가 몰라야 한다"는 원칙(CLAUDE.md에 명시)이 실전에서 증명된 사례다.

---

## 오늘 배운 것 — 한 줄 정리

1. Glance는 "홈 화면이 다른 프로세스"라는 제약 때문에, 컴포저블 종류와 클릭 처리 방식이 일반 Compose와 근본적으로 다르다.
2. Clean Architecture로 설계된 domain 레이어는 **UI 기술이 몇 번 바뀌어도 전혀 손댈 필요가 없다** — 이번 실험이 그 증거다.
3. 위젯 갱신은 WorkManager 주기 작업으로 처리하며, **최소 주기(15분)라는 시스템 제약**을 미리 알고 설계해야 한다.
4. "이전 기술에서 배운 지식이 재사용된다"는 건 API 차원이 아니라 **사고방식(상태 관리, 선언형 사고) 차원**에서 성립한다.
5. 실기기 검증은 위젯처럼 **다른 프로세스(런처)가 관여하는 UI**에서 특히 더 중요하다 — 에뮬레이터/컴파일만으로는 실제 배치 결과를 알 수 없다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*
*작성일: 2026-07-31*
