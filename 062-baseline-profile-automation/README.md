# 062 - Baseline Profile 자동화 — 한 줄 명령으로 프로필 갱신

**307줄 자동화 스크립트로 에뮬레이터 시작→프로필 생성→결과 검증→Git 안내까지 4분 3초 만에 완료**

---

## 목적

056일차 성능 프로파일링에서 "Baseline Profile 1.8MB가 Cold Start 527ms에 기여한다"고 확인.  
수동으로 생성하던 Baseline Profile을 **한 줄 명령으로 자동화**.

---

## Baseline Profile이란?

앱이 자주 실행하는 코드 경로를 AOT(Ahead-of-Time) 컴파일해서 Cold Start를 빠르게 만드는 Android 최적화 기법.  
코드가 변경되면 Profile을 갱신해야 효과가 유지된다.

---

## 자동화 전 vs 후

| 항목 | 자동화 전 | 자동화 후 |
|------|:--------:|:--------:|
| 에뮬레이터 시작 | 수동 | **자동** |
| API 레벨 확인 | 수동 | **자동 (28 미만 차단)** |
| adb root 권한 | 수동 | **자동** |
| 화면 활성화 | 수동 | **자동** |
| 프로필 생성 | `./gradlew` 명령 | **자동** |
| 결과 비교 | 수동 | **자동 (이전/이후 줄 수)** |
| 에뮬레이터 종료 | 수동 | **자동** |
| Git 커밋 안내 | 수동 | **자동 출력** |

---

## 사용법

```bash
# 에뮬레이터 자동 시작 후 생성
./scripts/generate_baseline_profile.sh --emulator

# 연결된 실기기 사용
./scripts/generate_baseline_profile.sh

# 빠른 재생성 (clean 생략)
./scripts/generate_baseline_profile.sh --emulator --skip-clean --keep-emulator
```

---

## 실행 결과

**소요 시간: 244초 (4분 3초)**

| 단계 | 결과 |
|------|------|
| 에뮬레이터 시작 (Pixel_10) | ✅ |
| adb root 권한 상승 | ✅ |
| 화면 활성화 | ✅ |
| generateReleaseBaselineProfile | ✅ |
| 프로필 변경 | +16,694줄 / -7,764줄 |

---

## 핵심 발견 — 에뮬레이터 오류 자동 해결

처음 실행 시 오류 발생:
```
Cannot locate tasks that match ':baselineprofile:generateBaselineProfile'
```
→ task 이름이 `:app:generateReleaseBaselineProfile`이 맞았음 (자동 수정)

두 번째 오류: "never flushed profiles"
```
google_apis 에뮬레이터 → user 빌드 → /data/misc/profiles/ 접근 권한 없음
    ↓
adb root로 권한 상승 → 해결 ✅
```

세 번째 개선: 화면 꺼짐으로 profile flush 실패 방지
```bash
adb shell input keyevent KEYCODE_WAKEUP
adb shell svc power stayon usb
```

**오류가 날 때마다 원인 분석 → 스크립트에 자동으로 반영** — 최종 스크립트는 이 모든 케이스를 처리한다.

---

## 스크립트 구조 (307줄)

```
1. 사전 확인 — local.properties, adb, gradlew 존재 여부
2. 에뮬레이터 시작 + sys.boot_completed 폴링 (최대 180초)
3. adb root 권한 상승
4. 화면 활성화 (screen-off 시 flush 실패 방지)
5. API 레벨 확인 (28 미만 차단)
6. 현재 프로필 통계 기록
7. Gradle clean (선택)
8. generateReleaseBaselineProfile 실행
9. 결과 검증 + 이전/이후 비교
10. Git 상태 확인 + 커밋 명령어 안내
11. 에뮬레이터 종료
```

---

## 056일차 → 062일차 성능 실험 시리즈 완성

| 실험 | 내용 | 핵심 발견 |
|------|------|---------|
| 056 | 성능 프로파일링 자동화 | 에뮬레이터 최대 11배 과장, Baseline Profile 효과 확인 |
| **062** | **Baseline Profile 자동화** | **한 줄 명령으로 프로필 갱신, 4분 3초** |

---

## 느낀 점

코드가 많이 바뀌면 Baseline Profile도 갱신해야 하는데, 기존엔 귀찮아서 미루게 된다.

`./scripts/generate_baseline_profile.sh --emulator` 한 줄이면 에뮬레이터부터 커밋 안내까지 자동으로 처리되니까 갱신 장벽이 사라진다.

**"귀찮아서 안 하던 최적화를 AI가 자동화해줬다" — 좋은 습관을 만드는 데 자동화가 필요하다.**

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-06-23*
