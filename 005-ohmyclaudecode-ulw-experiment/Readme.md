# AI 활용 기록 005 - oh-my-claudecode ulw 모드 실험

> oh-my-claudecode 플러그인을 설치하고, ulw(ultrawork) 모드와 일반 모드의 접근 방식 차이를 직접 비교한 기록

## 목적

대표님 제안으로 oh-my-claudecode 플러그인을 설치하고 핵심 기능인 `ulw` 모드를 실험  
동일한 기능(통계 화면 차트 시각화)을 일반 모드 vs ulw 모드로 각각 구현하여 차이 비교

## 사용한 도구

- Claude Code CLI (터미널)
- oh-my-claudecode v4.8.2 플러그인
- 실험 대상 프로젝트: [MovieFinder](https://github.com/ChooJeongHo/MovieFinder)

---

## oh-my-claudecode란?

Claude Code CLI에 설치하는 멀티 에이전트 오케스트레이션 플러그인

| 키워드 | 효과 |
|--------|------|
| `ulw` | 여러 에이전트가 병렬로 작업 — 최대 속도 실행 |
| `ralph` | 완성될 때까지 멈추지 않는 집요한 실행 모드 |
| `plan` | 막연한 아이디어를 질문으로 구체화 |
| `/team N:executor` | N개 에이전트 동시 실행 |

### 설치 방법

```bash
# Claude Code CLI 실행 후
/plugin marketplace add https://github.com/Yeachan-Heo/oh-my-claudecode
/plugin install oh-my-claudecode
/omc-setup
```

> Android Studio 플러그인에서는 설치 불가 — Claude Code CLI 전용

---

## 실험: 통계 화면 차트 시각화

4일차에 만든 통계 화면(StatsFragment)에 차트를 추가하는 작업을 두 가지 모드로 비교

### 요청한 프롬프트

```
MovieFinder 통계 화면(StatsFragment)에 차트 시각화를 추가해줘.
- 장르별 시청 비율 파이차트
- 월별 시청 편수 바차트
기존 Clean Architecture + MVVM 패턴 유지해줘.
```

ulw 모드는 앞에 `ulw` 키워드만 추가

---

## 결과 비교

### 일반 모드

| 항목 | 내용 |
|------|------|
| 차트 구현 방식 | MPAndroidChart v3.1.0 **외부 라이브러리** 사용 |
| 추가된 의존성 | JitPack 저장소 + MPAndroidChart 라이브러리 |
| 작업 방식 | 순차적으로 하나씩 처리 |
| 빌드/테스트 검증 | 미기록 |

**변경된 주요 내용:**
- MPAndroidChart 의존성 추가 (settings.gradle.kts, libs.versions.toml, build.gradle.kts)
- WatchHistoryDao: getMonthlyWatchCounts() 쿼리 추가
- WatchStats: allGenreCounts, monthlyWatchCounts 필드 추가
- StatsFragment: setupBarChart(), setupPieChart() 추가

---

### ulw 모드

| 항목 | 내용 |
|------|------|
| 차트 구현 방식 | Canvas 기반 **커스텀 뷰** 직접 구현 |
| 추가된 의존성 | 없음 (외부 라이브러리 미사용) |
| 작업 방식 | PieChartView + BarChartView 병렬 동시 작업 |
| 빌드/테스트 검증 | Build SUCCESS, 145 Tests PASS, Detekt PASS 자동 검증 |

**변경된 주요 내용:**
- PieChartView.kt: Canvas 기반 파이차트 커스텀 뷰 (8색 팔레트, 범례, 다크모드 지원)
- BarChartView.kt: Canvas 기반 바차트 커스텀 뷰 (라운드 바, 그리드 라인, 다크모드 지원)
- StatsFragment: 차트 바인딩 메서드 추가
- strings.xml (ko/en), dimens.xml: 리소스 추가

---

## 핵심 차이 요약

| 비교 항목 | 일반 모드 | ulw 모드 |
|-----------|----------|----------|
| 차트 구현 | 외부 라이브러리 (MPAndroidChart) | Canvas 커스텀 뷰 직접 구현 |
| 외부 의존성 | 추가됨 | 없음 |
| 다크모드 | 별도 설정 필요 | 자동 지원 |
| 에이전트 수 | 1개 순차 처리 | 병렬 에이전트 동시 작업 |
| 빌드/테스트 자동 검증 | 없음 | 있음 (자동으로 검증 후 완료) |

---

## 실행 화면

| 시청 통계 화면 |
|:---:|
| ![stats](./screenshots/stats_chart.png) |

> 장르별 시청 비율 파이차트 + 월별 시청 편수 바차트

---

## 느낀 점

ulw 모드는 단순히 빠른 게 아니라 **접근 방식 자체가 달랐다.**

일반 모드는 외부 라이브러리를 가져다 쓰는 방식을 선택했지만,  
ulw 모드는 의존성을 추가하지 않고 Canvas로 직접 구현했다.  
다크모드까지 자동으로 지원하고, 빌드/테스트 검증까지 스스로 해줬다.

또한 병렬 에이전트가 PieChartView와 BarChartView를 동시에 작업하는 것도 인상적이었다.  
복잡한 작업일수록 ulw 모드가 더 유리할 것 같다.

oh-my-claudecode는 단순한 속도 향상 도구가 아니라,  
**AI가 문제를 바라보는 시각 자체를 바꿔주는 도구**라는 것을 느꼈다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-03-18*
