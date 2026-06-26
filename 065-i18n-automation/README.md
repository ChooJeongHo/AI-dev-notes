# 065 - Claude Code로 i18n 자동화 — strings.xml 영어 번역 감사 및 수정

**KO/EN 301개 키 완전 일치 달성 — 누락 3개 추가, 오역 수정, plurals 문법 오류 발견**

---

## 목적

MovieFinder의 한국어/영어 다국어 지원 품질을 자동으로 감사하고 개선.  
수동으로 하나씩 확인하던 작업을 Claude Code가 자동으로 비교/수정.

---

## Android i18n 구조

```
values/strings.xml         → 기본(한국어) — fallback
values-en/strings.xml      → 영어 로케일 오버라이드
```

영어권 기기에서 values-en에 누락된 키가 있으면 → 한국어가 그대로 표시 = UX 버그

---

## 감사 결과

**감사 전:** KO 301개 / EN 298개 (3개 누락)  
**감사 후:** KO 301개 / EN 301개 ✅ 완전 일치

---

## 수정 완료 항목

| 유형 | 내용 |
|------|------|
| 누락 추가 (3개) | `import_empty`, `tmdb_disconnected`, `tmdb_no_browser` |
| 오역 수정 | `notification_channel_description` — "watchlist" → "favorited" (KO 즐겨찾기와 정렬) |
| 품질 개선 | `stats_chart_month_format` — `%1$d mo` → `%1$d` ("3 mo" 어색한 약어 제거) |

---

## 추가 발견 — plurals 문법 오류 (MAJOR)

```xml
<!-- 현재: 영어 기기에서 "1 movies" 문법 오류 발생 -->
<string name="stats_count_format">%d movies</string>

<!-- 개선: CLDR 복수형 규칙 적용 -->
<plurals name="stats_count_format">
    <item quantity="one">%d movie</item>
    <item quantity="other">%d movies</item>
</plurals>
```

영향받는 키 7개: `stats_count_format`, `stats_genre_count`, `reminder_count_format` 등  
수정 시 StatsFragment, FavoriteFragment, ReminderHistoryFragment 코드 변경 필요.

---

## I18N_REPORT.md 주요 내용

| 등급 | 항목 | 내용 |
|------|------|------|
| 🔴 MAJOR | plurals 미적용 (7개 키) | "1 movies", "1 Genres" 문법 오류 |
| 🟡 MINOR | person_known_for 번역 중복 | 칩용/상세용 맥락 분리 권장 |
| ℹ️ INFO | filter_year_value 단위 생략 | 현재 허용 범위 |

---

## 자동화 흐름

```
strings.xml 파싱
    ↓
KO/EN 키 비교 (Python 스크립트)
    ↓
누락 키 탐지 → 자동 번역 추가
오역 탐지 → 수정
포맷 스펙 불일치 탐지
    ↓
I18N_REPORT.md 생성 (151줄)
```

---

## 느낀 점

301개 키를 수동으로 하나씩 비교하면 시간이 엄청 걸린다. Claude Code가 Python 스크립트로 자동 파싱해서 누락/오역/문법 오류를 한 번에 찾아줬다.

특히 plurals 문법 오류는 영어권 사용자가 아니면 발견하기 어려운 버그인데, i18n 감사 과정에서 자동으로 잡혔다.

**"번역은 했는데 품질은 몰랐다" — AI가 국제화 품질을 수치로 만들어준다.**

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-06-26*
