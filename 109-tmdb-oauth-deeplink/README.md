# 109 - TMDB OAuth 플로우 — 최초 가설은 빙산의 일각, 실제론 3중 결함이었다

**"onCreate에 콜백 처리가 없다"는 최초 가설로 시작했는데, 실기기 검증을 진행하며 완전히 다른 성격의 결함 3개(OAuth 프로토콜 오해 2건 + Hilt ViewModel 스코프 버그 1건)가 순차적으로 드러났다 — 특히 3번째는 코드 리뷰만으로는 절대 못 잡을 버그였다**

---

## 목적

MovieFinder의 TMDB 계정 연동(OAuth) 플로우를, 101~106일차에서 반복된 "딥링크는 실기기에서만 검증된다"는 원칙으로 점검한다.

---

## 최초 가설과 실제 원인이 완전히 달랐다

**최초 가설:** `onCreate`에 OAuth 콜백 처리 로직이 없다 — 101/103/104/106일차와 비슷한 딥링크 처리 누락일 거라 예상했다.

**실제로 드러난 것:** 이건 빙산의 일각이었고, 실기기 검증 과정에서 **완전히 다른 성격의 결함 3개가 순차적으로** 드러났다.

---

## 발견된 결함 3가지

### 결함 1 — redirect_to 위치 오류

승인 URL의 **쿼리 파라미터**로 `redirect_to`를 붙였는데, **TMDB v4 API는 이걸 무시**했다.

**수정:** `request_token` 발급 시 **POST body**로 옮겼다.

### 결함 2 — TMDB v4 콜백에는 애초에 쿼리 파라미터가 없다

코드가 콜백 URL에 `request_token`이 echo되어 돌아올 거라고 기대하고 있었는데, **이건 잘못된 가정**이었다 — TMDB v4 콜백은 쿼리 파라미터를 아예 안 준다.

**수정:** 앱이 이미 보관 중인 `pendingRequestToken`을 콜백에서 그대로 사용하도록 변경.

### 결함 3 (가장 치명적) — MainActivity와 SettingsFragment가 서로 다른 ViewModel 인스턴스를 쓰고 있었다

`MainActivity`와 `SettingsFragment`가 **Activity 스코프 vs Fragment 스코프가 서로 달라서**, 같은 이름의 `SettingsViewModel`을 참조하고 있었지만 실제로는 **완전히 다른 인스턴스**였다. 그 결과 `pendingRequestToken`이 두 컴포넌트 사이에서 **절대 공유되지 않았다.**

**수정:** `SettingsFragment`를 `activityViewModels()`로 전환해서, `MainActivity`와 동일한 ViewModel 인스턴스를 공유하도록 했다.

> **왜 이게 제일 치명적인가:** 결함 1, 2는 OAuth 프로토콜에 대한 이해 부족이었지만, 결함 3은 **Hilt/Fragment의 ViewModel 스코프 규칙을 잘못 사용한, 코드만 읽어서는 절대 발견할 수 없는 버그**였다. 실기기 로그를 직접 추적하지 않았다면 찾지 못했을 것이다.

### 부수 수정

`onCreate`에 콜백 처리 로직도 추가했다 — 최초 가설이었던 "singleTask 프로세스 킬 대응"에 해당하는 부분.

---

## 실기기 검증 (SM-S926N, 실제 TMDB 계정)

세 가지 결함을 순서대로 고쳐가며, **`access_token`/`session convert` API가 200 OK로 성공**하고 설정 화면에 **"TMDB 계정이 연결되어 있습니다"**가 뜨는 것까지 확인했다.

**추가로 "액티비티 유지 안 함" 개발자 옵션**으로 프로세스 킬 시나리오를 재현했을 때도 **정상 복원·정상 연동**을 확인했다.

---

## 테스트

- 유닛 테스트 860개 전부 통과 (신규/갱신 포함)
- 신규 계측 테스트 `MainActivityOAuthCallbackTest` 추가
- 기존 `MainActivityDeepLinkBackStackTest`/`SettingsFragmentTest`/`MainActivityTest` 총 20개 **회귀 없음** 확인

### 테스트 커버리지에 관한 정직한 판단

`TmdbAuthRepository.kt`는 순수 인터페이스(메서드 시그니처 선언만, 구현/로직 없음)라 **이 파일 자체를 직접 테스트할 대상이 없다.** 실제로 이 프로젝트의 `domain/repository/` 아래 인터페이스 20개 중 **어느 것도 전용 테스트 파일이 없다** — `app/src/test/.../domain/repository/` 디렉터리 자체가 존재하지 않는다.

바뀐 `getRequestToken(redirectTo: String)` 시그니처의 실제 동작은 구현체 `TmdbAuthRepositoryImpl`에 있고, 이건 이미 `TmdbAuthRepositoryImplTest.kt`에 반영했다:

- `getRequestToken returns token from api` (시그니처 변경 반영)
- **`getRequestToken sends redirect_to in request body`** (신규 — 오늘 수정의 핵심인 "redirect_to를 body로 보내는지"를 직접 검증)
- `getRequestToken wraps api error in DomainException` (시그니처 변경 반영)

> 인터페이스 파일에 대응 테스트가 없는 건 오늘 세션에서 생긴 커버리지 공백이 아니라 **프로젝트 전체의 기존 패턴**이므로, 추가 테스트 없이 종료했다.

---

## 정리 — 101~106일차와는 다른 성격의 발견

이번 발견은 101~106일차의 "popBackStack 오용" 계열과 **완전히 다른 성격**이다 — **OAuth 프로토콜 오해(2건) + Hilt ViewModel 스코프 버그(1건)**였다.

| | 101~106일차 | 109일차 |
|---|---|---|
| 문제 성격 | Navigation API 오용, Intent-filter 불일치, 커버리지 누락 | OAuth 프로토콜 이해 오류 + ViewModel 스코프 불일치 |
| 발견 난이도 | 실기기 UI 흐름에서 발견 | **실기기 로그를 직접 추적해야만 발견 가능** (특히 결함 3) |

재사용 가능한 adb 실기기 테스트 기법(SystemUI 터치 가로채기, force-stop vs 프로세스 킬 차이, logcat 버퍼 회전)도 메모리에 기록해뒀다.

---

## 오늘 배운 것 — 한 줄 정리

1. **최초 가설이 맞았다고 확신하지 말고, 실기기 검증 과정에서 나오는 새로운 증거를 계속 따라가야 한다** — 오늘은 원래 가설이 빙산의 일각에 불과했다.
2. OAuth 같은 외부 프로토콜 연동은 **"쿼리로 보내면 되겠지"** 같은 가정을 실제 API 스펙으로 검증해야 한다.
3. **같은 이름의 ViewModel이라도 스코프(Activity vs Fragment)가 다르면 완전히 다른 인스턴스**다 — 이건 코드를 아무리 읽어도 안 보이고, 실제로 상태 공유가 실패하는 걸 실기기에서 확인해야 드러난다.
4. 인터페이스 파일 자체에 테스트가 없는 게 이상해 보여도, **프로젝트 전체의 기존 패턴이라면 그게 이 프로젝트의 정상 상태**다 — 오늘 새로 생긴 공백만 정확히 구분해서 처리하면 된다.
5. 재사용 가능한 디버깅 기법(개발자 옵션 활용, adb 로그 추적 방법)은 **결과뿐 아니라 과정 자체를 기록해두면** 다음에 비슷한 문제를 훨씬 빨리 풀 수 있다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*
*작성일: 2026-09-16*
