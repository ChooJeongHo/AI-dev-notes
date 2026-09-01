# 101 - Predictive Back Gesture — "마이그레이션할 게 없어서" 조사 방향을 바꾼 하루

**커스텀 뒤로가기 코드는 애초에 없었다 — 대신 실기기에서 직접 스와이프해보고서야, 문제의 원인이 코드가 아니라 XML 리소스 타입(res/anim vs res/animator)이었다는 걸 발견했다**

---

## 목적

070~100일차가 계속 "만든 것을 재검증"하는 흐름이었다면, 101일차는 완전히 새로운 Android UX 기능(Predictive Back Gesture)을 처음 다룬다.

---

## 작업 1 — 조사: 마이그레이션 대상 자체가 없었다

당초 예상은 "레거시 onBackPressed() 방식을 OnBackPressedCallback으로 마이그레이션"이었는데, 실제로 조사해보니:

- 커스텀 뒤로가기 처리(onBackPressed(), OnBackPressedCallback, KEYCODE_BACK 인터셉트, 더블탭 종료 등)가 프로젝트에 전혀 없었다. 다이얼로그의 dismiss() 호출들도 전부 "새 다이얼로그를 열기 전 이전 것을 닫는" 용도였지, back 콜백이 아니었다.
- android:enableOnBackInvokedCallback="true" + targetSdk 36은 이미 설정되어 있었다 (CLAUDE.md에도 이미 기록돼 있었음).
- Fragment 1.9.0 / Navigation 2.9.8 / Material 1.14.0 — predictive back을 지원하는 충분히 최신 라이브러리였다.

즉 원래 계획했던 "작업 2 마이그레이션" 자체가 성립하지 않는 상황이었다.

---

## 작업 2 — 계획 수정: 실기기로 확인해서야 진짜 원인을 찾다

Claude Code가 "전환할 코드가 없다"고 보고했을 때, 095/098일차의 교훈("짐작하지 말고 직접 확인하라")을 그대로 적용해서 "일단 실기기에서 실제로 스와이프해보고 판단하자"는 방향으로 갔다.

### 실기기(SM-S926N) 검증에서 드러난 진짜 문제

| 계층 | 결과 |
|---|---|
| 앱 종료(OS 레벨) | predictive back 완벽 동작 |
| 일반 Fragment 전환 | 뒤로가기는 되지만 스와이프 진행률 애니메이션 없음 — 손을 떼는 순간 순간이동 |
| DetailFragment 공유 요소 전환 | 마찬가지로 진행률 애니메이션 없음 |
| 다이얼로그 | 마찬가지로 진행률 애니메이션 없음 |

### 핵심 발견 — 원인은 코드가 아니라 XML 리소스 타입이었다

> nav_graph.xml의 액션들은 @anim/slide_in_right, @anim/fade_out 같은 애니메이션을 쓰는데, 실제 파일은 <translate>/<alpha> 태그를 쓰는 레거시 View Animation(tween) 방식이었다. Predictive Back의 "스와이프 진행률에 맞춰 화면을 스크러빙하는" 효과는 Animator(res/animator/ 폴더)가 지원하는 setCurrentPlayTime() 같은 seek 기능이 있어야 가능한데, 레거시 View Animation은 이 seek 기능이 없다. Fragment 1.7+가 predictive back을 자동 지원하는 조건이 바로 "Animator 기반 전환을 쓰고 있는가"였다.

이건 코드 로직 문제가 아니라 "어떤 종류의 XML 리소스를 참조하는가"의 문제였다 — 실기기로 직접 스와이프하며 프레임을 비교하지 않았다면 절대 발견하지 못했을 원인이다.

### 리스크 기반으로 범위를 좁혀서 진행

| 대상 | 리스크 | 진행 여부 |
|---|---|---|
| 일반 Fragment 전환 (res/anim → res/animator) | 낮음 | 진행 |
| DetailFragment 공유 요소 전환 (레거시 TransitionSet) | 기존 500ms postponeEnterTransition + Coil 리스너 흐름을 건드릴 수 있어 높음 | 리스크만 문서화, 보류 |

실제 변경: fade_in/fade_out/slide_in_right/slide_out_right 4개를 res/animator/로 전환, nav_graph.xml 8곳의 @anim/ → @animator/ 참조 변경, nav_transition_slide_distance(480dp) 신규 추가.

---

## 작업 3 — 테스트 + 회귀 검증

- 새 왕복 네비게이션 회귀 테스트 추가 (실기기 통과 확인)
- 전체 유닛 테스트 + 전체 Espresso 테스트(24개) 실기기 재실행 — 전부 통과, 회귀 없음

---

## 작업 4 — 실기기 검증에서 만난 예상 밖 장애물

Samsung One UI는 adb settings put secure navigation_mode 2만으로는 실제 제스처 바가 안 바뀌는 것도 확인했다 — 기기 설정 앱에서 직접 전환해야 했다. 변경 전/후를 실제 스와이프 프레임 캡처로 비교해서 개선을 시각적으로 확인했다.

---

## 작업 5 — 정리

"뒤로가기 처리는 전혀 흩어져 있지 않았다" — 애초에 커스텀 코드가 없어서 "마이그레이션"이라 부를 게 없었던 케이스였다. 대신 진짜 관건은 "왜 스와이프해도 미리보기가 안 나오는가"였고, 답은 코드가 아니라 리소스 타입의 차이였다.

CLAUDE.md에 predictive back 관련 지식(왜 res/animator인지, 공유요소 전환의 알려진 한계, 480dp 근사치의 이유)을 문서화해서 다음 세션이 같은 조사를 반복하지 않도록 했다.

---

## 095/098일차 교훈이 오늘 어떻게 실전에 적용됐나

| 이전 교훈 | 오늘 적용 |
|---|---|
| 095일차: "AI가 짐작으로 진행하면 위험하다" | "마이그레이션 대상이 없다"는 조사 결과를 받고, 바로 다음 단계로 넘어가지 않고 실기기로 실제 동작을 확인하는 방향으로 전환 |
| 098일차: "성급하게 결론 내리지 말고 근거를 확인하라" | 공유요소 전환 리스크를 Claude Code가 두 번 반복 경고했을 때, 위험 대비 이득이 낮은 작업은 진행하지 않기로 명확히 판단 |

---

## 오늘 배운 것 — 한 줄 정리

1. "전환할 코드가 없다"는 결과도 유효한 조사 결과다 — 예상과 다른 결과가 나왔을 때 계획을 억지로 밀어붙이지 않고 방향을 바꾸는 게 맞다.
2. Predictive Back의 스와이프 진행률 애니메이션은 Animator(seek 가능) 기반 전환에서만 지원되고, 레거시 View Animation(res/anim)은 지원하지 않는다.
3. 이런 종류의 문제는 코드를 읽는 것만으로는 절대 못 찾고, 실기기에서 직접 스와이프해봐야만 드러난다.
4. 리스크가 큰 작업(기존에 잘 튜닝된 흐름을 건드리는 것)은, 이득(시각 효과 개선)이 크지 않다면 보류하는 게 합리적 판단이다.
5. Samsung One UI 같은 제조사 커스텀 UI는 표준 adb 명령이 그대로 안 먹힐 수 있다 — 실기기 설정 앱에서 직접 확인이 필요하다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*
*작성일: 2026-09-01*
