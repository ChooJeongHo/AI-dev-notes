# 036 - Claude Code로 기술 문서 자동 생성

**프로젝트 전체 코드베이스를 분석해서 아키텍처/API/데이터 흐름까지 포함한 기술문서를 자동으로 생성**

---

## 목적

매번 수동으로 작성하던 기술문서를 Claude Code가 코드베이스를 직접 분석해서 자동 생성.  
README 수준이 아닌 아키텍처 설계 근거, 데이터 흐름, ADR까지 포함한 실제 기술문서 작성.

---

## 실험 내용

```
1. Claude Code에게 프로젝트 전체 구조 분석 요청
2. oh-my-claudecode explore 에이전트로 도메인/데이터/프레젠테이션 레이어 병렬 분석
3. Writer 에이전트가 TECHNICAL_DOCS.md 자동 작성
4. MovieFinder, MediScan 두 프로젝트 모두 적용
```

---

## 사용 프롬프트

```
프로젝트 전체 구조 분석해서 기술문서 자동 생성해줘.
대상: 아키텍처, 주요 모듈, API 연동, 데이터 흐름까지 포함한 기술문서
파일명: TECHNICAL_DOCS.md
```

---

## 결과 비교

| 항목 | MovieFinder | MediScan |
|------|------------|---------|
| 분석 시간 | 2분 23초 | 3분 58초 |
| 생성 분량 | 1,187줄 / 46KB | 754줄 |
| 섹션 수 | 17개 | 13개 |
| 토큰 사용 | 109.8k | 102.0k |

---

## MovieFinder TECHNICAL_DOCS.md 섹션 구성

| 섹션 | 내용 |
|------|------|
| 프로젝트 개요 | 핵심 기능 6개 + 구현 방식 |
| 아키텍처 | Clean Architecture 레이어 다이어그램, MVVM 단방향 흐름 |
| 빌드 환경 | SDK 버전표, AGP 9.x 주의사항, 핵심 라이브러리 버전표 |
| Domain 레이어 | 모델 24개, Repository 인터페이스 13개, UseCase 69개 전수 목록 |
| Data 레이어 | Retrofit 구성, API 엔드포인트 전체 표, Paging 4종 |
| Presentation 레이어 | 화면 10개, 어댑터 12개, 커스텀 Canvas View 5개 |
| 데이터 흐름 | 홈/검색/즐겨찾기/상세/통계 — ASCII 다이어그램 |
| API 연동 | 인증 방식, 캐시 전략, 재시도 전략, 에러 처리 |
| Room DB | 테이블 11개 스키마, 마이그레이션 v12→v21 히스토리 |
| Hilt DI | 모듈 4개, EntryPoint 패턴 |
| 보안 | Certificate Pinning, EncryptedSharedPreferences |
| 성능 최적화 | Coil 캐시, RecyclerView 설정, DB 인덱스 전략 |
| 테스트 | 유닛 509개 / UI 23개, 커버리지 기준 |
| CI/CD | 워크플로우 4개, Git Hooks, Dependabot |
| ADR | 설계 결정 6개 (ISP, Abstract DAO, Delegate 등) |

---

## MediScan TECHNICAL_DOCS.md 섹션 구성

| 섹션 | 내용 |
|------|------|
| 기술 스택 | 버전 포함 전체 스택 |
| 아키텍처 | 레이어 구조 + 핵심 규칙 |
| DI 설정 | 4개 Module 요약 |
| API 연동 | 3개 서비스 엔드포인트 + 3-round fallback 전략 |
| 데이터 흐름 | OCR / 검색 / 즐겨찾기 경로별 흐름 |
| Room DB | 7개 Entity + v17까지 Migration 이력 |
| OCR 파이프라인 | 4-priority 패턴 상세 |
| 알람 시스템 | 복약 시간 파싱 규칙 + 구성 요소 연결 구조 |
| UseCase 목록 | 21개 전부 |

---

## 핵심 발견

### Claude Code가 자동으로 파악한 것들

- **ADR(Architecture Decision Record)** — 설계 결정 근거를 코드에서 역추적해서 자동 문서화
- **마이그레이션 히스토리** — Room DB v12→v21 전체 이력 자동 추출
- **데이터 흐름 다이어그램** — ASCII 아트로 레이어 간 흐름 자동 생성

### ★ Insight (Claude Code 발언)

> "기술문서는 '무엇이 있나'보다 '왜 이렇게 설계했나'가 핵심입니다. 클래스 목록만 나열하는 문서는 코드에서 이미 읽을 수 있으므로, 각 레이어의 설계 근거와 트레이드오프, 데이터 흐름 다이어그램을 중심으로 작성합니다."

---

## 느낀 점

기술문서는 항상 "나중에 써야지" 하다가 미뤄지는 작업이었다.  
Claude Code가 코드베이스를 직접 읽고 설계 근거까지 역추적해서 문서화해준다는 게 인상적이었다.

특히 ADR 섹션은 사람이 직접 작성하려면 설계 당시 기억을 떠올려야 하는데, Claude Code는 코드 패턴에서 역으로 추론해서 뽑아냈다.

**모두의 창업 등 외부 제출용 기술문서를 별도 수고 없이 항상 최신 상태로 유지할 수 있다.**

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder), [ChooJeongHo/MediScan](https://github.com/ChooJeongHo/MediScan)*  
*작성일: 2026-05-14*
