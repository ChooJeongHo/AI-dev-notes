# 069. 의존성 버전 자동 진단 — "업데이트 있음"에서 "이건 안전, 이건 위험"으로

## 배경

Dependabot은 새 버전이 나오면 PR을 띄워줄 뿐, 그 업데이트가 안전한지 위험한지는 판단해주지 않는다.
결국 사람이 매번 changelog를 읽고 breaking change 여부를 확인해야 한다.

이번 실험의 목표는 그 판단 자체를 Claude Code에게 맡기는 것.
"자동 업데이트 안전" vs "리뷰 필요"로 분류까지 하게 시켰다.

## 실험 방법

**프롬프트**
```
libs.versions.toml에 있는 모든 라이브러리의 현재 버전과 최신 버전을 비교해줘.
각 항목마다:
1. 버전 차이가 major/minor/patch 중 무엇인지
2. 공식 changelog나 release note 기준 breaking change 여부
3. "자동 업데이트 안전" vs "리뷰 필요" 로 분류
표로 정리해줘.
```

**실행 방식**
- 34개 버전 항목을 7개 카테고리로 나눠 **병렬 서브에이전트**로 동시 조사
  (Toolchain / AndroidX Core-UI / 아키텍처 컴포넌트 / DI·네트워크 / Compose BOM / 이미지·서드파티 / 테스트)
- 각 에이전트가 실시간 웹 조회로 최신 버전 + 공식 release note 확인
- 총 소요 시간: 약 6분 (에이전트 중 가장 오래 걸린 건 Compose BOM, 3분 43초)

> 병렬로 안 돌렸다면 34개 항목을 순차 조회하면서 훨씬 오래 걸렸을 것. 조사 성격의 작업을 여러 서브에이전트로 쪼개는 게 이번 실험의 숨은 핵심.

## 결과 요약

**34개 버전 항목 중 30개는 이미 최신.** 실제로 손댈 만한 건 4개뿐이었다.

### 🔴 리뷰 필요 (자동 업데이트 금지)

| 라이브러리 | 현재 | 최신 | 차이 | 왜 위험한가 |
|---|---|---|---|---|
| Kotlin | 2.3.21 | 2.4.0 | minor | K1 컴파일러 완전 제거. Hilt 2.60이 2.4.0을 공식 지원하는지 미확정 |
| Detekt | 2.0.0-alpha.3 | 2.0.0-alpha.5 | patch(alpha) | alpha.4에서 API 변경(Glob→Regex) + Kotlin 2.4.0 기준 빌드로 전환 |
| androidx.benchmark | 1.5.0-alpha06 | 1.5.0-alpha07 | patch(alpha) | `requireAot` 기본값 true로 변경 → CI 영향 가능 |
| security-crypto | 1.1.0 | 1.1.0(동일) | - | 버전은 그대론데 API 전체가 deprecated 상태 |
| shimmer | 0.5.0 | 0.5.0(동일) | - | 저장소 자체가 2023년에 archived됨 |

### 🟢 자동 업데이트 안전

| 항목 | 내용 |
|---|---|
| Compose BOM | 2026.06.00 → 2026.06.01 (patch, breaking change 없음) |
| 나머지 29개 | 전부 이미 최신 버전 사용 중 |

## 핵심 발견 — 버전 숫자만 봐서는 안 되는 것들

1. **버전이 같아도 위험할 수 있다.** `security-crypto`와 `shimmer`는 버전 변경이 없는데도 "리뷰 필요"로 분류됨.
   - `security-crypto`: API가 deprecated → 장기적으로 Keystore 직접 구현 필요
   - `shimmer`: 저장소 자체가 archived → 더는 릴리즈 자체가 불가능한 상태
   - → 단순 버전 diff가 아니라 **생태계 건강도**까지 봐야 한다는 걸 확인

2. **패키지 묶음 업그레이드 시 매트릭스가 꼬일 수 있다.**
   - Kotlin 2.4.0 + Detekt 2.0.0-alpha.5는 같이 가야 궁합이 맞음
   - 하지만 Hilt 2.60이 Kotlin 2.4.0을 지원한다는 공식 근거가 아직 없음
   - → "낱개로 최신화"가 아니라 "조합으로 검증"해야 하는 경우가 실제로 존재

3. **Dependabot이 제안해도 자동 병합하면 안 되는 케이스**
   - LeakCanary 3.0-alpha가 존재하지만 stable(2.14) 유지가 맞음
   - Dependabot은 이런 맥락을 모르고 그냥 최신 버전을 제안함

## 권장 액션 (Claude Code가 제시)

1. **바로 반영 가능**: Compose BOM patch 업데이트
2. **개별 검증 후 반영**: Detekt alpha.5 (Kotlin은 그대로 두고 Detekt 자체 회귀만 우선 확인), benchmark alpha07 (baseline profile 스크립트 로컬 재검증 필요)
3. **묶어서 검토**: Kotlin 2.4.0은 Detekt 업그레이드와 함께, 단 Hilt 컴파일 여부를 별도 브랜치에서 먼저 검증
4. **백로그**: security-crypto → Keystore 직접 구현 마이그레이션, shimmer → Compose 기반 효과로 교체 (마침 068에서 SearchFragment를 Compose로 마이그레이션했으니 자연스러운 후속 작업)

## 이번 실험이 보여준 것

Dependabot이 할 수 있는 일: "새 버전 나왔어요, PR 보세요"
Claude Code가 추가로 한 일:
- semver 차이 분석
- 공식 release note에서 breaking change 유무 판단
- 버전이 같아도 위험한 케이스(archived, deprecated) 별도 탐지
- 업그레이드 간 의존 관계(Kotlin↔Detekt↔Hilt) 매트릭스 파악
- "자동 반영" / "검증 후 반영" / "묶어서 검토" / "백로그"로 실행 우선순위까지 분류

**다음에 이어서 하면 좋을 것**
- 이 판단 로직을 GitHub Actions 스케줄 워크플로우로 얹어서 주기적으로 자동 실행 (010일차 패턴 재사용)
- "자동 업데이트 안전" 항목은 실제로 컴파일·테스트까지 돌려서 자동 커밋하는 단계까지 확장
- Kotlin 2.4.0 + Detekt 2.0.0-alpha.5 조합을 별도 브랜치에서 실제로 검증
