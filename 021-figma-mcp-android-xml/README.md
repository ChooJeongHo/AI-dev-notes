# AI 활용 기록 021 - Figma MCP로 디자인 → Android XML 자동 변환

> 국비 때 Figma로 디자인만 하고 코드로 못 뽑아냈던 것을 Figma MCP + Claude Code로 해결

## 목적

Figma MCP를 Claude Code에 연동해서 디자인 파일을 직접 읽고 Android XML 레이아웃 코드로 자동 변환하는 실험

## 사용한 도구

- Claude Code CLI (터미널)
- Figma MCP (`figma-developer-mcp`)
- 대상 Figma 파일: 지역 공공시설 예약·탐색 모바일 앱 (116개 프레임)

---

## Figma MCP 설치 과정

```bash
claude mcp add --scope user figma \
  -e FIGMA_API_KEY=토큰값 \
  -- npx -y figma-developer-mcp --stdio
```

**주의: `--stdio` 플래그 필수!**

처음에 `--stdio` 없이 설치했더니 `figma · ✘ failed` 오류 발생.

원인: `figma-developer-mcp`가 기본적으로 HTTP 서버 모드(포트 3333)로 실행되는데 Claude Code는 stdio 모드를 기대하기 때문. `--stdio` 플래그를 추가해야 정상 연결.

---

## 실험 과정

### 1단계 - Figma 파일 분석

```
Figma MCP로 이 파일을 읽어서 어떤 화면들이 있는지 분석해줘.
https://www.figma.com/design/l0Hc7xXm8aFlEjtmcjUNWJ/최종-프로젝트-3조
```

**분석 결과:**

| 카테고리 | 개수 | 주요 화면 |
|---------|------|---------|
| 메인·카테고리 홈 | 19개 | 메인 페이지 + 상태별 변형 |
| 상세 페이지 | 14개 | 시설 상세 + 예약 상태별 변형 |
| 지도·검색 | 9개 | 지도, 마커 클릭, 필터 |
| 온보딩 | 9개 | 스플래시, 카테고리/지역 선택 |
| 마이페이지 | 6개 | 프로필, 관심 카테고리 편집 |
| 컴포넌트·에셋 | 41개 | 디자인 시스템 |

### 2단계 - Android XML 코드 변환

```
Figma MCP로 이 파일의 메인 페이지 화면을 읽고
Android XML 레이아웃 코드로 변환해줘.
```

**생성된 파일:**

| 파일 | 설명 |
|------|------|
| `res/layout/activity_main.xml` | 메인 페이지 전체 레이아웃 (567줄) |
| `res/layout/item_facility_card.xml` | 시설 카드 재사용 컴포넌트 |
| `res/drawable/bg_circle_pink.xml` | 앱 로고 원형 |
| `res/drawable/bg_gradient_bottom_dark.xml` | 카테고리 이미지 하단 그라데이션 |
| `res/drawable/bg_status_badge.xml` | 접수중 뱃지 |
| `res/drawable/bg_price_badge.xml` | 유료 뱃지 |
| `res/drawable/bg_bottom_nav.xml` | 하단 네비게이션 배경 |
| `res/drawable/bg_rounded_4/10/12.xml` | cornerRadius용 shape |

---

## Figma → Android 변환 매핑

| Figma | Android |
|-------|---------|
| 절대 좌표 (x, y) | ConstraintLayout margin/bias |
| borderRadius | shape XML corners |
| linear-gradient 180deg | GradientDrawable angle=270 |
| 프레임 (411×1010dp) | NestedScrollView + 고정 BottomNav |
| 비대칭 그리드 (좌3+우2) | 수평 LinearLayout 2개 |
| 카드 컴포넌트 | item_facility_card.xml (include) |

**주의:** Figma CSS `angle 180deg` → Android `angle 270` (Android는 시계 반대 방향 기준)

---

## 추가로 필요한 파일들

자동 변환 후 직접 추가해야 하는 것들:

### 아이콘 (Figma에서 직접 export 필요)
- `res/drawable/ic_search.xml`
- `res/drawable/ic_bell_outline.xml`
- `res/drawable/ic_home.xml`
- `res/drawable/ic_map.xml`
- `res/drawable/ic_person.xml`
- `res/drawable/ic_badminton.xml` 등 카테고리 아이콘 5개

### 이미지
- `res/drawable/img_sports.png` 등 시설 이미지 (Figma에서 PNG export)

### 코드 수정 필요
- `HorizontalScrollView` → `RecyclerView`로 교체 (동적 데이터 바인딩)
- `@drawable/ic_*` placeholder → 실제 아이콘 파일로 교체

---

## 느낀 점

국비 수업 때 Figma로 디자인만 하고 코드로 못 뽑아냈던 게 Figma MCP + Claude Code 조합으로 자동 변환이 됐다.

완벽하게 바로 쓸 수 있는 코드는 아니다. 아이콘 파일이나 이미지는 직접 추가해야 하고, RecyclerView로 교체하는 작업도 필요하다. 하지만 레이아웃 구조, ConstraintLayout 설정, drawable 파일들이 자동으로 생성된 것만으로도 개발 시간이 크게 단축된다.

**Figma MCP가 유용한 상황:**
- 디자이너가 Figma로 준 화면을 Android 코드로 빠르게 변환할 때
- 레이아웃 구조를 파악하고 싶을 때
- 컴포넌트별 색상, 크기, 간격 수치를 추출할 때

---

*실험 대상 Figma: 지역 공공시설 예약·탐색 모바일 앱*  
*작성일: 2026-04-22*
