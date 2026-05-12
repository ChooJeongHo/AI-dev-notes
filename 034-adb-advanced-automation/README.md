# 034 - ADB 심화 — 텍스트 입력 + 스크롤 + 프레임 성능 측정 자동화

**33일차 기본 탐색에서 한 단계 더 — 한글 텍스트 자동 입력, 스크롤, 프레임 드랍 측정**

---

## 목적

33일차 ADB 기본 실험(화면 이동 + 스크린샷)의 심화 버전.  
텍스트 자동 입력, 스크롤 자동화, gfxinfo로 프레임 성능 측정까지 확장.

---

## 실험 내용

```
1. 검색 탭으로 이동
2. 검색창에 "인터스텔라" 자동 입력
3. 검색 결과 스크린샷
4. 홈으로 돌아와서 스크롤 3회
5. 스크롤 후 스크린샷
6. 프레임 드랍 측정 (gfxinfo)
```

---

## 핵심 발견 — Samsung 기기 한글 ADB 입력 차단

```
adb shell input text '인터스텔라'
    ↓
❌ NullPointerException (Samsung이 non-ASCII 차단)
    ↓
Claude Code가 스스로 ADB Keyboard APK 설치 (18KB)
    ↓
am broadcast -a ADB_INPUT_TEXT --es msg '인터스텔라'
    ↓
✅ 한글 입력 성공
```

**Samsung 기기 ADB 한글 입력 해결법:**
```bash
# 1. ADB Keyboard 설치
gh release download v2.5-dev --repo senzhk/ADBKeyBoard --pattern "*.apk" -D /tmp/
adb install /tmp/keyboardservice-debug.apk

# 2. IME 활성화 및 전환
adb shell ime enable com.android.adbkeyboard/.AdbIME
adb shell ime set com.android.adbkeyboard/.AdbIME

# 3. 한글 입력
adb shell am broadcast -a ADB_INPUT_TEXT --es msg '인터스텔라'

# 4. 텍스트 삭제
adb shell am broadcast -a ADB_CLEAR_TEXT

# 5. 완료 후 원복
adb shell ime set com.samsung.android.honeyboard/.service.HoneyBoardService
adb uninstall com.android.adbkeyboard
```

---

## 프레임 성능 측정 결과 (스크롤 3회)

```bash
adb shell dumpsys gfxinfo com.choo.moviefinder reset
# 스크롤 실행
adb shell dumpsys gfxinfo com.choo.moviefinder
```

| 지표 | 수치 | 평가 |
|------|------|------|
| Total frames | 537 | - |
| Janky frames | 5 (0.93%) | ✅ 우수 |
| 50th percentile | 14ms | ✅ 정상 |
| 90th percentile | 16ms | ✅ 60fps 경계 |
| 99th percentile | 19ms | ✅ 허용 범위 |
| Missed Vsync | 0 | ✅ 완벽 |
| GPU 99th percentile | 9ms | ✅ GPU 여유 충분 |

**결론: MovieFinder 스크롤 성능 정상. Janky 0.93%는 체감 버벅임 없음.**

---

## 발견된 이슈

### ⚠️ 이슈 1 — 스크롤 후 이미지 플레이스홀더 미표시

스크롤 후 빠르게 캡처하면 일부 카드가 Shimmer 대신 단순 회색 박스로 표시됨.

- 원인: 스크롤 속도가 빨라 Coil이 이미지 디코딩 전에 캡처됨
- 확인 필요: `MovieGridViewHolder`에서 `placeholder()` 설정 여부

### ⚠️ 이슈 2 — Samsung 기기 ADB 한글 입력 차단

- Samsung `InputShellCommand.runText()`가 non-ASCII 차단
- CI 자동화 시 Galaxy 기기 대상이라면 ADB Keyboard 미리 설치 필요

---

## ADB 자동화 시 주의사항

**Samsung 기기 특이사항:**
- `adb shell input text` 한글 미지원 → ADB Keyboard 우회 필요
- 화면 타임아웃으로 자동화 중단 가능 → 먼저 타임아웃 늘리기

```bash
# 화면 타임아웃 10분으로 설정
adb shell settings put system screen_off_timeout 600000

# 완료 후 원복
adb shell settings put system screen_off_timeout 30000
```

**성능 측정 정확도:**
- ADB 브로드캐스트, 화면 잠금 등 자동화 스크립트 자체 지연이 포함됨
- `High input latency` 수치는 자동화 환경 영향으로 실제 사용자 체감과 다름

---

## 33일차 vs 34일차 비교

| 구분 | 33일차 | 34일차 |
|------|--------|--------|
| 주요 기능 | 앱 탐색 + 스크린샷 | 텍스트 입력 + 스크롤 + 성능 측정 |
| 발견 버그 | 3개 (UI 레이아웃) | 2개 (플레이스홀더, Samsung 한글) |
| 성능 분석 | 없음 | gfxinfo 프레임 측정 포함 |
| 특이사항 | UI dump 좌표 파악 | Samsung 한글 입력 차단 해결 |

---

## 느낀 점

Samsung 기기에서 한글 입력이 차단되는 걸 Claude Code가 스스로 파악하고 ADB Keyboard APK를 찾아서 설치하고 우회하는 과정이 인상적이었다.

사람이라면 "Samsung 한글 입력 ADB 안 됨"을 구글링하고 해결책을 찾는 데 시간이 걸렸을 텐데, Claude Code가 오류 메시지를 보고 스스로 원인 파악 → APK 다운로드 → 설치 → 우회 입력까지 자율적으로 처리했다.

**gfxinfo로 프레임 성능까지 측정할 수 있다는 것도 새로운 발견이었다.** 단순히 화면을 찍는 것에서 성능 수치까지 자동으로 수집하는 테스트 자동화가 가능하다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*테스트 기기: Samsung Galaxy (R3CX80BS9MX)*  
*작성일: 2026-05-12*
