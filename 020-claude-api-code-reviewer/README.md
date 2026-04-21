# AI 활용 기록 020 - Claude API로 커스텀 코드 리뷰어 만들기

> Android가 아닌 환경에서 Claude API를 직접 호출하는 도구를 만들어본 실험

## 목적

지금까지 Android(MovieFinder) 위주로 실험했는데, Claude API 활용 능력이 플랫폼에 종속되지 않는다는 걸 확인하고 싶었다. 코드를 붙여넣으면 Claude가 리뷰해주는 로컬 HTML 도구를 직접 만들어보는 실험.

---

## 만든 것

**code-reviewer.html** — 브라우저에서 바로 열리는 AI 코드 리뷰어

기능:
- 언어 선택 (Kotlin, Java, Python, JavaScript 등)
- 리뷰 관점 선택 (전반적 품질 / 성능 / 보안 / 가독성 / 테스트)
- Claude API 호출 → 등급(A/B/C/D), 이슈 목록, 잘된 점 출력

---

## 코드 리뷰 결과 (NetworkModule.kt)

Claude Code로 리뷰한 결과 (API 크레딧 소진으로 웹앱 대신 Claude Code 활용):

**등급: B+**

주요 발견 이슈:
- Cache-Control 인터셉터가 검색/리뷰 등 모든 엔드포인트에 동일하게 5분 캐시 적용 → URL 경로별 TTL 분리 필요
- 이미지 OkHttpClient에 HTTP 캐시 없음 (Coil 자체 캐시로 수용 가능하나 주석 명시 필요)
- DebugEventListener 인스턴스 두 곳에서 중복 생성
- API 키가 쿼리 파라미터 노출 → Bearer Token 방식으로 전환 고려

잘된 점: Certificate Pinning, Timeout 통일, Singleton 범위 보장, 관심사 분리, 디버그 로깅 분리 등

---

## 실험 과정에서 배운 것

### 1. 아티팩트(iframe)에서 API 호출이 막히는 이유

처음에 Claude.ai 아티팩트 위젯으로 만들었더니 `Failed to fetch` 오류가 발생했다.

이유: 아티팩트는 **보안 샌드박스(iframe)** 안에서 실행되기 때문에 외부 API로의 직접 HTTP 요청이 브라우저 수준에서 차단된다.

```
아티팩트(iframe) → api.anthropic.com ❌ CORS 차단
로컬 HTML 파일   → api.anthropic.com ✅ 정상 동작
```

### 2. 로컬 HTML vs 서버 방식

| 방식 | 장점 | 단점 |
|------|------|------|
| 로컬 HTML | 설치 없이 파일 하나로 동작 | API 키 필요 |
| Python 스크립트 | 터미널에서 바로 실행 | Python 설치 필요 |
| 로컬 서버 | 팀원 공유 가능 | 셋업 복잡 |

개인 개발 도구 용도라면 **로컬 HTML이 가장 심플하다.**

### 3. API 키 관리 + 크레딧 구조

- Anthropic API 키는 생성 시 딱 한 번만 표시 → 반드시 저장
- 노출됐거나 잃어버리면 삭제 후 재발급 (무제한)
- **Claude.ai 구독 ≠ Anthropic API 크레딧** → 완전히 별개 과금
- API로 서비스 만들 때 비용 모니터링 필수

---

## Claude API 호출 구조

```javascript
const response = await fetch('https://api.anthropic.com/v1/messages', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-api-key': apiKey,
    'anthropic-version': '2023-06-01'
  },
  body: JSON.stringify({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 1000,
    system: '시스템 프롬프트...',
    messages: [{ role: 'user', content: '코드 리뷰 요청...' }]
  })
});
```

이 구조는 Android든 Python이든 어떤 환경에서도 동일하게 적용된다.

---

## 느낀 점

Claude Code CLI, GitHub MCP, oh-my-claudecode 같은 기존 도구들을 쓰는 것과 달리, Claude API를 직접 호출해서 도구를 만드는 건 완전히 다른 역량이다.

도구를 **사용하는 것**에서 도구를 **만드는 것**으로 한 단계 올라간 느낌이었다.

그리고 지금까지 Android 프로젝트로만 실험했는데, Claude API 호출 방식은 플랫폼에 무관하다는 걸 직접 확인했다. HTML, Python, Kotlin 어디서든 같은 방식으로 Claude를 활용할 수 있다.

---

*작성일: 2026-04-21*
