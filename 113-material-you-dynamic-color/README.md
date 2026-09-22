# 113 - Material You 동적 색상 — "안 된다"고 오판할 뻔한 세 겹의 함정

**명제("M3 없이도 동적 색상 가능")는 맞았다 — 하지만 라이브러리 버그 2건과 삼성 특유의 수동 팔레트 선택 방식까지 세 겹의 함정이 겹쳐서, 각 단계마다 "적용됐다/안 됐다"를 성급히 오판할 뻔했다**

---

## 목적

MovieFinder에 Material You 동적 색상(사용자 배경화면 색상에 맞춰 앱 테마가 자동으로 바뀌는 Android 12+ 기능)을 추가한다.

---

## 선행 정리 — 프롬프트에 담긴 근거 없는 전제 처리

작업 시작 전, Claude Code가 프롬프트에 담긴 "091/098/112일차에서 이런 결론을 냈다"는 언급을 git 로그와 메모리 전체에서 검색했는데, 실제 기록과 일치하지 않았다. 이런 결론은 git 이력이나 memory 어디에도 없었다.

처리 방향: 출처 불명확한 과거 언급은 빼고, 순수하게 지금 코드 상태를 직접 확인한 사실만 근거로 진행하기로 했다.

---

## 작업 1 — 조사

Theme.MovieFinder는 Theme.MaterialComponents.DayNight.NoActionBar 기반, 색상은 전부 하드코딩. material 라이브러리 1.14.0으로 DynamicColors API 요구사항을 충족하고, minSdk 24에서도 API 31 미만은 자동 no-op이라 안전하다는 걸 확인했다.

---

## 작업 2 — 구현: 편의 메서드 한 줄로 시작했다가 버그 2개 발견

처음엔 DynamicColors.applyToActivitiesIfAvailable(this) 한 줄로 시작했지만, 실기기 검증 과정에서 라이브러리/AppCompat 관련 진짜 버그 2개를 찾아 고쳤다.

### 버그 1 — 인자 없이 호출하면 조용히 아무것도 안 함

Theme.MaterialComponents 계열엔 없는 attr(dynamicColorThemeOverlay)를 찾다가, 찾지 못해도 조용히 아무 동작도 안 했다(no-op). **material 1.14.0의 .class 파일을 직접 디컴파일해서 원인을 확정**했다.

### 버그 2 — Application.onCreate()에서 적용하면 AppCompatActivity가 덮어써버림

Application.onCreate()(Activity 생성 전)에서 적용하면, AppCompatActivity의 super.onCreate()가 베이스 테마를 재적용하면서 덮어써버렸다.

### 최종 코드

MainActivity.onCreate()의 super.onCreate() 직후에 명시적 오버레이를 지정해서 적용한다:

```kotlin
DynamicColors.applyIfAvailable(
    this,
    com.google.android.material.R.style.ThemeOverlay_Material3_DynamicColors_DayNight,
)
```

---

## 작업 3 — 테스트

MainActivityDynamicColorTest.kt 계측 테스트 2개 신규 작성 (API 게이팅 확인 + 앱 정상 기동 확인). 유닛 테스트 856개 전부 통과, detekt/컴파일 전부 통과.

---

## 작업 4 — 실기기 검증

### 1. 하드코딩 색상이 동적 색상 적용을 방해하는가

**방해하지 않았다.** 버그 2개를 고친 뒤 **Pixel 에뮬레이터와 삼성 실기기(SM-S926N) 양쪽 모두에서** 실제로 색이 바뀌는 걸 스크린샷으로 직접 확인했다.

### 2. 실기기에서만 드러난 것 — 처음엔 오판할 뻔했다

삼성 One UI는 **벽지를 바꿔도 시스템이 자동으로 색 팔레트를 만들어주지 않는다.** "배경화면 및 스타일 → 컬러 팔레트"라는 **별도 메뉴에서 직접 선택**해야만 시스템 값(`theme_customization_overlay_packages`)이 채워지고, 그래야 앱에도 반영된다.

> **처음엔 이걸 몰라서 "삼성은 안 된다"고 잘못 결론 내렸었는데, 실제로는 정상 동작하되 진입 방식이 다른 것이었다.**

### 3. 별개 발견 — 오늘 스코프 밖

`SearchFragment`의 `ComposeView` 2곳과 `OnboardingFragment`(전체 Compose)는 `MaterialTheme { }`을 옵션 없이 써서 `dynamicColorScheme()`이 미배선 — 여전히 M3 기본 보라색으로 남아있다 (별도 작업 필요).

---

## 작업 5 — 정리

> "M3 마이그레이션 없이도 동적 색상 적용 가능"이라는 전제 자체는 맞았지만, 그걸 증명하는 과정에서 **라이브러리 버그 2개(무인자 호출 no-op, AppCompat 타이밍 충돌)와 삼성 고유의 수동 팔레트 선택 방식까지 세 겹의 함정**이 겹쳐 있어서, 각 단계마다 "적용됐다/안 됐다"를 성급히 오판할 뻔했다.

최종적으로는 코드·테스트·양쪽 실기기 검증 모두 통과한 상태로 커밋(`ec88ae5`) 및 푸시 완료했다.

---

## 095~113을 관통하는 패턴 — 실기기 전용 버그, 여덟 번째, 그리고 "겹친 함정"이라는 새로운 변주

101/103/104/106/109/110/112일차에 이어, 113일차도 실기기에서만 드러나는 문제였다. 다만 이번엔 **함정이 세 겹으로 겹쳐 있어서, 하나를 고쳤다고 "이제 됐다"고 성급히 결론 내릴 뻔했다는 게 새로운 지점**이다.

| 종류 | 사례 |
|---|---|
| 우리 코드의 실수 | 101(리소스 타입), 103(API 오용), 104(커버리지 누락), 106(Intent 매칭), 109(ViewModel 스코프), 110(WorkManager 정책) |
| 우리가 고치려다 만든 회귀 | 112(API 24~32 영속화 시도 → 33+ 회귀) |
| 라이브러리 사각지대 + 기기별 UX 차이가 겹침 | 113(편의 메서드 no-op + AppCompat 타이밍 + 삼성 수동 팔레트 선택) |

---

## 오늘 배운 것 — 한 줄 정리

1. "이론적으로 가능하다"는 명제와 "표준 API로 바로 된다"는 건 다른 문제다 — 편의 메서드가 암묵적으로 특정 조건(M3 테마)을 가정하고 있을 수 있다.
2. 라이브러리 함수가 조건 불충족 시 조용히 no-op하면, **소스가 없을 때는 `.class` 파일을 직접 디컴파일**해서라도 원인을 확정하는 게 확실하다.
3. `Application.onCreate()`와 `Activity.onCreate()`는 테마 적용 시점에서 실행 순서가 중요하다 — 너무 일찍 적용하면 나중에 덮어써질 수 있다.
4. **제조사별 UX 차이(삼성의 수동 팔레트 선택)를 "버그"로 오판하지 않으려면, 그 기기의 설정 메뉴 구조까지 직접 확인**해야 한다 — 기능이 "안 된다"와 "진입 방식이 다르다"는 다른 결론이다.
5. 여러 함정이 겹쳐 있을 땐 **하나를 고쳤다고 성급히 "해결됐다"고 결론짓지 말고, 각 단계를 독립적으로 재검증**해야 한다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*
