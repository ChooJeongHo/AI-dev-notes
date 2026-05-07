# 031 - 멀티모달 활용 — 스크린샷 → Compose 코드 자동 생성

**이미지를 입력으로 주면 AI가 UI 구조를 분석하고 Compose 코드를 생성하는 실험**

---

## 목적

텍스트가 아닌 **이미지를 입력**으로 써서 UI 코드를 생성하는 멀티모달 AI 활용 실험.  
MovieFinder 홈 화면 스크린샷을 Claude에게 주고 Compose 코드 자동 생성.

---

## 실험 방법

Claude Code 터미널은 이미지 입력이 불가능하기 때문에 **Claude.ai 채팅창**에서 진행.

```
[스크린샷 이미지 첨부]
이 화면을 Compose로 구현해줘.
어떤 컴포넌트들이 필요한지 분석하고 코드도 작성해줘.
```

---

## 분석된 UI 컴포넌트

이미지만 보고 파악한 것들:

| 컴포넌트 | XML | Compose |
|---------|-----|---------|
| 상단 바 | Toolbar | TopAppBar |
| 탭 | TabLayout | TabRow |
| 영화 목록 | RecyclerView (GridLayoutManager) | LazyVerticalGrid(GridCells.Fixed(2)) |
| 영화 카드 | CardView + ImageView | Card + AsyncImage |
| 평점 뱃지 | ImageView overlay | Box + clip(CircleShape) |
| 하단 네비게이션 | BottomNavigationView | NavigationBar |

**이미지에서 추출한 세부 정보:**
- 2열 그리드
- 평점 뱃지: 포스터 좌하단 반투명 원형 overlay
- TopAppBar 배경색: 네이비 (#1A3A5C 추정)
- 탭 인디케이터: 흰색

---

## 자동 생성된 Compose 코드

```kotlin
@Composable
fun HomeScreen(
    tabs: List<String> = listOf("현재 상영작", "인기 영화"),
    selectedTab: Int = 0,
    onTabSelected: (Int) -> Unit = {},
    movies: LazyPagingItems<Movie>,
    onMovieClick: (Int) -> Unit = {}
) {
    Scaffold(
        topBar = {
            Column {
                TopAppBar(
                    title = { Text("MovieFinder") },
                    colors = TopAppBarDefaults.topAppBarColors(
                        containerColor = Color(0xFF1A3A5C),
                        titleContentColor = Color.White
                    )
                )
                TabRow(selectedTabIndex = selectedTab) {
                    tabs.forEachIndexed { index, title ->
                        Tab(
                            selected = selectedTab == index,
                            onClick = { onTabSelected(index) },
                            text = { Text(title) }
                        )
                    }
                }
            }
        },
        bottomBar = {
            NavigationBar {
                NavigationBarItem(
                    selected = true,
                    onClick = {},
                    icon = { Icon(Icons.Default.Home, "홈") },
                    label = { Text("홈") }
                )
                NavigationBarItem(
                    selected = false,
                    onClick = {},
                    icon = { Icon(Icons.Default.Search, "검색") },
                    label = { Text("검색") }
                )
                NavigationBarItem(
                    selected = false,
                    onClick = {},
                    icon = { Icon(Icons.Default.Favorite, "즐겨찾기") },
                    label = { Text("즐겨찾기") }
                )
            }
        }
    ) { padding ->
        LazyVerticalGrid(
            columns = GridCells.Fixed(2),
            modifier = Modifier.padding(padding)
        ) {
            items(movies.itemCount) { index ->
                movies[index]?.let { movie ->
                    MovieCard(
                        movie = movie,
                        onClick = { onMovieClick(movie.id) }
                    )
                }
            }
        }
    }
}

@Composable
fun MovieCard(movie: Movie, onClick: () -> Unit) {
    Card(onClick = onClick, modifier = Modifier.padding(4.dp)) {
        Column {
            Box {
                AsyncImage(
                    model = movie.posterUrl,
                    contentDescription = movie.title,
                    contentScale = ContentScale.Crop,
                    modifier = Modifier.fillMaxWidth().aspectRatio(0.67f)
                )
                // 평점 뱃지 — 좌하단 반투명 원형
                Box(
                    modifier = Modifier
                        .padding(6.dp)
                        .size(36.dp)
                        .clip(CircleShape)
                        .background(Color(0xCC000000))
                        .align(Alignment.BottomStart),
                    contentAlignment = Alignment.Center
                ) {
                    Text(
                        text = movie.voteAverage.toString(),
                        color = Color.White,
                        fontSize = 11.sp,
                        fontWeight = FontWeight.Bold
                    )
                }
            }
            Column(modifier = Modifier.padding(8.dp)) {
                Text(
                    text = movie.title,
                    fontWeight = FontWeight.Medium,
                    maxLines = 1,
                    overflow = TextOverflow.Ellipsis
                )
                Text(
                    text = movie.releaseDate,
                    fontSize = 12.sp,
                    color = MaterialTheme.colorScheme.onSurfaceVariant
                )
            }
        }
    }
}
```

---

## 멀티모달이 실전에서 유용한 상황

| 상황 | 활용 방법 |
|------|---------|
| 버그 리포트 | 이상한 화면 스크린샷 → 원인 분석 |
| 디자이너 목업 | 목업 이미지 → 구현 계획 자동 생성 |
| 레거시 앱 | 기존 앱 스크린샷 → Compose 코드 생성 |
| QA 테스트 | 테스트 중 이상 화면 → 즉시 분석 |

---

## Claude Code vs Claude.ai 입력 방식 차이

| 구분 | Claude Code (터미널) | Claude.ai (채팅창) |
|------|-------------------|----------------|
| 텍스트 입력 | ✅ | ✅ |
| 이미지 입력 | ❌ | ✅ |
| 파일 시스템 접근 | ✅ | ❌ |
| MCP 사용 | ✅ | ❌ |

멀티모달(이미지) 활용은 Claude.ai 채팅창에서, 코드 파일 분석은 Claude Code에서.

---

## 느낀 점

코드 파일을 주지 않고 스크린샷 이미지 하나만으로 UI 구조를 파악하고 Compose 코드까지 생성해줬다.

특히 평점 뱃지가 포스터 좌하단에 반투명 원형으로 overlay 되는 것, 네이비 배경색, 2열 그리드 구조 등 세부적인 디자인 요소까지 이미지에서 읽어냈다.

**멀티모달의 핵심:** 텍스트로 설명하기 어려운 것들을 이미지로 대신할 수 있다. 디자이너와 개발자 사이의 커뮤니케이션 비용을 줄여주는 도구로 활용 가능하다.

---

*실험 대상: [ChooJeongHo/MovieFinder](https://github.com/ChooJeongHo/MovieFinder)*  
*작성일: 2026-05-07*
