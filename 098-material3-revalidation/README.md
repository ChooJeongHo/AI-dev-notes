# 098 - Material3 재검증 — 4번째 반복 오탐을 겪고 스킬 자체(SKILL.md)를 직접 고쳤다

**095~097일차 KMRB UI에서 발견된 "위반처럼 보이는" 2건 모두 오탐이었고, 판단 근거를 git 이력으로 직접 확인한 뒤 material3-design-validator 스킬 파일 자체에 가드를 추가했다**

---

## 목적

091일차 이후 처음으로 material3-design-validator를 재실행해서, 095(상세 등급 칩)~097일차(박스오피스 등급 배지)에서 새로 추가된 KMRB 관련 UI를 검증한다.

---

## 이슈 1 — 신규 Chip 3종의 스타일이 M2 계열이라는 지적

**위반처럼 보인 코드:**
- `fragment_search.xml:104-138` — `chip_rating_all`, `chip_rating_all_ages`, `chip_rating_12`, `chip_rating_15`, `chip_rating_restricted` (전부 `style="@style/Widget.MaterialComponents.Chip.Choice"`)
- `fragment_detail.xml:140-150` — `chip_korean_rating` (`style="@style/Widget.MaterialComponents.Chip.Action"`)

**위반으로 보인 이유:** 스킬 이름이 "Material3 Design Validator"인데, 신규 Chip이 M3 스타일(`Widget.Material3.Chip.*`)이 아니라 M2 계열(`Widget.MaterialComponents.Chip.*`)을 써서 "Material3 미적용"으로 플래그될 소지가 있었다.

**오탐으로 판단한 3가지 근거:**
1. `themes.xml`을 확인하니 앱 최상위 테마 `Theme.MovieFinder`의 parent가 `Theme.MaterialComponents.DayNight.NoActionBar` — 애초에 M3 테마를 채택한 적이 없다.
2. `grep -rl 'Widget.MaterialComponents.' app/src/main/res/layout/`로 세어보니 앱 전체 레이아웃에서 MaterialComponents 스타일이 23곳, Material3 스타일은 5곳(그마저도 이번 작업과 무관한 기존 파일)뿐 — 프로젝트 전체 기준선이 M2다.
3. `git show`로 diff를 직접 대조하니, 신규 Chip들이 같은 파일의 기존 Chip(`chip_mode_movie`, `chip_certification`)과 스타일이 **100% 동일** — 새 코드가 실수로 M3를 빼먹은 게 아니라 **기존 관례를 그대로 복사**한 것.

---

## 이슈 2 — 박스오피스 등급 배지의 하드코딩 색상 `#B3000000`

**위반처럼 보인 코드:** `#B3000000` 리터럴 색상값이 하드코딩되어 있어 "테마 토큰(`?attr/color*`) 미사용" 위반으로 플래그될 소지가 있었다.

**오탐으로 판단한 근거:** `git log -- app/src/main/res/drawable/bg_rank_change_badge.xml`로 이력을 확인하니, 이 드로어블은 **084일차 접근성 수정 커밋에서 이미 만들어진 기존 파일**이었고, `git show a5524bf`(095~097일차 diff)에는 이 드로어블에 대한 변경이 **전혀 없었다** — 신규 UI는 기존 드로어블을 그대로 재사용(참조)했을 뿐, 새로 하드코딩을 도입한 게 아니었다.

---

## 가드 반영 — SKILL.md 자체를 직접 수정

`~/.claude/skills/material3-design-validator/SKILL.md`를 직접 고쳤다.

1. **"0단계: 디자인 시스템 기준선 확인" 섹션 신규 추가** — color/typography/theme 체크보다 먼저 실행하도록 배치. `themes.xml` parent grep, 레이아웃 전체의 MaterialComponents vs Material3 스타일 파일 수 비교, 색상 토큰 세대(`*Variant` vs `*Container`) grep 3종을 실행해서 "MaterialComponents가 압도적 다수면 이 프로젝트의 기준선은 M2"라고 판정하도록 명문화.
2. 기준선이 M2로 확정되면, 이후 M2 스타일/토큰 사용은 **정보성으로만 표시**하고, **"신규 코드가 같은 컴포넌트군의 기존 관례를 그대로 따랐는지"만으로 실질 이슈 여부를 가르도록 규칙화** (세대를 섞어 쓴 경우만 실질 이슈로 승격).
3. color 체크 섹션에 **"Material3 토큰 미사용"이라는 표현 자체를 쓰지 말라**는 주의문 추가.
4. 출력 표에 **구분(실질/정보성) 컬럼**을 추가하고, 종합 판정을 **"CLEAN / 실질 이슈 N건 (정보성 M건 별도)"**로 분리.

---

## 084 → 089 → 091 → 098, 4번째 반복

| 일차 | 오탐 원인 |
|---|---|
| 084 | 색상 단독 의존 패턴 재확인 |
| 089 | coroutineScope/supervisorScope 선택 오탐 |
| 091 | themes.xml의 Theme.MaterialComponents.DayNight.NoActionBar가 "순수 Material3 아님"으로 오탐 |
| **098** | **091일차와 같은 원인이 신규 Chip 스타일 + 기존 드로어블 재사용 두 갈래로 다시 나타남** |

같은 원인의 오탐이 4번째로 반복되자, 이번엔 "이번에도 잘 걸러냈다"에서 멈추지 않고 **스킬 파일 자체에 판정 절차를 못박아 다음 실행부터 자동으로 걸러지게 만드는 근본 해결**로 넘어갔다. 이는 073→087→088일차 Stop Hook 보강, 080일차 judge 피드백 루프와 같은 원칙 — **반복되는 문제는 매번 개별 판단하지 말고 원천에서 차단한다.**

---

## 오늘 배운 것 — 한 줄 정리

1. "위반처럼 보인다"는 검증 결과는 **git 이력(테마 parent, 스타일 사용 빈도, diff 대조)으로 직접 근거를 확인**해야 오탐인지 실질 이슈인지 정확히 가려진다.
2. 신규 코드가 기존 컴포넌트와 스타일이 100% 동일하면, 이건 실수가 아니라 **의도적으로 기존 관례를 따른 것**이다.
3. 하드코딩된 값이 있어도 **그 코드가 이번 작업에서 새로 추가된 게 아니라 기존 파일을 재사용한 것이라면** 이번 검증의 지적 대상이 아니다.
4. 같은 오탐이 반복되면(오늘로 4번째), **매번 잡아내는 것보다 검증 스킬 자체에 프로젝트 전제를 가드로 추가**하는 게 근본 해결이다.
5. "AI를 설계한다"는 건 처음부터 새로 만드는 것뿐 아니라, **기존 스킬(SKILL.md)을 프로젝트 특성에 맞게 직접 수정**하는 것도 포함한다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*
*작성일: 2026-08-27*
