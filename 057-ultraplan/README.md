# 057 - /ultraplan 실험 — 웹 Plan + 로컬 실행

**웹에서 심층 계획을 세우고 로컬에서 자율 실행 — Plan Mode보다 깊은 아키텍처 개선 계획 + 사이드 이슈 2개 추가 발견**

---

## 목적

048일차 Plan Mode의 심화 버전인 `/ultraplan` 실험.  
계획을 로컬이 아닌 **클라우드 웹 환경**에서 생성하고, 계획을 검토 후 로컬에서 실행하는 방식.

---

## ultraplan이란?

```
/ultraplan <프롬프트>
    ↓
레포지토리를 클라우드에 업로드
    ↓
웹에서 심층 계획 생성 (claude.ai/code)
    ↓
계획 검토 후 로컬에서 실행
```

Plan Mode와의 차이: 계획 생성 자체를 클라우드 환경에서 하기 때문에 더 많은 컨텍스트를 활용 가능.

---

## 설정 이슈

첫 실행 시 403 오류 발생 → `claude auth login`으로 재인증 필요.

---

## 생성된 계획 (4가지 개선 항목)

### 1. WHAT 주석 제거
메서드명으로 자명한 한국어 주석 제거, WHY 주석만 유지.

### 2. SettingsViewModel 아키텍처 개선
```
현재: ViewModel → Json.encodeToString() → Context.getString()
변경: UseCase가 Json 처리, Fragment가 getString(), ViewModel은 순수 로직만
```

### 3. 에러 처리 일관성
`startTmdbAuth()` 수동 try-catch → `launchWithErrorHandler` 통일.

### 4. 테스트 보일러플레이트 추출
8개 ViewModel 테스트에 반복된 `Dispatchers.setMain/resetMain` → `CoroutineTestBase` 추출.

---

## 실행 결과

**모든 테스트 통과 ✅ (638개 → 전체 통과)**

| 작업 | 결과 |
|------|------|
| WHAT 주석 제거 | SettingsViewModel, SettingsFragment 등 다수 파일 |
| SettingsViewModel 아키텍처 | Context/Json 의존성 제거, SyncResult sealed class 추가 |
| 에러 처리 통일 | `startTmdbAuth()` launchWithErrorHandler 적용 |
| 테스트 베이스 추출 | `CoroutineTestBase.kt` 신규 생성, 9개 파일 상속 |

---

## 사이드 이슈 2개 추가 발견

PostToolUse 훅(detekt-on-save.sh)이 계속 오류를 뱉었는데, 알고 보니 코드 문제가 아니라 **Detekt 버전 버그**였음:

**이슈 1 — detekt 2.0.0-alpha.4 버그**
```
dev.detekt:ktlint-repackage:2.0.0-alpha.4 → Maven Central에 미배포
→ detekt 2.0.0-alpha.3으로 다운그레이드로 해결
```

**이슈 2 — AGP 9.2.1 + Hilt 호환성 문제**
```
AGP 9.2.1이 kotlin-stdlib을 2.4.0으로 강제 승격
→ Hilt 2.59.2의 kotlin-metadata-jvm이 2.3.0까지만 지원
→ resolutionStrategy.force("kotlin-stdlib:2.3.21")으로 핀 고정
```

---

## Plan Mode vs ultraplan 비교

| 항목 | Plan Mode (048) | ultraplan (057) |
|------|:--------------:|:--------------:|
| 계획 생성 위치 | 로컬 | **클라우드 웹** |
| 계획 깊이 | 파일 수준 | **아키텍처 수준** |
| 사이드 이슈 발견 | 0건 | **2건** |
| 소요 시간 | 4분 41초 | 더 김 (웹 업로드 포함) |
| 계획 검토 | 터미널 텍스트 | **웹 UI에서 편집 가능** |

---

## ★ Insight (Claude Code 발언)

> "ViewModel에 Context 주입은 레이어 위반: ViewModel이 context.getString()을 호출하면 UI 문자열 결정권이 Presentation 레이어 안으로 이중 진입함. Sealed class로 이벤트를 올리고 Fragment에서 getString()하면 ViewModel은 순수 로직만 담당."

---

## 느낀 점

Plan Mode가 "실행 전 계획 확인"이라면, ultraplan은 "클라우드에서 더 깊이 생각한 계획"이다.

특히 SettingsViewModel 아키텍처 개선처럼 여러 파일에 걸친 리팩토링을 계획할 때 ultraplan이 더 체계적인 다이어그램과 작업 순서를 제시했다.

PostToolUse 훅이 오류를 뱉는 게 오히려 detekt 버전 버그를 발견하는 계기가 됐다 — **훅이 없었다면 그냥 넘어갔을 문제**.

**"계획의 깊이가 실행의 품질을 결정한다."**

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-06-16*
