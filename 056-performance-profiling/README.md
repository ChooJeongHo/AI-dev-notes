# 056 - Claude Code로 앱 성능 프로파일링 자동화

**에뮬레이터 Janky 31% → 실기기 5.41% — 에뮬레이터가 최대 11배 과장된다는 것을 수치로 증명**

---

## 목적

Android 앱 성능 측정을 Claude Code가 자동화.  
Cold Start, 렌더링, 메모리, 배터리를 한 번에 측정하고 PERFORMANCE_REPORT.md 자동 생성.

---

## 측정 항목 및 방법

| 항목 | 도구 |
|------|------|
| Cold Start | `adb shell am start-activity -W` |
| 렌더링 성능 | `adb shell dumpsys gfxinfo` |
| 메모리 사용량 | `adb shell dumpsys meminfo` |
| 배터리 소모 | `adb shell dumpsys batterystats` |

---

## 실기기 측정 결과 (Galaxy S24+ / Android 16)

| 항목 | 측정값 | 상태 |
|------|--------|------|
| Cold Start 평균 | **527ms** | ✅ 양호 |
| Janky Frame 비율 | **5.41% (4/74)** | ✅ 양호 |
| 50th 프레임 | **5ms** | ✅ 60fps 기준 3배 여유 |
| 90th 프레임 | **13ms** | ✅ 우수 |
| 95th 프레임 | **53ms** | ⚠️ 가끔 스파이크 |
| 총 메모리 (PSS) | **185MB** | ✅ 양호 |
| Graphics PSS | **73MB** | ✅ 영화 포스터 GPU 텍스처 정상 |
| 배터리 CPU 소모 | **2.37mAh / 28분** | ✅ 매우 낮음 |

---

## 핵심 발견 — 에뮬레이터 vs 실기기 최대 11배 차이

| 지표 | 에뮬레이터 | 실기기 (S24+) | 배율 |
|------|-----------|--------------|------|
| Cold Start | 828ms | 527ms | 1.6× |
| **Janky 비율** | **31.25%** | **5.41%** | **5.8×** |
| **50th 프레임** | **32ms** | **5ms** | **6.4×** |
| **90th 프레임** | **150ms** | **13ms** | **11.5×** |
| 99th 프레임 | 400ms | 150ms | 2.7× |
| GPU 파이프라인 | Skia (OpenGL) | Skia (Vulkan) | — |

**에뮬레이터에서 🔴였던 렌더링 항목들이 실기기에서 전부 ✅로 뒤집혔다.**

---

## 정적 분석 결과

| 항목 | 상태 |
|------|------|
| `onDraw` 내 객체 생성 | ✅ 없음 (Paint/RectF 모두 클래스 필드) |
| 어댑터 `dispose()` 호출 | ✅ 15곳 |
| Fragment `_binding = null` | ✅ 9개 |
| `ViewSizeResolver` | ✅ 9개 어댑터 |
| 정적 Context 참조 | ✅ 없음 |
| LinearLayout 중첩 | ⚠️ 30개 파일 (체감 영향 낮음) |

---

## 커스텀 뷰 5종 검증

`BarChartView`, `CalendarHeatmapView`, `PieChartView`, `CircularRatingView`, `HistogramView`  
→ 모두 `onDraw` 내 객체 할당 없음 ✅

---

## 선택적 개선 포인트

```kotlin
// 현재 — DataStore 손상 시 무한 블로킹 가능
val themeMode = runBlocking { repository.getThemeMode().first() }

// 개선안 — 100ms 초과 시 시스템 기본값 사용
val themeMode = runBlocking {
    withTimeoutOrNull(100) { repository.getThemeMode().first() } ?: ThemeMode.SYSTEM
}
```

---

## 느낀 점

에뮬레이터에서 Janky 31%라는 결과가 나와서 심각한 성능 문제가 있는 줄 알았는데, 실기기에서 측정하니 5.41%로 정상이었다.

에뮬레이터는 소프트웨어 OpenGL을 쓰고 실기기는 Vulkan GPU를 쓰기 때문에 최대 11배까지 차이가 날 수 있다는 것을 수치로 직접 확인했다.

**"성능 측정은 반드시 실기기로" — 에뮬레이터 수치를 믿으면 없는 문제를 만들어낼 수 있다.**

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*측정 기기: Samsung Galaxy S24+ (SM-S926N) / Android 16 (API 36)*  
*작성일: 2026-06-15*
