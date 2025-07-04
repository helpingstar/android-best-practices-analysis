
## fun HomeScreen


```kotlin
@ExperimentalMaterial3ExpressiveApi  
@OptIn(ExperimentalMaterial3Api::class, ExperimentalMaterial3ExpressiveApi::class)  
@Composable  
fun HomeScreen(  
    homeScreenViewModel: HomeViewModel = hiltViewModel(),  
    isMediumWindowSize: Boolean = isAtLeastMedium(),  
    onClickLetsGo: (IntOffset) -> Unit = {},  
    onAboutClicked: () -> Unit = {},  
)
```

```kotlin
val state = homeScreenViewModel.state.collectAsStateWithLifecycle()
```
* `homeScreenViewModel.state` : 일반적으로 `StateFlow` 또는 `Flow` 타입
* `collectAsStateWithLifecycle()` : `StateFlow` 또는 `Flow`를 lifecycle를 인식하는 `State<T>`로 변환해준다.
	* `State<T>` : Jetpack Compose에서 UI 상태를 추적하고 자동으로 recomposition(화면 갱신)을 트리거하는 객체
		* `T` 타입의 값을 담고 있는 immutable wrapper
		* 값이 바뀌면 자동으로 Compose UI를 recomposition
	* `Lifecycle.State.STARTED` 이상일 때만 collect
		* `STARTED` 상태 이상(STARTED state or above) : Lifecycle 상태가 `STARTED` 또는 `RESUMED` 상태에 있음
		* View 가 보여지고 있을 때만 collect. -> 불필요한 리소스 사용 방지.

| 항목              | `Flow`              | `StateFlow`          |
| --------------- | ------------------- | -------------------- |
| **Cold vs Hot** | Cold (collect 시 시작) | Hot (항상 실행 중)        |
| **값 유지**        | ❌ 유지 안함             | ✅ 현재 값 유지 (`.value`) |
| **초기값 제공**      | ❌ 없음                | ✅ 있음                 |
| **UI 상태 관리 적합** | ❌ 부적합               | ✅ 적합                 |
| **새 collect 시** | 새로 시작               | 즉시 현재 값 전달           |
| **구독자 없음**      | 동작 안 함              | 여전히 값 보유             |
### @ExperimentalMaterial3ExpressiveApi

**Material3의 "Expressive" 테마 기능**이 **실험적(experimental)** 임
- 이 API는 아직 **안정화되지 않았고, 향후 변경될 수 있는** 기능을 포함한다.
- `Expressive` 테마는 Material 3 디자인 시스템에서 **다양한 창 크기와 장치에 맞춰 더욱 풍부하게 표현되는 UI 스타일**을 지원
	- 예: 대형 화면에서 더 동적인 색상과 레이아웃.

### @OptIn(ExperimentalMaterial3Api::class, ExperimentalMaterial3ExpressiveApi::class)

`@OptIn` : 특정 **experimental API 사용에 대한 동의(opt-in)** 를 명시하는 데 사용
* 이 annotation이 붙은 함수나 클래스에서는 해당 실험적 API들을 **경고 없이 자유롭게 사용할 수 있게** 된다.

### Parameter
#### homeScreenViewModel
[[feature-home-HomeViewModel.kt#class HomeViewModel| HomeViewModel]]
```kotlin
homeScreenViewModel: HomeViewModel = hiltViewModel()
```
#### isMediumWindowSize

* `HomeScreen()` 함수를 호출할 때 `isMediumWindowSize` 파라미터에 값을 전달하지 않으면 기본값인 [[core-util-LayoutUtils.kt#fun isAtLeastMedium|isAtLeastMedium()|]] 함수가 자동으로 호출됨
	* `isAtLeastMedium()` : 현재 화면의 크기가 중간 크기(Medium) 이상인지 판단하는 함수

#### onClickLetsGo

#### onAboutClicked





## fun HomeScreenContents

```kotlin
@Composable  
fun HomeScreenContents(  
    videoLink: String?,  
    dancingBotLink: String?,  
    isMediumWindowSize: Boolean,  
    onClickLetsGo: (IntOffset) -> Unit,  
    onAboutClicked: () -> Unit,  
)
```

* `Box`
	* [[core-theme-components-Backgrounds.kt#fun SquiggleBackground|SquiggleBackground()]]
	* `Column`
		* IF `isMediumWindowSize` == TRUE
			* [[core-theme-components-TopAppBar.kt#fun AndroidifyTranslucentTopAppBar|AndroidifyTranslucentTopAppBar()]]
			* `Row`
				* `Box`
					* [[#private fun VideoPlayerRotatedCard|VideoPlayerRotatedCard()]]
				* `Box`
					* [[#private fun MainHomeContent|MainHomeContent()]]
					* [[#private fun HomePageButton|HomePageButton()]]
						* `onClick = { onClickLetsGo(positionButtonClick) }`
		* IF `isMediumWindowSize` == FALSE
			* [[#private fun CompactPager|CompactPager()]]
## private fun CompactPager

```kotlin
@Composable  
private fun CompactPager(  
    videoLink: String?,  
    dancingBotLink: String?,  
    onClick: (IntOffset) -> Unit,  
    onAboutClicked: () -> Unit,  
)
```
```kotlin
val pagerState = rememberPagerState(pageCount = { 2 })
```
`HorizontalPager` 또는 `VerticalPager`를 사용할 때 Pager state를 관리하는 데 사용되는 부분
* `rememberPagerState(...)` : Compose에서 `HorizontalPager`나 `VerticalPager`를 사용할 때 **현재 페이지 번호나 스크롤 위치 등을 State로 유지**할 수 있도록 도와주는 함수로 `PagerState`를 반환
	* 즉, 사용자가 스와이프하여 페이지를 넘겼을 때 그 상태를 추적하거나 조작할 수 있다.
	* `pageCount = { 2 }` : 람다 함수로 페이지 수를 정의한다.
		* `pageCount`에 람다를 전달하는 것은 지연 평가(lazy evaluation)를 가능하게 한다.
		* `pageCount = { viewModel.items.size }` : 처럼 동적으로 페이지 수를 지정할 수 있다.
	* `PagerState`
		* `currentPage` : The page that sits closest to the snapped position. This is an observable value and will change as the pager scrolls either by gesture or animation.
		* `pageCount` : The total amount of pages present in this pager. The source of this data should be observable.
* `Column`
	* [[core-theme-components-TopAppBar.kt#fun AndroidifyTopAppBar|AndroidifyTopAppBar()]]
	* `HorizontalPager()`
		* page == 0
			* [[#private fun MainHomeContent|MainHomeContent]]
				* 
		* page != 0
## private fun VideoPlayerRotatedCard

```kotlin
@Composable  
private fun VideoPlayerRotatedCard(  
    videoLink: String?,  
    modifier: Modifier = Modifier,  
)
```

## private fun MainHomeContent

```kotlin
@Composable  
private fun MainHomeContent(  
    dancingBotLink: String?,  
    modifier: Modifier = Modifier,  
)
```


## private fun HomePageButton
![[Pasted image 20250702092523.png]]

```kotlin
private fun HomePageButton(  
    modifier: Modifier = Modifier,  
    colors: ButtonColors = ButtonDefaults.buttonColors().copy(containerColor = Blue),  
    onClick: () -> Unit = {},  
)
```