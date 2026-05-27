# 044 - Claude Code로 신규 개발자 온보딩 문서 자동 생성

**실제 코드 패턴을 분석해서 "왜 이렇게 짰는가"까지 설명하는 온보딩 문서 자동 생성**

---

## 목적

신규 개발자가 합류했을 때 구두로 설명하거나 직접 써야 했던 온보딩 문서를 자동화.  
README나 주석이 아닌 **실제 코드 패턴과 Git Hooks까지 분석**해서 정확한 문서 생성.

---

## 생성된 ONBOARDING.md 구성

| 섹션 | 핵심 내용 |
|------|---------|
| 개발 환경 셋업 | JDK 21 필수, local.properties 두 키, Git Hooks 활성화 명령어 |
| 프로젝트 구조 | Clean Architecture 레이어 규칙, 디렉토리 맵, 핵심 파일 빠른 참조표 |
| 첫 PR 흐름 | 브랜치 → 커밋(pre-commit) → 푸시(pre-push) → CI → PR 체크리스트 |
| 자주 하는 실수 | 8가지 (키 누락/Hooks 미설정/레이어 위반/CancellationException 등) |
| 핵심 패턴 | 9가지 (repeatOnLifecycle, Mutex.tryLock(), WhileSubscribed5s 등) |

**총 563줄 — 모든 코드 예제는 실제 코드베이스 패턴 그대로 반영**

---

## 핵심 발견

### "자주 하는 실수" 섹션이 가장 가치 있다

코드를 아무리 읽어도 알 수 없는 것들이 자동으로 포함됐다:

```
❌ git config core.hooksPath .githooks 실행 안 함
→ pre-commit, pre-push 훅이 동작하지 않아 Detekt/테스트 없이 push 가능

❌ AGP 9에서 kotlinOptions 블록 사용
→ 컴파일 오류 발생 (AGP 9부터 compilerOptions 블록으로 변경)

❌ CancellationException을 catch (e: Exception)으로 삼킴
→ 코루틴 취소가 무시되어 메모리 누수 및 비정상 동작
```

### "왜 이 방식인가"를 함께 설명

단순 사용법이 아니라 대안과 이유까지 자동으로 포함됐다:

| 패턴 | 대안 | 이유 |
|------|------|------|
| `Mutex.tryLock()` | `if (isLoading) return` | 원자적 보장 — 체크-후-세트 경쟁 조건 방지 |
| `Channel.CONFLATED` | `StateFlow` | 중간값 드롭 필요한 단방향 이벤트에 적합 |
| `WhileSubscribed5s` 공유 인스턴스 | 매번 새 인스턴스 | magic number 제거 + 인스턴스 재사용 |

### pre-commit vs pre-push 설계 의도 자동 파악

```
pre-commit: Detekt + 컴파일만 → 커밋 속도 유지
pre-push: 전체 테스트 실행 → 안전성 보장
```

신규 개발자가 "왜 push가 느린지" 당황하지 않도록 이유까지 문서화됐다.

---

## 36일차 기술문서 vs 44일차 온보딩 문서 비교

| 항목 | 036 기술문서 | 044 온보딩 문서 |
|------|------------|---------------|
| 대상 독자 | 프로젝트 이해가 필요한 개발자 | 처음 합류하는 신규 개발자 |
| 분석 소스 | 전체 코드베이스 | 코드 패턴 + Git Hooks + 빌드 설정 |
| 분량 | 1,187줄 | 563줄 |
| 핵심 가치 | "무엇이 있고 왜 이렇게 설계했나" | "어떻게 시작하고 어떤 실수를 피하나" |

---

## 느낀 점

온보딩 문서는 항상 "나중에 써야지" 하다가 미뤄지는 작업이었다.  
Claude Code가 Git Hooks 설정, AGP 9 특이사항, 코루틴 패턴까지 실제 코드에서 역추적해서 문서화해줬다.

특히 **"자주 하는 실수" 섹션**은 팀 내에서 수개월 경험이 쌓여야 알 수 있는 내용인데, Claude Code가 코드 패턴을 분석해서 자동으로 뽑아낸 게 인상적이었다.

**새 팀원이 들어올 때마다 이 명령어 하나로 온보딩 문서를 최신 상태로 재생성할 수 있다.**

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-05-24*
