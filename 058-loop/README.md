# 058 - /loop 명령어 실험 — 5분마다 자율 반복 코드 개선

**5분 간격으로 코드 리뷰 → 개선 → 컴파일 검증을 3회 자율 반복 — 총 14개 파일 자동 수정**

---

## 목적

`/loop` 명령어로 반복 작업을 자동화.  
5분마다 코드 개선점을 찾아서 수정하는 사이클을 자율로 반복하는 실험.

---

## /loop란?

```
/loop <간격> <작업>
    ↓
cron 스케줄러 등록
    ↓
지정된 간격마다 자동 실행
    ↓
세션 종료 시 자동 삭제 (CronDelete로 수동 종료도 가능)
```

---

## 실험 설정

```
/loop 5m MovieFinder 코드에서 개선할 점 찾아서 수정해줘
```

**주기**: 5분마다 자동 실행  
**총 회차**: 3회차  
**수정 파일**: 14개

---

## 회차별 수정 내용

### 1회차 — Settings 영역

| 파일 | 수정 내용 |
|------|---------|
| SettingsViewModel.kt | `pendingRequestToken` → `SavedStateHandle` 기반으로 변경 (OAuth 중 프로세스 종료 후 토큰 복원) |
| SettingsViewModel.kt | `disconnectTmdb()` 성공 시 `_disconnectSuccess` 이벤트 추가 |
| SettingsFragment.kt | TMDB 인증 URL `startActivity()` → `ActivityNotFoundException` 가드 추가 |
| SettingsFragment.kt | export 후 파일 피커 실행을 `RESUMED` 상태일 때만 호출하도록 생명주기 가드 |
| SettingsFragment.kt | `disconnectSuccess` 구독 → "연결 해제됨" Snackbar 표시 |
| strings.xml | `tmdb_disconnected`, `tmdb_no_browser` 문자열 추가 |

### 2회차 — Search / Home 영역

| 파일 | 수정 내용 |
|------|---------|
| SearchViewModel.kt | PERSON→MOVIE 모드 전환 시 `_isPersonSearchLoading` true 잔존 → `false` 추가 |
| HomeFragment.kt | `observeSelectedTab()`이 `tab.ordinal`로 탭 탐색 → `tab.tag` 기반으로 교체 |
| SearchFragment.kt | 오프라인 검색 후 재연결 시 오프라인 결과 잔존 → `networkMonitor` 구독 추가 |

### 3회차 — Detail / Backup 영역

| 파일 | 수정 내용 |
|------|---------|
| DetailViewModel.kt | `ratingMutex` 추가 → `submitTmdbRating()` 인플라이트 중 재제출 방지 |
| ImportUserDataUseCase.kt | 복원 항목 수(Int) 반환하도록 변경 |
| SettingsViewModel.kt | `_importSuccess` → `Channel<Int>`, 복원 건수 전달 |
| SettingsFragment.kt | 복원 건수 0일 때 "복원할 데이터가 없습니다" Snackbar 표시 |
| strings.xml | `import_empty` 문자열 추가 |

---

## PostToolUse 훅의 역할

053일차에 만든 `detekt-on-save.sh` 훅이 loop와 시너지를 냈다:

```
loop가 파일 수정
    ↓
PostToolUse 훅 자동 실행
    ↓
컴파일 오류 즉시 감지 → Claude Code에 알림
    ↓
loop가 바로 오류 수정
```

**훅이 없었다면** 컴파일 오류를 나중에야 발견했을 것.

---

## /loop vs 다른 자동화 비교

| 항목 | /autoresearch (051) | /ultrawork (055) | /loop (058) |
|------|:------------------:|:---------------:|:-----------:|
| 목표 | 수치 달성까지 루프 | 병렬 동시 실행 | 시간 간격 반복 |
| 종료 조건 | 목표 달성 | 작업 완료 | 수동 종료 |
| 적합한 상황 | 커버리지 향상 | 독립 병렬 작업 | 지속적 개선 |

---

## 느낀 점

/loop는 "CI처럼 주기적으로 코드 품질을 점검하는 AI"다.

5분마다 다른 영역(Settings → Search/Home → Detail/Backup)을 자율로 탐색하면서 매 회차마다 새로운 버그를 찾아냈다.

특히 PostToolUse 훅과의 시너지가 인상적이었다 — 훅이 실시간으로 컴파일 오류를 잡아주니 loop가 오류를 즉시 수정하고 다음 작업으로 넘어갔다.

**"loop + 훅 = AI가 스스로 품질을 유지하는 자율 파이프라인"**

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-06-17*
