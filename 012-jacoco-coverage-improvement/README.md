# AI 활용 기록 012 - Claude Code로 테스트 커버리지 분석 및 개선

> "테스트가 많아도 커버리지는 별개다" — JaCoCo 분석부터 자동 테스트 추가까지

## 목적

Claude Code가 JaCoCo 커버리지 리포트를 분석하고, 커버리지가 낮은 부분을 스스로 파악하여 테스트를 자동으로 추가하는 실험  
기존 161개 테스트가 있어도 전체 커버리지가 낮은 이유를 파악하고 개선

## 사용한 도구

- Claude Code CLI (터미널)
- JaCoCo (Gradle 태스크)
- 대상 프로젝트: [MovieFinder](https://github.com/ChooJeongHo/MovieFinder)

---

## 핵심 발견 — 테스트 수와 커버리지는 다르다

실험 전 MovieFinder 상태:
- 유닛 테스트: **161개**
- Line 커버리지: **26.3%**

테스트가 161개나 있는데 커버리지가 26%밖에 안 됐다.

**이유:**
1. Fragment, Adapter, ViewHolder 등 UI 코드는 유닛 테스트가 불가능 (Espresso 필요)
2. 이 UI 코드들이 전체 코드의 상당 부분을 차지해서 분모가 커짐
3. ViewModel/UseCase/Repository에 집중된 테스트는 실제로 잘 작성됐지만, 전체 비율로 보면 낮게 나옴

---

## 실험 과정

### 1단계 - 현재 커버리지 측정

```bash
./gradlew jacocoTestReport
```

**초기 커버리지 (전체 코드 기준):**

| 지표 | 수치 |
|------|------|
| Line | 26.3% (1,294 / 4,924) |
| Branch | 13.2% (163 / 1,236) |
| Instruction | 27.2% (8,632 / 31,729) |
| Method | 32.5% |
| Class | 34.4% |

### 2단계 - Claude Code에 개선 요청

```
JaCoCo 커버리지 리포트를 분석해서 커버리지가 낮은 클래스들을 찾아줘.
ViewModel, UseCase, Repository 중에서 테스트가 없거나 부족한 것들 위주로
테스트를 추가해줘. UI 코드(Fragment, Adapter)는 제외하고.
목표는 Line 커버리지 40% 이상으로 올리는 거야.
```

### 3단계 - Claude Code가 수행한 작업

**테스트 132개 신규 추가:**
- UseCase 테스트: 91개
- TagRepository, CircuitBreaker, ExponentialBackoff 등: 41개

**JaCoCo exclude 목록 정비 (업계 표준 관행):**

유닛 테스트로 측정이 의미 없는 코드를 분모에서 제외:
```kotlin
// UI — Espresso로만 테스트 가능
"**/presentation/**/*Fragment*.*",
"**/presentation/**/*Adapter*.*",
"**/presentation/**/*ViewHolder*.*",
"**/presentation/widget/**",
"**/presentation/common/**",
// Application / Activity
"**/MovieFinderApp*.*",
"**/MainActivity*.*",
// Debug-only utilities
"**/core/startup/**",
"**/core/notification/*Worker*.*"
```

---

## 결과

### 커버리지 전후 비교

| 지표 | 이전 | 이후 | 변화 |
|------|------|------|------|
| Line | 26.3% | **74.8%** | +48.5%p |
| Branch | 13.2% | **50.3%** | +37.1%p |
| Instruction | 27.2% | **71.7%** | +44.5%p |
| Method | 32.5% | **69.3%** | +36.8%p |
| Class | 34.4% | **80.6%** | +46.2%p |

목표 Line 40% → 실제 **74.8%** 달성 (목표 대비 약 2배)

### 테스트 수 변화

| 이전 | 이후 | 추가 |
|------|------|------|
| 161개 | **293개** | +132개 |

---

## 배운 것

**커버리지를 올리는 두 가지 방법:**

1. **테스트 추가** → UseCase, Repository 등 미테스트 영역 커버
2. **exclude 설정** → 유닛 테스트가 불가능한 UI 코드를 분모에서 제외

두 가지를 함께 해야 의미 있는 커버리지 수치가 나온다.

**Claude Code의 역할:**
- JaCoCo XML 리포트를 직접 파싱해서 커버리지 낮은 클래스 파악
- 기존 테스트 패턴(MockK, Turbine, StandardTestDispatcher)을 자동으로 따라서 테스트 작성
- 업계 표준에 맞는 exclude 설정 자동 적용

---

## 느낀 점

테스트를 많이 만들었다고 커버리지가 높은 건 아니라는 걸 직접 확인했다.

Claude Code가 리포트를 분석해서 테스트가 필요한 곳을 스스로 찾아내고, 기존 코드 패턴에 맞게 테스트를 추가하는 과정이 인상적이었다. 특히 JaCoCo exclude 설정을 업계 표준 관행에 맞게 자동으로 정비한 부분이 유용했다.

단순히 테스트 수를 늘리는 게 아니라 **의미 있는 커버리지**를 만드는 것이 중요하다는 걸 알게 됐다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-04-03*
