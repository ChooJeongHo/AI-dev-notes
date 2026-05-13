# 035 - ADB + Claude Code로 UI Visual Regression Test 자동화

**UI 변경 전후 스크린샷을 자동 캡처하고 Claude가 차이를 분석하는 시각적 회귀 테스트**

---

## 목적

UI 코드 변경 시 의도하지 않은 화면 깨짐을 자동으로 감지하는 Visual Regression Test 파이프라인 구축.  
ADB로 before/after 스크린샷을 자동 캡처하고, Claude Code의 멀티모달 능력으로 비교 분석.

---

## 실험 내용

```
1. ADB로 4개 화면 자동 캡처 → before 폴더 저장
2. UI 코드 변경 (홈 그리드 2열 → 3열)
3. 빌드 후 after 폴더 저장
4. Claude Code가 before/after 이미지 비교 분석
5. 마크다운 리포트 자동 생성
```

---

## capture.sh — 자동 캡처 스크립트

```bash
#!/bin/bash
FOLDER=$1
PACKAGE="com.choo.moviefinder"

echo "📸 [$FOLDER] 캡처 시작"

adb shell am start -n $PACKAGE/.MainActivity
sleep 2

# 홈
adb shell input tap 135 2150
sleep 1
adb exec-out screencap -p > screenshot_test/$FOLDER/home.png
echo "✅ 홈 캡처 완료"

# 검색
adb shell input tap 405 2150
sleep 1
adb exec-out screencap -p > screenshot_test/$FOLDER/search.png
echo "✅ 검색 캡처 완료"

# 즐겨찾기
adb shell input tap 675 2150
sleep 1
adb exec-out screencap -p > screenshot_test/$FOLDER/favorite.png
echo "✅ 즐겨찾기 캡처 완료"

# 설정
adb shell input tap 945 2150
sleep 1
adb exec-out screencap -p > screenshot_test/$FOLDER/settings.png
echo "✅ 설정 캡처 완료"

echo "🎉 [$FOLDER] 전체 캡처 완료!"
```

**사용법:**
```bash
bash screenshot_test/capture.sh before   # 변경 전
bash screenshot_test/capture.sh after    # 변경 후
```

---

## 탭 좌표 파악 방법

스크린샷을 찍은 후 Claude Code에게 이미지를 분석시켜 탭 좌표를 자동으로 추출했다.

```
temp.png 보고 하단 탭바에서 홈/검색/즐겨찾기/설정 탭의 x,y 좌표 알려줘
```

Claude Code가 1080×2340 해상도 기준으로 4개 탭 좌표를 자동 산출했다.

| 탭 | X | Y |
|----|-----|------|
| 홈 | 135 | 2150 |
| 검색 | 405 | 2150 |
| 즐겨찾기 | 675 | 2150 |
| 설정 | 945 | 2150 |

---

## UI 변경 — 그리드 컬럼 2열 → 3열

```kotlin
// HomeFragment.kt
// Before
val spanCount = requireActivity().computeWindowWidthSizeClass().toMovieGridSpanCount()

// After
val spanCount = maxOf(requireActivity().computeWindowWidthSizeClass().toMovieGridSpanCount(), 3)
```

공용 함수(`toMovieGridSpanCount`)는 건드리지 않고 홈 화면에만 `maxOf()`로 최솟값 3을 강제해 태블릿 로직을 유지했다.

---

## before vs after 비교 결과

| 화면 | 변경 여부 | 변경 내용 |
|------|----------|---------|
| 홈 | ✅ 변경됨 | 그리드 2열 → 3열, 카드 크기 축소 |
| 검색 | ✅ 유지됨 | 변경 없음 |
| 즐겨찾기 | ✅ 유지됨 | 변경 없음 |
| 설정 | ✅ 유지됨 | 변경 없음 |

**홈 화면 상세 비교:**

| | before | after |
|---|--------|-------|
| 1행 영화 수 | 2개 (란 12.3, CONQUEROR) | 3개 (란 12.3, CONQUEROR, BEAST) |
| 카드 크기 | 크게 2개 | 작게 3개 |
| 2행 | 뒤바뀐 친구들, 슈퍼 마리오 | 뒤바뀐 친구들, 슈퍼 마리오 갤럭시, 정점 |

---

## 성능 측정 포함 (gfxinfo)

report.md에 스크롤 성능 데이터도 자동 포함됐다.

| 지표 | 수치 | 평가 |
|------|------|------|
| Janky frames | 0.93% | ✅ 3열 전환 후에도 정상 |

---

## 리포트 자동 생성

Claude Code가 before/after 이미지를 읽고 `screenshot_test/report.md`를 자동 작성했다.  
포함 내용: 4탭 비교표 + 코드 변경 내역 + gfxinfo 성능 분석.

---

## Visual Regression Test 흐름 요약

```
before 캡처 → UI 코드 수정 → 빌드 → after 캡처
    ↓
Claude Code가 이미지 비교 분석
    ↓
report.md 자동 생성 (변경된 화면 / 유지된 화면 / 성능 수치)
```

---

## 느낀 점

기존 Visual Regression Test 도구(Applitools, Percy 등)는 별도 설정과 유료 플랜이 필요하다.  
ADB + Claude Code 조합으로 **추가 도구 없이** 같은 결과를 낼 수 있었다.

특히 `capture.sh` 하나로 before/after를 동일한 순서로 자동 캡처하기 때문에 사람이 일일이 화면을 이동할 필요가 없다.  
이후 CI에 연결하면 PR마다 UI 회귀 테스트가 자동으로 돌아가는 파이프라인이 된다.

**34일차 gfxinfo 성능 측정에 이어, 35일차는 시각적 회귀 감지까지 ADB 자동화 범위를 확장했다.**

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*테스트 기기: Samsung Galaxy (R3CX80BS9MX)*  
*작성일: 2026-05-13*
