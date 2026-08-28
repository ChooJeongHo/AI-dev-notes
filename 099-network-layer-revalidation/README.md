# 099 - Network Layer 재검증 — 이번엔 오탐이 아니라 "처음부터 제대로 만들어져 있었다"

**098일차와 정반대 결과: 095~097일차 KMRB 네트워크 클라이언트는 검증 스킬이 걸릴 만한 오탐도, 진짜 이슈도 없었다 — callTimeout은 코드를 재사용해서 자동으로 상속됐고, 나머지 차이는 전부 문서화된 의도적 설계였다**

---

## 목적

089일차 이후 처음으로 network-layer-validator를 재실행해서, 095~097일차에 새로 추가된 KMRB 네트워크 클라이언트(NetworkModule.kt, KmrbApiService.kt, KmrbRatingXmlParser.kt, KoreanRatingRepositoryImpl.kt, GetKoreanRatingUseCase.kt, FilterMoviesByKoreanRatingUseCase.kt)를 검증한다.

---

## 결과: 실질 이슈 0건

### 1. callTimeout — "기억해서 챙긴 것"이 아니라 "구조적으로 상속됨"

NetworkModule.kt의 provideKmrbOkHttpClient()는 공유 확장함수 applyCommonConfig()를 호출하는데, 여기에 callTimeout(25.seconds)가 이미 들어있다.

git 이력으로 시점을 대조해보니:
- callTimeout(25.seconds)는 089일차 커밋(50de1f6, 2026-08-06)에서 이 공유 함수에 추가됨
- KMRB 클라이언트는 095일차 커밋(0069339, 2026-08-17)에서 11일 뒤에 만들어지면서, 이미 callTimeout이 들어있는 applyCommonConfig()를 그대로 호출

> 누가 "KMRB에도 callTimeout 잊지 말고 넣어야지" 하고 수동으로 챙긴 게 아니라, 공유 설정 함수를 재사용한 결과 자동으로 상속됐다 — 가장 튼튼한 형태의 일관성이다.

### 2. KMRB의 에러 응답 형식 — 081일차 KOFIC 패턴을 정확히 인지하고 따름

KMRB도 KOFIC과 동일하게 HTTP 200 OK + 본문에 에러 코드를 담아 보내는 방식이다.

- resultCode?.startsWith("30") 또는 resultMsg에 "SERVICE_KEY" 포함 → DomainException.Unauthorized
- 인식 못하는 코드 → DomainException.Unknown으로 던져서, "일시적 서버 오류"를 "등급 없음"으로 영구 캐싱하는 걸 방지 (코드 주석에 명시)

081일차 KOFIC(errorCode?.startsWith("32") → Unauthorized)도 같은 구조다. 두 정부 Open API 모두 "HTTP 상태 코드가 아니라 200 OK + 바디 내 에러 코드"라는 동일한 특성을 갖고, KMRB 구현이 이걸 정확히 인지하고 같은 방식으로 처리하고 있었다.

### 3. 오프라인/재시도/캐시 일관성 — 전략은 다르지만 전부 의도적 차이

| 항목 | TMDB(메인) | KOFIC/KMRB | 판단 |
|---|---|---|---|
| SSL 재시도 | 적용 | 미적용 | 정보성 — KOFIC/KMRB는 애초에 인증서 피닝을 안 함(CLAUDE.md 문서화) |
| 429 Retry-After | 적용 | 미적용 | 정보성 — KOFIC/KMRB는 rate limit도 바디 내 에러 코드로 알림, 429 자체가 발생 안 함 |
| OkHttp 디스크 캐시 | 적용 | 미적용 | 정보성 — 대신 Room 레벨 캐시(등급 확정=영구, 등급없음=24시간 TTL) 사용, 범용 HTTP 캐시보다 도메인에 더 맞음 |
| 실패 시 그레이스풀 디그레이드 | loadOptionalNullable 패턴 | suspendRunCatching으로 흡수 → null 반환 + 경고 로그 | 동일 패턴 |

suspendRunCatching이 CancellationException을 정확히 재전파하는 것도 확인했다.

### 4. 유일하게 남은 사소한 차이 (수정 안 함)

KOFIC의 인식 못한 에러 코드는 DomainException.ServerError(0, cause), KMRB는 DomainException.Unknown으로 서로 다른 타입을 쓴다. 하지만 KMRB 쪽 예외는 GetKoreanRatingUseCase에서 무조건 흡수돼 UI에 절대 노출되지 않으므로("실패를 흡수하는 얇은 래퍼"라고 주석에 명시), 실제 동작 차이는 없다. 수정하지 않았다.

---

## 098일차와의 대조 — 왜 이번엔 스킬 가드가 필요 없었나

| | 098일차 (Material3) | 099일차 (Network Layer) |
|---|---|---|
| 검증 결과 | 오탐 2건 발견 (4번째 반복) | 오탐 자체가 거의 없음 |
| 원인 | 스킬의 체크리스트가 프로젝트 전제(M2 기반)를 모름 | KMRB 구현 자체가 처음부터 기존 관례를 정확히 따라 만들어짐 |
| 조치 | SKILL.md에 "0단계 기준선 확인" 가드 추가 | 가드 불필요 판단 |

> 098일차는 "위반처럼 보이는 게 실제론 의도적 설계"였던 오탐이 반복돼서 스킬 자체를 고쳤지만, 099일차는 검증 항목 4가지 전부 처음부터 명확한 설계 근거가 코드 주석·CLAUDE.md·git 이력에 남아있어서 "위반처럼 보인다" 단계까지도 거의 가지 않았다.

---

## 097일차와의 연결 — "구조가 좋으면 다음 확장이 저절로 안전해진다"

097일차에서 096일차의 범용 캐시가 박스오피스 섹션에 조정 없이 재사용됐던 것과, 오늘 callTimeout이 KMRB 클라이언트에 자동으로 상속된 것은 같은 종류의 발견이다.

> 좋은 구조적 설계(공유 함수, 범용 캐시)는 다음에 추가되는 코드가 "실수로 빠뜨리는 것" 자체를 원천적으로 방지한다. 사람이 매번 "이것도 챙겨야지"라고 기억할 필요 없이, 재사용하기만 하면 자동으로 올바르게 동작한다.

---

## 오늘 배운 것 — 한 줄 정리

1. 설정값이 공유 함수를 통해 상속되면, 새로 추가되는 클라이언트가 그 설정을 빠뜨릴 위험 자체가 사라진다 — "기억해서 챙기기"보다 훨씬 안전한 방식이다.
2. 서로 다른 API가 같은 특이 구조(200 OK + 바디 에러코드)를 갖는다면, 나중에 만든 클라이언트가 그 패턴을 인지하고 똑같이 처리했는지가 검증 포인트다.
3. 클라이언트마다 재시도/캐시 전략이 달라도, 각 차이에 문서화된 이유가 있다면 전부 정보성이지 결함이 아니다.
4. 검증 재실행 결과가 "클린"으로 나올 때도 이유는 다를 수 있다 — "검증 스킬이 오탐을 잘 걸러냈다"(098)와 "애초에 코드가 잘 만들어져서 걸릴 게 없었다"(099)는 다른 성격의 클린이다.
5. 스킬에 가드를 추가하는 건 "재검증할 때마다 하는 일"이 아니라, 실제로 반복되는 오탐이 확인됐을 때만 하는 조치다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*
*작성일: 2026-08-28*
