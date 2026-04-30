# 027 - Claude Code로 XML → Jetpack Compose 마이그레이션

**처음 Compose를 접하는 상태에서 Claude Code가 얼마나 잘 변환하고 설명해주는가**

---

## 목적

XML 기반 MovieFinder의 화면을 Jetpack Compose로 점진적 마이그레이션.  
Compose를 한 번도 써본 적 없는 상태에서 Claude Code의 가이드만으로 변환이 가능한지 실험.

---

## 마이그레이션 대상

| 단계 | 파일 | 이유 |
|------|------|------|
| 1단계 | OnboardingPageFragment | 가장 단순한 순수 표시 화면 (ViewModel/State/Click 없음) |
| 2단계 | OnboardingFragment | dots + ViewPager2 → 선언형 UI의 핵심 개념 체험 |

---

## 1단계 - OnboardingPageFragment

### Before vs After

```kotlin
// Before: 53줄 - 생명주기 직접 관리
private var _binding: FragmentOnboardingPageBinding? = null
private val binding get() = _binding!!
override fun onCreateView(...) = binding.root
override fun onViewCreated(...) { binding.ivOnboardingIcon.setImageResource(...) }
override fun onDestroyView() { _binding = null }  // 메모리 누수 방지 의례

// After: 40줄 - 생명주기는 Compose 런타임이 담당
override fun onCreateView(...): View = ComposeView(requireContext()).apply {
    setViewCompositionStrategy(ViewCompositionStrategy.DisposeOnViewTreeLifecycleDestroyed)
    setContent { MaterialTheme { OnboardingPage(...) } }
}
```

**새로 만들어진 파일:**
```kotlin
@Composable
fun OnboardingPage(icon: Int, title: String, desc: String) {
    Column(
        modifier = Modifier.fillMaxSize().padding(32.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Image(painterResource(icon), contentDescription = null, modifier = Modifier.size(120.dp))
        Spacer(Modifier.height(24.dp))
        Text(title, style = MaterialTheme.typography.headlineSmall)
        Spacer(Modifier.height(16.dp))
        Text(desc, style = MaterialTheme.typography.bodyLarge)
    }
}
```

**배운 개념:** `@Composable`, `Column`, `Modifier`, `painterResource`, `MaterialTheme.typography`

---

## 2단계 - OnboardingFragment

### XML → Compose 패턴 매핑

| XML 패턴 | Compose 패턴 |
|---------|------------|
| ViewPager2 | HorizontalPager { page -> ... } |
| OnPageChangeCallback | pagerState (State라서 자동) |
| dot_0 / dot_1 / dot_2 하드코딩 | repeat(pages.size) {} 한 줄 |
| setBackgroundResource() | animateColorAsState() |
| binding.btnNext.text = | if (isLastPage) ... 자동 리컴포지션 |
| viewPager.currentItem = | pagerState.animateScrollToPage() |
| OnboardingPagerAdapter | HorizontalPager 람다로 대체 |

### dots 변환 — 선언형 UI의 핵심

```kotlin
// Before: dot_0, dot_1, dot_2 각각 View 3개 하드코딩 + updateDots() 함수
fun updateDots(position: Int) {
    listOf(dot0, dot1, dot2).forEachIndexed { index, view ->
        view.setBackgroundResource(if (index == position) activeDrawable else inactiveDrawable)
    }
}

// After: repeat 한 줄 + animateColorAsState
Row {
    repeat(pages.size) { i ->
        val color by animateColorAsState(
            if (i == pagerState.currentPage) activeColor else inactiveColor
        )
        Box(Modifier.size(10.dp).background(color, CircleShape))
    }
}
```

페이지 수가 바뀌어도 코드 수정 불필요 → 이게 선언형 UI의 핵심

**배운 개념:** `remember`, `pagerState`, `HorizontalPager`, `animateColorAsState`, 이벤트 호이스팅

---

## 최종 파일 구조

```
onboarding/
├── OnboardingFragment.kt      ← Hilt + NavController만 담당 (48줄)
├── OnboardingScreen.kt        ← 전체 UI 로직 (115줄) ★ 새 파일
├── OnboardingPage.kt          ← 단일 페이지 컴포저블 (55줄) ★ 새 파일
└── OnboardingPageFragment.kt  ← 미사용 (삭제 가능)
```

---

## 점진적 마이그레이션 방식

기존 Fragment를 삭제하지 않고 `ComposeView`를 사용해서 Fragment 껍데기는 유지하면서 내부만 Compose로 교체.

```kotlin
// Fragment 안에서 Compose 사용
override fun onCreateView(...): View = ComposeView(requireContext()).apply {
    setViewCompositionStrategy(ViewCompositionStrategy.DisposeOnViewTreeLifecycleDestroyed)
    setContent { MaterialTheme { OnboardingScreen(...) } }
}
```

나머지 XML 기반 화면들은 영향 없음 → 실무에서 실제로 쓰는 마이그레이션 방식

---

## 느낀 점

Compose를 한 번도 안 써봤는데 Claude Code가 각 변환마다 XML의 어떤 패턴이 Compose에서 어떻게 바뀌는지 설명해줘서 이해하면서 진행할 수 있었다.

특히 dots 하드코딩(dot_0, dot_1, dot_2) → `repeat(pages.size)` 한 줄로 바뀌는 순간이 "선언형 UI가 왜 좋은가"를 직접 느끼게 해줬다. XML에서는 페이지 수가 바뀌면 코드도 바꿔야 하지만, Compose는 데이터가 바뀌면 UI가 자동으로 따라온다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-04-30*
