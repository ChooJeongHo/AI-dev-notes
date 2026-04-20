# AI 활용 기록 019 - Claude Code AFK 자율 실행 모드 실험

> 이슈 목록만 던져주고 Claude Code가 스스로 판단하고 수정하고 PR까지 올리는 실험

## 목적

사람이 개입 없이 Claude Code가 얼마나 자율적으로 작업을 처리할 수 있는지 실험  
18일차에 발견한 성능 이슈 목록을 전달하고 AFK(Away From Keyboard) 상태로 내버려둠

---

## 실험 프롬프트

```
ultra think

아래 성능 이슈 목록을 우선순위에 따라 스스로 처리해줘.
각 이슈 수정 후 테스트 통과 확인하고,
완료된 것들은 GitHub MCP로 이슈 등록 후 PR까지 자동으로 올려줘.
나는 개입하지 않을 거야. 알아서 처리해줘.
```

---

## Claude Code가 자율적으로 한 것들

```
이슈 목록 수신
    ↓
현재 코드 상태 분석
    ↓
이미 구현된 것 스스로 판단해서 스킵 ← 핵심
    ↓
새로운 것만 수정
    ↓
GitHub MCP로 이슈 #41~45 생성
    ↓
테스트 통과 확인 (465개)
    ↓
이슈 클로즈 + main 푸시까지 완료
```

---

## 결과

### 처리된 이슈 (5개)

| 이슈 | 내용 | 수정 파일 |
|------|------|---------|
| #41 | SettingsFragment 메인 스레드 블로킹 → withContext(IO) | SettingsFragment.kt |
| #42 | OkHttp Cache-Control 무효 → max-age=300 | NetworkModule.kt |
| #43 | ExponentialBackoff 미사용 → MovieRemoteMediator 적용 | MovieRemoteMediator.kt |
| #44 | Stats strftime() 인덱스 무용 → yearMonth 컬럼 + DB v17 | WatchHistoryEntity 외 3개 |
| #45 | SearchFragment shimmer stopShimmer() 순서 오류 | SearchFragment.kt |

### 스스로 스킵한 이슈 (5개)

이미 구현되어 있어서 Claude Code가 자체 판단으로 제외:
- flatMapLatest, composite index, combine, debounce, itemAnimator=null

### 테스트 수 변화

| 이전 | 이후 | 추가 |
|------|------|------|
| 389개 | **465개** | +76개 |

---

## 핵심 발견 — "스스로 판단해서 스킵"

단순히 시키는 대로 하는 게 아니라 **현재 코드 상태를 먼저 파악하고 불필요한 작업을 스스로 제외**했다.

이게 중요한 이유:
- 사람이 모든 걸 검증하고 지시하지 않아도 됨
- 중복 작업으로 인한 코드 충돌 방지
- 실제 필요한 것만 처리해서 효율적

---

## AFK 모드가 유용한 상황

- 이슈 백로그가 쌓여있을 때
- 반복적인 코드 개선 작업
- 테스트 추가 및 커버리지 향상

반면 **사람의 판단이 필요한 것들** (비즈니스 로직 변경, 아키텍처 결정)은 AFK 모드가 적합하지 않다.

---

## 느낀 점

이슈 목록만 던져주고 자리를 비웠는데, 돌아오니 이슈 생성 → 코드 수정 → 테스트 통과 → 이슈 클로즈 → 푸시까지 전부 완료되어 있었다.

특히 이미 구현된 것을 스스로 파악해서 스킵하는 부분이 인상적이었다. 단순한 자동화가 아니라 **현재 상태를 이해하고 판단하는 자율성**이 있었다.

**AI 인재로서 핵심 역량:** 무엇을 시킬지 명확하게 정의하는 것. 이슈 목록을 잘 작성할수록 Claude가 더 정확하게 처리한다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*생성된 이슈: #41 ~ #45*  
*작성일: 2026-04-18*
