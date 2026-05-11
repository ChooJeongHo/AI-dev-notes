# 033 - ADB + Claude Code로 실기기 UI 자동 테스트

**실기기를 ADB로 연결해서 앱을 자동 조작하고 버그를 자동으로 발견하는 실험**

---

## 목적

Android 에뮬레이터/실기기를 ADB로 연결해서 Claude Code가 자동으로:
- 앱 실행
- 화면 이동
- 스크린샷 캡처
- UI dump 분석
- 버그 리포트 작성

까지 수행하는 자동화 테스트 실험.

---

## 실험 환경

- 기기: Samsung Galaxy (R3CX80BS9MX)
- 연결: USB ADB
- 앱: MovieFinder (com.choo.moviefinder)

---

## 실험 전 준비

**일반 터미널에서 ADB PATH 설정:**
```bash
echo 'export PATH="$HOME/Library/Android/sdk/platform-tools:$PATH"' >> ~/.zshrc && source ~/.zshrc
```

**기기 연결 확인:**
```bash
adb devices
# List of devices attached
# R3CX80BS9MX    device
```

---

## Claude Code에 요청한 프롬프트

```
ADB로 연결된 실기기에서 MovieFinder 앱을 자동으로 조작해줘.
1. 앱 실행
2. 홈 화면 스크린샷
3. 검색 화면으로 이동 후 스크린샷
4. 즐겨찾기 화면으로 이동 후 스크린샷
5. 각 화면에서 UI 이상한 부분 있으면 버그 리포트 작성해줘.
```

---

## Claude Code가 자동으로 한 것들

```
패키지명 확인 (adb shell pm list packages)
    ↓
앱 실행 (adb shell am start)
    ↓
스크린샷 캡처 (adb shell screencap)
    ↓
탭 좌표 파악 실패 → UI dump로 정확한 좌표 재확인
(adb shell uiautomator dump)
    ↓
기기 잠금 감지 → 자동 해제 후 재시도
    ↓
홈/검색/즐겨찾기 화면 순서대로 캡처
    ↓
UI dump 수치 + 스크린샷 분석 → 버그 리포트 자동 작성
```

---

## 발견된 버그 3개

### 🔴 BUG-01 — 홈 화면 최근 본 영화 RecyclerView 높이 21px

| 항목 | 내용 |
|------|------|
| 화면 | HomeFragment |
| 증상 | 최근 본 영화 섹션 RecyclerView가 21px에 불과 |
| UI Dump 증거 | `rv_watch_history bounds: [0,441][1080,462]` → 높이 21px |
| 영향 | 시청 기록이 있어도 영화 썸네일 표시 불가 |
| 추정 원인 | wrap_content 높이에서 데이터 없을 때 0으로 collapse |
| 수정 방향 | `android:minHeight` 설정 또는 데이터 없을 때 섹션 GONE 처리 |

### 🟡 BUG-02 — 검색 화면 공백 차이 중복 검색어 저장

| 항목 | 내용 |
|------|------|
| 화면 | SearchFragment |
| 증상 | "반지의 제왕"과 "반지의제왕" 별개로 저장됨 |
| 영향 | 검색 기록 오염, 사용자 혼란 |
| 수정 방향 | 저장 전 `query.trim()` 적용 |

```kotlin
// 현재 (추정)
repository.saveSearch(query)

// 수정안
repository.saveSearch(query.trim())
```

### 🟢 BUG-03 — 검색 화면 내용 없을 때 스크롤바 노출

| 항목 | 내용 |
|------|------|
| 화면 | SearchFragment |
| 증상 | 최근 검색어 2개만 있는 빈 화면에서 스크롤바 표시 |
| 수정 방향 | `android:scrollbars="none"` 또는 `android:fadeScrollbars="true"` |

---

## 화면별 테스트 결과

| 화면 | 상태 |
|------|------|
| 홈 | ⚠️ BUG-01 |
| 검색 | ⚠️ BUG-02, BUG-03 |
| 즐겨찾기 | ✅ 정상 |

---

## ADB 자동화의 핵심 명령어들

```bash
# 앱 실행
adb shell am start -n com.package/.MainActivity

# 스크린샷 캡처
adb shell screencap -p /sdcard/screen.png
adb pull /sdcard/screen.png /tmp/screen.png

# UI 계층 dump (정확한 좌표 파악)
adb shell uiautomator dump /sdcard/ui.xml
adb pull /sdcard/ui.xml /tmp/ui.xml

# 화면 탭
adb shell input tap X Y

# 기기 해상도 확인
adb shell wm size

# 기기 깨우기
adb shell input keyevent KEYCODE_WAKEUP
```

---

## 느낀 점

ADB + Claude Code 조합으로 실기기에서 자동으로 앱을 조작하고 버그를 발견할 수 있었다.

특히 BUG-01의 RecyclerView 21px 버그는 사람이 눈으로 보면 그냥 지나칠 수 있는데, UI dump 수치 분석으로 정확하게 잡아냈다.

**QA 테스터 역할을 AI가 대신할 수 있다는 게 핵심이다.** 매번 수동으로 앱을 클릭하면서 테스트하지 않아도, Claude Code에 화면 목록만 던져주면 자동으로 돌아다니면서 버그를 찾아준다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*테스트 기기: Samsung Galaxy (R3CX80BS9MX)*  
*작성일: 2026-05-11*
