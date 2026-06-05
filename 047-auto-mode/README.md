# 047 - Claude Code Auto Mode 실험

**매번 승인 없이 Claude가 자율적으로 코드 수정 → 컴파일 → 오류 수정 → 커밋까지 완료**

---

## 목적

Claude Code Auto Mode가 실제 개발 작업에서 얼마나 자율적으로 동작하는지 확인.  
"코드 품질 개선점 3가지 찾아서 수정하고 커밋해줘" 한 마디로 전체 플로우 완주.

---

## Auto Mode란?

기존 Claude Code는 파일 쓰기, bash 명령 실행마다 매번 사용자 승인이 필요했다.  
Auto Mode는 AI 분류기가 각 툴 호출을 사전 검토해서 안전한 작업은 자동 실행, 위험한 작업은 차단한다.

| 모드 | 동작 방식 |
|------|---------|
| Default | 모든 작업 매번 승인 필요 |
| Plan | 계획만 세우고 실행은 승인 후 |
| **Auto** | AI 분류기가 자동 승인/차단 |

---

## 활성화 방법

Claude Code 세션에서 **Shift + Tab** 눌러서 모드 전환.

---

## 실험 결과

**요청:**
```
MovieFinder 앱의 코드 품질 개선점 3가지를 찾아서 직접 수정하고 커밋까지 해줘
```

**Claude가 자율로 한 것 (승인 0회):**

1. TECH_DEBT_REPORT.md 읽고 개선 후보 선정
2. 관련 파일 병렬 읽기
3. 코드 수정 4개 파일
4. `./gradlew compileDebugKotlin` 실행 → 컴파일 오류 발견 → 자동 수정
5. `git add` → `git commit` 실행
6. pre-commit Detekt 오류 발견 → 자동 수정 → 재커밋

**총 소요 시간: 6분 29초**

---

## 자동 수정된 내용

| 파일 | 개선 내용 |
|------|---------|
| CollectionViewModel.kt | `launchWithErrorHandler` + `asStateFlow()` 적용, import 3개 제거 |
| SearchViewModel.kt | `suspendRunCatching` 적용, `!!` 2건 null-safe 처리 |
| NetworkModule.kt | `applyCommonConfig()` 추출 — timeout+DebugEventListener 중복 8줄 제거 |
| FavoriteViewModel.kt | `Eagerly` → `WhileSubscribed5s` 교체 (045일차 기술 부채 리포트 즉시 수정 권장 항목) |

---

## 핵심 발견

### Detekt 오류 자율 해결

```
커밋 시도
    ↓
pre-commit Detekt 오류 발생 (TooManyFunctions)
    ↓
@Suppress 추가 + 코드 분리로 자동 수정
    ↓
재커밋 성공
```

사람이 했다면 오류 메시지 읽고 → 원인 파악 → 수정 → 재시도 과정이 필요했는데, Claude가 스스로 처리했다.

### ★ Insight (Claude Code 발언)

> "Detekt의 TooManyFunctions 오류가 applyCommonConfig() 추가로 발생한 것은 역설적입니다 — 중복을 줄이기 위해 함수를 추출했더니 함수 개수가 늘어 경고가 생겼습니다. 이런 경우 @Suppress는 규칙의 한계를 인정하는 정당한 사용입니다."

---

## Default Mode vs Auto Mode 비교

| 항목 | Default Mode | Auto Mode |
|------|-------------|----------|
| 파일 수정 | 매번 승인 | 자동 |
| bash 실행 | 매번 승인 | 자동 |
| git commit | 매번 승인 | 자동 |
| 사용자 개입 | 수십 번 | 0번 |
| 소요 시간 | 더 김 | 6분 29초 |

---

## 주의사항

- 위험한 작업(대량 파일 삭제, 외부 API 호출 등)은 자동 차단됨
- 분류기가 같은 작업을 3회 연속 차단하면 Auto Mode 자동 해제
- 격리된 환경(로컬 개발)에서 사용 권장

---

## 느낀 점

"코드 품질 개선하고 커밋해줘" 한 마디로 6분 만에 4개 파일 수정 + 컴파일 검증 + Detekt 통과 + 커밋까지 완료됐다.

특히 045일차 기술 부채 리포트에서 "즉시 수정 권장" 으로 표시한 `Eagerly → WhileSubscribed5s` 항목을 Auto Mode가 스스로 찾아서 수정한 게 인상적이었다.

**발견 → 분류 → 등록 → 수치화 → 자동 수정까지 완전한 AI 코드 품질 관리 사이클이 완성됐다.**

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-06-02*
