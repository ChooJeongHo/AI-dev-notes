# 067 - 네트워크 보안 강화 자동화

**이중 피닝 전략 적용 — OS 레벨 + 앱 레벨로 Certificate Pinning 강화, Bearer 토큰 로그 마스킹**

---

## 목적

016일차 보안 취약점 분석에서 Certificate Pin Mismatch 이슈가 발견됐는데,  
이번엔 실제로 보안을 **강화**하는 단계.

---

## 현황 분석 결과

**이미 구현된 것들 ✅**
- cleartext 트래픽 차단 (network_security_config.xml)
- OkHttp 앱 레벨 Certificate Pinning
- debug-only 로깅
- API 키 interceptor 주입

**개선 포인트 3가지**
1. `network_security_config.xml` — OS 레벨 pin-set 없음 (OkHttp에만 의존)
2. `OkHttpDebugPlugin.kt` — Authorization Bearer 토큰이 debug 로그에 평문 노출
3. `TmdbV4OkHttpClient` — `addDebugLogging()` 누락

---

## 적용된 변경 사항

| 파일 | 변경 내용 |
|------|---------|
| `res/xml/network_security_config.xml` | OS 레벨 `<pin-set>` 추가 (TMDB 전 도메인), `debug-overrides overridePins="true"` |
| `src/debug/.../OkHttpDebugPlugin.kt` | `redactHeader("Authorization")` — Bearer 토큰 로그 마스킹 |
| `di/NetworkModule.kt` | TmdbV4OkHttpClient에 `addDebugLogging()` 추가 |
| `NETWORK_SECURITY_REPORT.md` | 전체 보안 현황 + 개선 사항 + 핀 갱신 절차 문서화 |

---

## 핵심 — 이중 피닝 전략

```
[앱 레벨] OkHttp CertificatePinner
    → 앱 프로세스 내 SPKI 해시 검증
    → 루팅+Xposed 훅으로 우회 가능

[OS 레벨] network_security_config.xml pin-set
    → Android 네트워크 스택 레벨 검증
    → 앱 코드 조작만으로는 우회 불가
```

두 레이어 모두 적용 → 루팅된 기기에서 OkHttp 우회해도 OS 레벨에서 차단.

---

## debug-overrides 설정의 중요성

```xml
<!-- 이게 없으면 디버그 빌드에서도 OS가 핀 검사 → Charles Proxy 전혀 동작 안 함 -->
<debug-overrides overridePins="true">
    <trust-anchors>
        <certificates src="user"/>
    </trust-anchors>
</debug-overrides>
```

---

## Bearer 토큰 마스킹

```kotlin
// 수정 전: Authorization: Bearer eyJhbGci... 로그에 평문 노출
.apply { level = HttpLoggingInterceptor.Level.HEADERS }

// 수정 후: Authorization: ██ 마스킹
.apply {
    level = HttpLoggingInterceptor.Level.HEADERS
    redactHeader("Authorization")
}
```

---

## 016일차 vs 067일차 비교

| 항목 | 016일차 (취약점 분석) | 067일차 (보안 강화) |
|------|:------------------:|:-----------------:|
| 목적 | 문제 발견 | **문제 해결** |
| 방법 | Exa MCP로 외부 검색 | **코드 직접 수정** |
| 결과 | 취약점 리포트 | **실제 보안 강화 적용** |
| OS 레벨 피닝 | ❌ 없음 | **✅ 추가** |
| 토큰 마스킹 | ❌ 노출 | **✅ 마스킹** |

---

## 느낀 점

보안은 "한 레이어만 막으면 된다"는 생각이 위험하다. OkHttp 피닝만 있었는데, 루팅된 기기에서 Xposed로 OkHttp를 후킹하면 우회할 수 있다.

OS 레벨 피닝을 추가하니 앱 코드를 아무리 조작해도 네트워크 스택 레벨에서 막힌다.

**"보안은 심층 방어(Defense in Depth)다" — 한 레이어가 뚫려도 다음 레이어가 막는다.**

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-06-30*
