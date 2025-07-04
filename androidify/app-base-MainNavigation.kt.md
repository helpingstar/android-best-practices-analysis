
## fun MainNavigation

```kotlin
@ExperimentalMaterial3ExpressiveApi  
@Composable  
fun MainNavigation()
```

> **Jetpack Compose**, **Navigation 3**, 그리고 **Material 3 Expressive API**를 사용해서 **앱의 메인 화면에서 여러 화면 간의 내비게이션과 전환 애니메이션을 관리하는 구성 요소**

```kotlin
val backStack = rememberMutableStateListOf<NavigationRoute>(Home)
```
* 네비게이션 스택을 관리하는 상태 리스트
* 리스트의 내용이 변경될 때마다 타입 안정성을 보장한다.
* 리스트 요소의 타입(제네릭) : [[app-navigation-NavigationRoutes.kt#sealed interface NavigationRoute| `NavigationRoute`]]
* 초기값으로 [[app-navigation-NavigationRoutes.kt#data object Home| `Home`]] 라우트를 선정한다.

```kotlin
var positionReveal by remember {  
    mutableStateOf(IntOffset.Zero)  
}
```
- TODO : 사용자가 "Let's Go" 버튼을 클릭한 정확한 화면 위치를 저장
- `IntOffset.Zero`: 초기값으로 (0, 0) 좌표를 설정

```kotlin
val motionScheme = MaterialTheme.motionScheme
```

### NavDisPlay



`NavDisplay`: Compose 환경에서 직접 정의한 **네비게이션 스택**을 기반으로 화면 전환을 처리해 주는 Composable
* `backStack = backStack`
	* `backStack` : 현재 보여줄 `NavigationRoute` 객체들이 순서대로 담긴다.
* `onBack = { backStack.removeLastOrNull() }`
	* `onBack` : 뒤로가기(Back) 동작이 발생했을 때 호출되는 콜백
	* `removeLastOrNull` : **변경 가능한 리스트**(`MutableList`)에서 마지막 요소를 안전하게 꺼내 반환하고, 리스트가 비어 있으면 `null`을 반환하는 확장 함수
* `entryDecorator` : `NavDisplay`가 각 네비게이션 엔트리(화면) 객체를 생성·표시할 때, 그 엔트리에 추가적인 데코레이션 기능을 붙여주는 콜백들의 리스트를 전달하는 파라미터
	* `rememberSavedStateNavEntryDecorator()`
		* 각 네비게이션 엔트리가 화면에 “들어올(entry)” 때마다 `SavedStateRegistryOwner` 를 제공
		- Compose 에서 `rememberSaveable { … }` 같은 API 를 쓸 때, 뒤로 갔다가 다시 들어오는 과정에서도 상태를 곧바로 복원(restoration)할 수 있게 된다.
	- `rememberViewModelStoreNavEntryDecorator()`
		- 각 네비게이션 엔트리에 `ViewModelStoreOwner` 를 붙임
		- 엔트리별로 ViewModel을 생성하고, 해당 화면이 사라졌다가 다시 돌아올 때 같은 ViewModel 인스턴스를 재사용할 수 있음
- `transitionSpec` : UI
- `popTransitionSpec` : UI
- `entryProvider` : `NavDisplay`에게 “어떤 `NavigationRoute` 타입이 왔을 때, 그걸 실제로 어떻게 화면에 그릴지” 알려주는 **팩토리(Factory)** 역할을 함
	- `entry<T> { entry -> ... }` 호출 : 만약 `backStack`에 `T` 타입의 `NavigationRoute`가 있으면, 이 람다를 실행해서 화면을 그려라
		- 람다의 파라미터 `entry` 의 타입은 `T` 이다
	- `entry<Home>`
		- [[feature-home-HomeScreen.kt#fun HomeScreen| HomeScreen]]
	- `entry<Camera>`
		- [[feature-camera-base-CameraScreen.kt#fun CameraPreviewScreen|CameraPreviewScreen]]
		- `backStack.removeAll { it is Create }`
			- 만약 스택에 [[app-navigation-NavigationRoutes.kt#data class Create|Create]] 화면이 있다면 제거, 중복 화면 방지를 위해 먼저 제거하는 처리
		- `backStack.add(Create(uri.toString()))`
			- 새로 찍은 사진의 `uri`를 문자열로 바꿔서 `Create` 화면으로 전달
				- ex) `Create(fileName = "content://...")`
		- `backStack.removeAll { it is Camera }`
			- 현재 카메라 화면을 스택에서 제거, 스택에는 `Create` 화면만 남고 Camera는 완전히 종료됨
	- `entry<Create>`
		- TODO
	- `entry<About>`
		- 