# Claude Code Skills — 완전 분석 가이드

> Claude.ai의 Skills 시스템이 무엇인지, 왜 유용한지, 어떻게 만드는지에 대한 완전 분석  
> 실제 `/mnt/skills/public/` 내부를 직접 탐색하여 작성

---

## 📌 목차

1. [스킬이란?](#1-스킬이란)
2. [다른 AI의 스킬과의 차이](#2-다른-ai의-스킬과의-차이)
3. [Anthropic 공식 제공 스킬 목록](#3-anthropic-공식-제공-스킬-목록)
4. [스킬의 장점](#4-스킬의-장점)
5. [스킬 내부 구조 해부](#5-스킬-내부-구조-해부)
6. [스킬 만드는 법 — 단계별 가이드](#6-스킬-만드는-법--단계별-가이드)
7. [직접 만든 예시 스킬](#7-직접-만든-예시-스킬)
8. [공식 스킬 활용 예시 (pptx 스킬)](#8-공식-스킬-활용-예시-pptx-스킬)

---

## 1. 스킬이란?

Claude Code Skills는 **Claude가 특정 작업을 수행할 때 먼저 읽는 마크다운 지침서 파일**이다.

```
일반 Claude:
  "PPT 만들어줘" → 그때그때 다른 방식으로 생성

Skills가 있는 Claude:
  "PPT 만들어줘"
    → pptx/SKILL.md 파일을 view 도구로 읽음
    → 정해진 색상 팔레트, 레이아웃 가이드, QA 체크리스트에 따라 작업
    → 일관된 전문가 수준의 결과물 생성
```

**핵심 원리:**
- 스킬 = `SKILL.md` 마크다운 파일
- Claude가 사용자 요청에서 특정 키워드를 감지하면 해당 파일을 자동으로 읽음
- 파일 안의 지침을 그 자리에서 습득하여 작업 수행
- 코드 실행이 아닌 **"지식 주입"** 방식

---

## 2. 다른 AI의 스킬과의 차이

| 구분 | 다른 AI의 스킬 (Copilot, Alexa 등) | Claude Skills |
|------|--------------------------------------|----------------|
| **형태** | API 함수 / 플러그인 / 외부 서비스 | 마크다운 텍스트 파일 |
| **동작 방식** | 코드를 실행하거나 외부 서버 호출 | Claude가 파일을 읽고 지침을 따름 |
| **만드는 방법** | 코딩 필수 (API 명세 작성) | 마크다운 작성만으로 가능 |
| **저장 위치** | 외부 서버 / 클라우드 | `/mnt/skills/` 로컬 파일 시스템 |
| **확장성** | 개발자만 만들 수 있음 | 누구든 텍스트 파일만 만들면 됨 |
| **AI 역할** | 스킬을 호출하는 주체 | 스킬을 읽고 스스로 실행하는 주체 |

> **비유:** 다른 AI의 스킬이 "자동화 버튼"이라면, Claude Skills는 "전문가 매뉴얼"이다.

---

## 3. Anthropic 공식 제공 스킬 목록

실제 `/mnt/skills/` 디렉토리 구조:

```
/mnt/skills/
├── public/                        ← Anthropic 공식 스킬 (항상 사용 가능)
│   ├── pptx/SKILL.md              → PowerPoint 생성/편집
│   ├── docx/SKILL.md              → Word 문서 생성/편집
│   ├── xlsx/SKILL.md              → Excel 스프레드시트 생성/편집
│   ├── pdf/SKILL.md               → PDF 생성, 병합, OCR
│   ├── frontend-design/SKILL.md   → 고품질 UI/UX 컴포넌트
│   └── product-self-knowledge/    → Anthropic 제품 정보 정확성
│
├── examples/                      ← 참고/예시 스킬
│   ├── skill-creator/             → 새 스킬 설계·테스트 메타스킬
│   ├── mcp-builder/               → MCP 서버 구축 가이드
│   ├── slack-gif-creator/         → Slack GIF 생성
│   ├── algorithmic-art/           → 알고리즘 기반 예술 생성
│   ├── canvas-design/             → Canvas API 디자인
│   ├── brand-guidelines/          → Anthropic 브랜드 스타일
│   └── web-artifacts-builder/     → 웹 아티팩트 생성
│
└── user/                          ← 사용자 커스텀 스킬 추가 위치
```

### 공식 스킬 상세

| 스킬 | 트리거 키워드 | 핵심 기능 |
|------|-------------|---------|
| **pptx** | "PPT", "슬라이드", "발표자료", "deck" | 디자인 팔레트, 레이아웃, QA 포함 |
| **docx** | "Word", "워드", ".docx", "보고서" | XML 언팩 방식, 트랙 변경 지원 |
| **xlsx** | "Excel", "엑셀", "스프레드시트" | 수식, 차트, 데이터 정제 |
| **pdf** | "PDF", "폼 작성", "병합" | 생성/병합/OCR/암호화 |
| **frontend-design** | "UI", "컴포넌트", "웹 디자인" | 고품질 프로덕션 수준 UI |
| **product-self-knowledge** | "Claude 기능", "요금", "API" | 제품 정보 정확성 보장 |

---

## 4. 스킬의 장점

### ① 일관성 보장

스킬이 없으면 Claude는 매번 다른 방식으로 작업한다.  
스킬이 있으면 항상 동일한 표준 절차를 따른다.

```
# pptx 스킬이 강제하는 표준
1. 색상 팔레트 선택 (주색 60-70%, 강조색 1-2개)
2. 슬라이드별 시각 요소 필수 포함 (이미지/차트/아이콘)
3. QA: markitdown으로 텍스트 검증 → 이미지로 변환 → 시각 검증
4. 발견된 문제 수정 → 재검증 반복
```

### ② 전문 지식의 캡슐화

도메인 전문가의 노하우를 파일에 담아두면,  
Claude가 매번 새로 학습하지 않아도 전문가 수준으로 작업한다.

```
pptx/SKILL.md 안에 담긴 전문 지식 예시:
- "절대 타이틀 아래 장식선 쓰지 말 것 (AI 생성물처럼 보임)"
- "Dark/Light 대비: 제목·결론 슬라이드는 다크, 내용 슬라이드는 라이트"
- "텍스트만 있는 슬라이드 금지 — 항상 시각 요소 포함"
```

### ③ 3단계 점진적 로딩 (메모리 효율)

```
레벨 1: 메타데이터 (name + description)
        → 항상 메모리에 있음 (~100 단어)
        → 트리거 여부 판단에 사용

레벨 2: SKILL.md 본문
        → 스킬이 트리거될 때만 로딩 (500줄 권장)
        → 핵심 절차와 규칙

레벨 3: 번들 리소스 (scripts/, references/, assets/)
        → 필요할 때만 로딩 (무제한)
        → 대용량 참조 문서, 실행 스크립트
```

불필요한 스킬이 메모리를 차지하지 않으면서도  
필요할 때 깊은 전문성을 발휘할 수 있다.

### ④ 코딩 없이 확장 가능

마크다운만 쓸 줄 알면 Claude에게 새로운 능력을 추가할 수 있다.  
개발자가 아니어도 된다.

### ⑤ 테스트·개선 사이클 지원

`skill-creator` 예시 스킬을 통해:
- 테스트 케이스 작성 → 스킬 있을 때 vs 없을 때 결과 비교
- 정량적·정성적 평가 → SKILL.md 개선
- 만족할 때까지 반복

---

## 5. 스킬 내부 구조 해부

### SKILL.md 파일 구조

```markdown
---
name: skill-name                     ← 스킬 식별자
description: "언제 이 스킬을 쓸지,   ← 트리거 메커니즘 (가장 중요!)
  무엇을 하는지 상세히 기술.
  키워드도 여기에 포함."
license: 라이선스 정보 (선택)
---

# 스킬 제목

## 개요
스킬이 해결하는 문제

## Quick Reference
| 작업 | 명령어/방법 |

## 상세 절차
단계별 실행 방법

## 참조 파일
추가 문서 링크 (references/ 폴더)

## QA 체크리스트
작업 완료 후 검증 항목
```

### 번들 리소스 구조

```
my-skill/
├── SKILL.md                   ← 필수 (진입점)
├── references/                ← 대용량 참조 문서
│   ├── advanced-guide.md
│   └── api-reference.md
├── scripts/                   ← 재사용 유틸리티 스크립트
│   ├── convert.py
│   └── validate.sh
└── assets/                    ← 템플릿, 아이콘, 폰트
    └── template.pptx
```

### description 필드가 핵심인 이유

`description`은 단순한 설명이 아니라 **트리거 메커니즘**이다.

```yaml
# 나쁜 예 — 너무 좁아서 필요할 때 안 트리거됨
description: "PowerPoint 파일 생성"

# 좋은 예 — 다양한 상황에서 정확하게 트리거됨
description: "Use this skill any time a .pptx file is involved in any way —
  as input, output, or both. Trigger whenever the user mentions 'deck,'
  'slides,' 'presentation,' or references a .pptx filename, regardless
  of what they plan to do with the content afterward."
```

---

## 6. 스킬 만드는 법 — 단계별 가이드

### Step 1: 의도 정의

스킬을 만들기 전에 세 가지를 명확히 한다:

```
1. 이 스킬은 무엇을 가능하게 하는가?
2. 언제 트리거되어야 하는가? (어떤 사용자 표현/상황에서)
3. 예상 출력 형식은 무엇인가?
```

### Step 2: SKILL.md 작성

#### YAML 프론트매터 (트리거 키워드 포함)

```yaml
---
name: my-skill
description: >
  [무엇을 하는지]
  [언제 사용할지 - 구체적인 트리거 표현과 키워드 포함]
  트리거 키워드: keyword1, keyword2, keyword3
---
```

#### 본문 작성 원칙

```
✅ 좋은 스킬
- 명령형 문체 ("확인하라", "사용하라")
- 왜 중요한지 이유 설명 포함
- 구체적인 예시 포함
- 흔한 실수 목록 포함
- QA 체크리스트 포함
- 500줄 이내 (넘으면 references/ 파일로 분리)

❌ 나쁜 스킬
- 모호한 지침 ("잘 만들어라")
- 이유 없는 규칙 나열
- 예외 상황 미처리
- QA 없음
```

### Step 3: 디렉토리 배치

```bash
# 사용자 커스텀 스킬 위치
/mnt/skills/user/my-skill/SKILL.md

# Claude가 자동으로 description을 읽고 트리거 여부 판단
```

### Step 4: 테스트 (skill-creator 방법론)

```json
// evals/evals.json
{
  "skill_name": "my-skill",
  "evals": [
    {
      "id": 1,
      "prompt": "실제 사용자가 할 법한 요청",
      "expected_output": "기대하는 결과 설명"
    }
  ]
}
```

1. 테스트 프롬프트 2~3개 작성
2. 스킬 있을 때 vs 없을 때 결과 비교
3. 결과 평가 후 SKILL.md 수정
4. 만족할 때까지 반복

---

## 7. 직접 만든 예시 스킬

아래는 **한국어 비즈니스 이메일** 스킬의 실제 구현이다.

### 파일: `examples/korean-business-email/SKILL.md`

```markdown
---
name: korean-business-email
description: >
  Use this skill whenever the user asks to write a Korean business email,
  formal letter, or official document in Korean.
  Triggers: '이메일 써줘', '업무 메일', '공문', '비즈니스 메일',
  '정중한 메일', '회사 메일', '거래처 메일'
---

# 한국어 비즈니스 이메일 스킬

## 이메일 기본 구조

[제목] 목적이 명확하게 (20자 이내)
1. 인사 + 자기소개 (첫 연락 시)
2. 이메일 목적 (두 번째 줄에 바로)
3. 핵심 내용 (단락 구분 명확히)
4. 요청/확인 사항 (번호 매기기)
5. 마무리 인사
6. 서명 (소속 / 이름 / 연락처)

## 존댓말 규칙
- 금지: ~해요체 (업무 메일에 너무 가벼움)
- 권장: ~합니다/~습니다 체 (합쇼체)

## QA 체크리스트
- [ ] 제목이 목적을 명확히 나타내는가?
- [ ] 합쇼체를 일관되게 사용했는가?
- [ ] 요청 사항에 기한이 명시되어 있는가?
- [ ] 서명이 포함되어 있는가?
```

### 이 스킬의 포인트

| 항목 | 내용 |
|------|------|
| **트리거** | "업무 메일 써줘", "비즈니스 이메일", "공문" |
| **핵심 가치** | 합쇼체 규칙, 구조, 흔한 실수 방지 |
| **QA** | 완성 후 체크리스트 자동 적용 |
| **공식 스킬과의 공통점** | YAML 프론트매터, 규칙+예시+QA 구조 동일 |

---

## 8. 공식 스킬 활용 예시 (pptx 스킬)

pptx 스킬이 실제로 Claude에게 주입하는 전문 지식 일부:

### 디자인 색상 팔레트 (스킬이 제공하는 표준)

| 테마 | Primary | Secondary | Accent |
|------|---------|-----------|--------|
| Midnight Executive | `#1E2761` (navy) | `#CADCFC` (ice blue) | `#FFFFFF` |
| Coral Energy | `#F96167` (coral) | `#F9E795` (gold) | `#2F3C7E` |
| Warm Terracotta | `#B85042` | `#E7E8D1` (sand) | `#A7BEAE` |

### QA 절차 (스킬이 강제하는 검증)

```bash
# 1. 텍스트 검증
python -m markitdown output.pptx

# 2. 이미지 변환 후 시각 검증
python scripts/office/soffice.py --headless --convert-to pdf output.pptx
pdftoppm -jpeg -r 150 output.pdf slide

# 3. 문제 발견 → 수정 → 재검증 반복
```

### 스킬이 금지하는 것들

```
❌ 타이틀 아래 장식선 (AI 생성물처럼 보임)
❌ 텍스트만 있는 슬라이드 (시각 요소 필수)
❌ 모든 색상에 동일한 비중 (주색 60-70% 원칙)
❌ 본문 텍스트 가운데 정렬 (제목만 가능)
```

---

## 요약

```
스킬   = 마크다운 지침서 파일 (SKILL.md)
트리거 = description 필드의 키워드 매칭
로딩   = Claude가 view 도구로 SKILL.md를 읽음
실행   = 지침에 따라 작업 수행

장점:
  - 일관성: 항상 동일한 표준 절차
  - 전문성: 도메인 노하우 캡슐화
  - 효율성: 3단계 점진적 로딩
  - 접근성: 마크다운만 쓸 줄 알면 누구든 만들 수 있음
```

---

*분석 기준: `/mnt/skills/public/` 및 `/mnt/skills/examples/` 직접 탐색*  
*작성일: 2026-03-10*
