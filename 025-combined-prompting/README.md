# 025 - System Prompt + Few-shot + CoT 조합 실험

**프롬프트 엔지니어링 3기법을 동시에 적용하면 얼마나 강력한가**

---

## 목적

22일차(System Prompt), 23일차(Few-shot), 24일차(CoT)를 각각 실험했으니 세 가지를 동시에 조합하면 결과가 얼마나 달라지는지 확인.  
실험 대상은 MovieFinder 대신 새로운 프로젝트 **MediScan** (약 봉투/처방전 OCR 스캔 Android 앱)을 사용.

---

## 실험 조건

**동일한 요청:** `MediScan의 코드에서 개선할 수 있는 부분을 찾아줘.`

| 단계 | 방식 |
|------|------|
| 일반 요청 | 아무 기법 없이 바로 요청 |
| 조합 적용 | System Prompt + Few-shot + CoT 동시 적용 |

---

## 조합 프롬프트 구조

```
[System Prompt - 역할 + 제약 조건]
당신은 10년 경력의 시니어 Android 개발자이자 Clean Architecture 전문가입니다.
1. Clean Architecture 관점에서 레이어 위반 위주로 분석
2. 각 이슈에 우선순위(P0/P1/P2)와 수정 비용 표시
3. 개선 코드 예시 반드시 포함
4. 테스트 가능성 관점도 포함

[Few-shot - 출력 형식 고정]
이슈: DrugRepositoryImpl God class
우선순위: P0 / 비용: 중간
원인: 4개 인터페이스를 1개 클래스가 구현
개선: DrugSearchRepositoryImpl + LocalRepositoryImpl 2개로 분리

[Chain of Thought - 단계별 사고 강제]
답하기 전에 다음 순서로 생각해줘:
1단계: 레이어별 의존성 방향 파악
2단계: 각 레이어 위반 사례 수집
3단계: 우선순위 판단
4단계: 최종 개선 방향 제시
```

---

## 결과 비교

| 구분 | 일반 요청 | 조합 적용 |
|------|---------|---------|
| 발견 이슈 수 | 5개 | **10개** |
| 레이어 위반 분석 | 없음 | **의존성 방향 다이어그램** |
| 개선 코드 예시 | 없음 | **전 이슈 Before/After 코드** |
| 테스트 가능성 | 없음 | **각 이슈마다 포함** |
| 우선순위 분류 | P0/P1/P2 | **P0/P1/P2 + 수정 비용까지** |
| 분석 깊이 | 표면적 | **파일명 + 라인 수까지 명시** |

---

## 조합에서만 발견된 이슈들

일반 요청에서 못 찾은 것들:

**P0**
- `DrugShapeSearchViewModel.kt:8` — Presentation이 Data 레이어 직접 import (CsvImportWorker)

**P1**
- `DrugInfo.kt:3,6` — Domain 모델이 @Serializable에 직접 의존 (프레임워크 결합)
- `AnalyzeInteractionsUseCase.kt` — 에러 처리 정책 없음 (인프라 예외가 ViewModel까지 노출)
- `DrugSearchRepositoryImpl.kt:204-207` — Round 5 fallback에 타임아웃 없음

**P2**
- `ResultViewModel.kt:272-296` — 비즈니스 로직(성분 분석, 카테고리 수집)이 ViewModel에 직접 구현
- 에러 sealed class 6곳 중복 정의
- DrugInfo Anemic Model (27개 nullable 필드, 행위 없음)

---

## 프롬프트 엔지니어링 3부작 최종 정리

| 기법 | 단독 효과 | 조합 시 역할 |
|------|---------|------------|
| System Prompt | 분석 깊이 향상 | 역할과 제약 조건 설정 |
| Few-shot | 형식 통제 | 출력 형식 고정 |
| CoT | 근거 있는 분석 | 단계별 사고 강제 |
| **세 가지 조합** | **발견 이슈 2배, Before/After 코드, 테스트 관점까지** | |

---

## 언제 조합을 쓰면 좋을까

**단독으로 충분한 경우:**
- 빠른 질문, 간단한 작업 → 아무것도 안 써도 됨
- 형식만 맞추면 되는 경우 → Few-shot만

**조합이 필요한 경우:**
- 코드 리뷰, 아키텍처 분석 같은 복잡한 작업
- 팀 내 AI 활용 표준화가 필요할 때
- 일관된 품질의 결과물을 반복 생성할 때

---

## 느낀 점

세 가지를 각각 실험했을 때보다 조합했을 때 결과가 훨씬 강력했다. 발견 이슈가 2배로 늘었고, Before/After 코드와 테스트 관점까지 자동으로 나왔다.

프롬프트 엔지니어링은 도구를 외우는 게 아니라 **"AI 답변이 마음에 안 들 때 어떤 기법을 꺼내 쓸지 판단하는 능력"**이다. 세 가지 기법을 알고 있으면 상황에 맞게 하나씩 또는 조합해서 쓸 수 있다.

---

*실험 대상: MediScan (private)*  
*작성일: 2026-04-28*
