# 068 - SearchFragment Compose 마이그레이션 (심화) — ViewModel 연동 + 콜백 브릿지 패턴

**027일차가 "State 없는 화면"이었다면, 068일차는 "ViewModel + Paging3 + RecyclerView가 얽힌 785줄 화면"의 하이브리드 마이그레이션**

---

## 목적

027일차(OnboardingFragment)는 State가 없는 단순 화면이었다.  
이번엔 **ViewModel의 StateFlow 구독 + Paging3 무한 스크롤 + RecyclerView와의 상호운용**까지 다루는 실전 마이그레이션.

---

## 왜 SearchFragment 전체가 아니라 "핵심만"인가

SearchFragment는 785줄짜리 화면으로 아래 기능이 얽혀 있었다:
- 검색 입력 + 결과 리스트
- 필터 다이얼로그(연도/장르/정렬)
- 배우 검색 모드
- 오프라인 검색
- Shimmer 로딩
- FAB, Shared Element Transition

**전체를 한 번에 옮기면 리스크가 너무 크다.** 그래서 **"검색 TextField + 결과 목록"만 Compose로, 나머지는 XML 그대로 유지**하는 하이브리드 전략을 택했다. 이게 실무에서도 흔한 마이그레이션 방식이다 (한 번에 다 바꾸지 않고 점진적으로).

---

## 핵심 개념 1 — ComposeView로 XML 안에 Compose 심기

기존 XML의 일부만 Compose로 바꿀 때 쓰는 방법. `ComposeView`는 "XML 레이아웃 안에 박아 넣는 Compose 캔버스"라고 생각하면 된다.

```xml
<!-- Before: XML 검색창 -->
<com.google.android.material.textfield.TextInputLayout>
    <com.google.android.material.textfield.TextInputEditText
        android:id="@+id/et_search" ... />
</com.google.android.material.textfield.TextInputLayout>

<!-- After: Compose를 담는 그릇 -->
<androidx.compose.ui.platform.ComposeView
    android:id="@+id/compose_search_input"
    android:layout_width="match_parent"
    android:layout_height="wrap_content" />
```

```kotlin
// Fragment에서 ComposeView에 실제 Compose 내용을 채워 넣는다
binding.composeSearchInput.setViewCompositionStrategy(
    ViewCompositionStrategy.DisposeOnViewTreeLifecycleDestroyed  // 생명주기에 맞춰 정리
)
binding.composeSearchInput.setContent {
    MaterialTheme {
        SearchInputField(query = ..., onQueryChange = ..., onSearch = ...)
    }
}
```

> **`DisposeOnViewTreeLifecycleDestroyed`가 뭔가?**  
> Compose는 자기만의 생명주기를 가지므로, Fragment의 View가 파괴될 때 Compose 쪽도 같이 정리되도록 명시적으로 연결해줘야 한다. 안 하면 메모리 누수 위험이 있다.

---

## 핵심 개념 2 — 상태 호이스팅 (State Hoisting)

Compose의 핵심 패턴. **컴포저블 자신은 상태를 갖지 않고, 상위(호출하는 쪽)가 상태를 들고 있다가 넘겨준다.**

```kotlin
// SearchInputField는 텍스트를 "직접 소유"하지 않는다
@Composable
fun SearchInputField(
    query: String,              // 상태를 파라미터로 받기만 함
    onQueryChange: (String) -> Unit,  // 변경은 콜백으로 위로 전달
    onSearch: () -> Unit,
)
```

```kotlin
// Fragment가 실제 상태를 들고 있음 (mutableStateOf)
private var searchQueryState = mutableStateOf("")

// XML의 binding.etSearch.text 읽기/쓰기를 이 한 변수로 통일
```

| XML 방식 | Compose 방식 |
|---------|-------------|
| `binding.etSearch.text.toString()` 매번 읽기 | `searchQueryState.value`로 한 곳에서 관리 |
| `binding.etSearch.setText(query)` | `searchQueryState.value = query` |
| EditText 자체가 상태를 들고 있음 | 상태는 Fragment가 소유, TextField는 그리기만 |

**왜 이렇게 하나?** 상태가 한 곳(Fragment)에만 있으면, "최근 검색어 클릭 → 검색창에 텍스트 채우기"처럼 여러 곳에서 검색어를 건드려야 할 때도 코드가 꼬이지 않는다.

---

## 핵심 개념 3 — Paging3 + Compose: collectAsLazyPagingItems()

```kotlin
// XML/RecyclerView 방식 (기존)
lifecycleScope.launch {
    viewModel.searchResults.collectLatest { searchAdapter.submitData(it) }
}
lifecycleScope.launch {
    searchAdapter.loadStateFlow.collectLatest { handleLoadStates(it) }
}

// Compose 방식 (신규)
val pagingItems = viewModel.searchResults.collectAsLazyPagingItems()
// pagingItems 하나에 데이터 + 로딩 상태가 전부 들어있음
LaunchedEffect(pagingItems.loadState, pagingItems.itemCount) {
    handleLoadStates(pagingItems.loadState, pagingItems.itemCount)
}
```

`collectAsLazyPagingItems()`는 Flow를 Compose가 구독할 수 있는 `LazyPagingItems` 객체로 바꿔준다. RecyclerView의 `PagingDataAdapter` 역할을 대신한다고 보면 된다.

---

## 핵심 개념 4 — LazyColumn / LazyVerticalGrid = RecyclerView의 Compose 버전

```kotlin
// GRID 모드: 예전 GridLayoutManager
LazyVerticalGrid(
    columns = GridCells.Fixed(spanCount),
) {
    items(count = pagingItems.itemCount, key = pagingItems.itemKey { it.id }) { index ->
        pagingItems[index]?.let { movie -> MoviePosterCard(movie, onClick = {...}) }
    }
}

// LIST 모드: 예전 LinearLayoutManager
LazyColumn {
    items(count = pagingItems.itemCount, key = pagingItems.itemKey { it.id }) { index ->
        pagingItems[index]?.let { movie -> MovieListRow(movie, onClick = {...}) }
    }
}
```

| XML/RecyclerView | Compose |
|---|---|
| `GridLayoutManager` / `LinearLayoutManager` 교체 | `LazyVerticalGrid` / `LazyColumn` 각각 별도 컴포저블 |
| `adapter.viewMode = mode` 후 `notifyDataSetChanged()` | `when (viewMode) { ... }`로 아예 다른 컴포저블 분기 |
| `RecyclerView.ViewHolder` | 없음 — 각 아이템이 그냥 함수 호출 |

**`key = pagingItems.itemKey { it.id }`가 중요한 이유:** RecyclerView의 `DiffUtil`과 같은 역할. 아이템의 고유 id를 알려주면 Compose가 "어떤 아이템이 실제로 바뀌었는지" 정확히 알아서 불필요한 다시 그리기를 줄인다.

---

## 핵심 개념 5 — 콜백 브릿지 패턴 (Compose ↔ XML FAB 연결)

RecyclerView가 사라지면서 기존 `addOnScrollListener`로 구현했던 "스크롤하면 FAB 보이기/숨기기"가 끊어졌다. 이걸 다시 연결한 방법:

```kotlin
// Compose 쪽: 스크롤 상태를 감지해서 콜백으로 밖에 알려줌
val gridState = rememberLazyGridState()
LaunchedEffect(gridState) {
    snapshotFlow { gridState.canScrollBackward }.collect(onScrollStateChanged)
}

// "맨 위로 스크롤" 동작 자체도 람다로 등록해서 밖에서 호출 가능하게 함
LaunchedEffect(Unit) {
    onScrollControllerReady { scope.launch { gridState.animateScrollToItem(0) } }
}
```

```kotlin
// Fragment(XML) 쪽: 콜백을 받아서 XML의 FAB와 연결
SearchResultsList(
    ...,
    onScrollStateChanged = { canScrollBack ->
        if (canScrollBack) binding.fabScrollTop.show() else binding.fabScrollTop.hide()
    },
    onScrollControllerReady = { scrollToTop -> scrollToTopAction = scrollToTop },
)

// FAB 클릭 시 Compose 쪽에 저장해둔 함수를 그냥 호출
binding.fabScrollTop.setOnClickListener {
    scrollToTopAction?.invoke()
    binding.fabScrollTop.hide()
}
```

**핵심 아이디어:** Compose와 XML은 서로 직접 참조할 수 없다. 그래서 "함수를 변수에 담아 서로 전달"하는 방식(콜백 브릿지)으로 두 세계를 연결한다.

> **`snapshotFlow`란?**  
> Compose의 State(예: `canScrollBackward`)를 일반 코루틴 Flow처럼 구독할 수 있게 변환해주는 함수. Compose 밖의 로직(FAB show/hide)과 연결할 때 쓴다.

---

## 트러블슈팅 — 자주 만난 컴파일 오류

| 오류 | 원인 | 해결 |
|------|------|------|
| `Unresolved reference 'Icons'` | `material-icons-extended` 의존성 미추가 | 프로젝트에 이미 있는 벡터 드로어블을 `painterResource()`로 재사용 |
| `No value passed for parameter 'items'` | `items()`가 최상위 함수가 아니라 `LazyListScope`의 멤버 함수인데 잘못된 import 충돌 | 불필요한 `import ... as gridItems` 제거 |
| `Unused import [NoUnusedImports]` (Detekt) | 코드 리팩토링 중 안 쓰게 된 import | PostToolUse 훅이 즉시 알려줘서 바로 제거 |

---

## 의도적으로 남긴 트레이드오프

| 기존 XML 기능 | Compose 전환 후 |
|--------------|----------------|
| Shared Element Transition (포스터 클릭 시 확대 애니메이션) | ❌ 제외 — Compose 항목엔 실제 포스터 View가 없어서 |
| CircularRatingView (원형 평점 그래프) | "★ 7.2" 텍스트로 단순화 |
| 리스트 fall-down 애니메이션 | 이번 범위에서 제외 |

**왜 트레이드오프를 그냥 받아들였나?** 이번 실험 목적은 "완벽한 마이그레이션"이 아니라 **"핵심 패턴을 이해하는 것"**. 부가 기능까지 다 옮기려다 범위가 커지면 배움의 밀도가 떨어진다.

---

## 027일차 vs 068일차 비교

| 항목 | 027일차 (Onboarding) | 068일차 (Search) |
|------|:--------------------:|:-----------------:|
| 상태 관리 | 없음 | ViewModel StateFlow + mutableStateOf |
| 데이터 소스 | 정적 리스트 | Paging3 무한 스크롤 |
| 목록 렌더링 | 없음 (페이지 스와이프만) | LazyColumn / LazyVerticalGrid |
| XML과의 공존 | 전체 페이지 교체 | **부분 교체 (하이브리드)** |
| 외부 연동 | 없음 | FAB, 필터 다이얼로그와 콜백 브릿지로 연결 |

---

## 검증

```bash
./gradlew assembleDebug   # ✅ 통과
./gradlew :app:detekt     # ✅ 통과 (내가 건드린 파일은 0 오류)
```

---

## 오늘 배운 것 — 한 줄 정리

1. **ComposeView**로 XML 안에 Compose를 부분적으로 심을 수 있다.
2. **상태 호이스팅**: 컴포저블은 상태를 갖지 않고, 상위가 들고 있다가 넘겨준다.
3. **collectAsLazyPagingItems()**로 Paging3 Flow를 Compose가 바로 구독할 수 있다.
4. **LazyColumn/LazyVerticalGrid**가 RecyclerView를 대체하고, `key`가 DiffUtil 역할을 한다.
5. Compose와 XML은 직접 연결이 안 되므로 **콜백을 서로 전달하는 "브릿지 패턴"**으로 이어준다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-07-01*
