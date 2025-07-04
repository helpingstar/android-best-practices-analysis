
## fun CameraPreviewScreen
```kotlin
@OptIn(  
    ExperimentalPermissionsApi::class,  
    ExperimentalSharedTransitionApi::class,  
    ExperimentalMaterial3ExpressiveApi::class,  
)  
@Composable  
fun CameraPreviewScreen(  
    modifier: Modifier = Modifier,  
    viewModel: CameraViewModel = hiltViewModel(),  
    onImageCaptured: (Uri) -> Unit,  
)
```

```kotlin
val sharedTransitionScope = LocalSharedTransitionScope.current
```
* `LocalSharedTransitionScope.current` 를 통해 현재 컴포넌트 트리에서 사용 가능한 SharedTransitionScope를 가져온다.
	* 이는 Compose의 CompositionLocal을 통해 제공되는 공유 전환 애니메이션을 관리하는 스코프이다.
	* `current`를 호출함으로써 현재 Composition 트리에서 상위로부터 제공된 `SharedTransitionScope` 객체를 가져온다.
```kotlin
with(sharedTransitionScope) {...}
```
* 블록 내에서 공유 전환 애니메이션을 사용할 수 있다.
* 이 블록 내의 컴포넌트들은 다른 화면과의 전환 시 애니메이션 효과를 적용받는다.

* TODO
	* [[core-theme-base-SharedElementsConfig.kt#fun Modifier.sharedBoundsRevealWithShapeMorph|sharedBoundsRevealWithShapeMorph]]
		* `SharedElementKey.CameraButtonToFullScreenCamera`
	* `SharedElementKey.CameraButtonToFullScreenCamera`
	* [[core-theme-base-SharedElementsConfig.kt#fun Modifier.sharedBoundsWithDefaults|sharedBoundsWithDefaults]]
		* `SharedElementKey.CaptureImageToDetails`

```kotlin
val cameraPermissionState = rememberPermissionState(Manifest.permission.CAMERA)
```
* Accompanist Permissions 라이브러리를 사용하여, 카메라 권한 상태를 추적하고 UI와 권한 로직을 연결
### cameraPermissionState.status.isGranted == true

```kotlin
val uiState by viewModel.uiState.collectAsStateWithLifecycle()  
```
* `viewModel.uiState` : `StateFlow<CameraUiState>` 타입의 상태 값으로 ViewModel이 관리하는 카메라 UI State
* `.collectAsStateWithLifecycle()` : Compose가 상태를 관


```kotlin
val activity = LocalActivity.current as ComponentActivity
```

```kotlin
val foldingFeature by viewModel.foldingFeature.collectAsState()
```

```kotlin
LaunchedEffect(uiState.imageUri) {  
    // this is used to navigate to the next screen, when an image is captured, it signals to NavController that we should do something with the URI,  
    // once the signal has been sent, the value is set to null to not trigger navigation over and over again.    val uri = uiState.imageUri  
    if (uri != null) {  
        onImageCaptured(uri)  
        viewModel.setCapturedImage(null)  
    }  
}
```

- `imageUri` 값이 **변경될 때마다 실행되는 effect 블록**
- **이미지가 새로 캡처되었을 때만 실행**
- `LaunchedEffect` : `CoroutineScope` 안에서 작동하는 side-effect 처리 도구

```kotlin
val scope = rememberCoroutineScope()  
```
* 현재 컴포지션의 수명에 맞춰 유지되는 코루틴 스코프를 생성
	* **현재 컴포지션** : 특정 `@Composable` 함수가 **실행되고 UI 트리를 구성하는 과정**과 **그 상태를 관리하는 영역**
* 이 스코프는 화면이 컴포즈되는 동안 살아있고, 컴포지션이 사라지면 자동으로 취소
```kotlin
LifecycleStartEffect(viewModel) {  
    val job = scope.launch { viewModel.bindToCamera() }  
    onStopOrDispose { job.cancel() }  
}
```
* `LifecycleStartEffect`
	* Android Compose의 생명주기 인식 API
	* `CameraPreviewScreen` Composable 이 속한 `LifecycleOwner` (즉, Activity나 Fragment)의 Lifecycle이 다른 상태였다가 `STARTED` 상태 이상이 될 때만 블록 내의 코드(effect)를 실행
		* `STARTED` 상태 이상(STARTED state or above) : Lifecycle 상태가 `STARTED` 또는 `RESUMED` 상태에 있음(`PAUSED` 포함, 내부적으로 `STARTED` 상태)
		* `PAUSED` -> `RESUMED` : 실행 안 됨 : 상태는 변했지만 STARTED 이상을 계쏙 유지하고 있어서 다시 실행되지 않음
	* `viewModel` : 이 effect를 다시 실행할 지 판단할 때 쓰는 키
		* key가 변경되면 기존 effect는 `onStopOrDispose`로 정리된다. 하지만 정리할 작업이 있다면 `onStopOrDispose` 로 명시적으로 등록해야 한다.
	* `val job = scope.launch { viewModel.bindToCamera() }`
		* `rememberCoroutineScope()`로 만든 수명에 묶인 `CoroutineScope`에서 코루틴을 실행한다.
			* `CoroutineScope` : Kotlin Coroutine에서 Coroutine이 실행되는 범위를 의미, 코루틴의 수명(lifecycle)과 취소(cancle)를 관리하는 컨테이너이다.
		* [[feature-camera-base-CameraViewModel.kt#suspend fun bindToCamera|bindToCamera()]] 는 `suspend` 함수이고 `LifecycleStartEffect` 안에서 `suspend` 함수를 `launch` 로 호출한다 -> 즉 비동기로 실행한다.
	* `onStopOrDispose { job.cancel() }` : Lifecycle이 STOPPED 이하로 내려가거나 key가 바뀌는 경우에 실행

```kotlin
LaunchedEffect(Unit) {  
    viewModel.calculateFoldingFeature(activity)  
    viewModel.initRearDisplayFeature(activity)  
}
```

`fun LaunchedEffect(key1: Any?, block: suspend CoroutineScope.() -> Unit): Unit`
* `LaunchedEffect`가 **컴포지션에 들어오면**, 주어진 `block`을 해당 컴포지션의 `CoroutineContext`에서 실행  
* `key1`이 변경되어 recomposition(재컴포지션)이 발생하면,  실행 중이던 코루틴은 **자동으로 취소되고**, 새로 **재실행**
* `LaunchedEffect`가 컴포지션에서 **사라지면**, 코루틴도 **취소**

TODO : 블록 안의 내용은 폴더블 핸드폰 디스플레이 컨트롤에 관한 내용
* [[feature-camera-base-CameraViewModel.kt#fun calculateFoldingFeature|calculateFoldingFeature()]]
* [[feature-camera-base-CameraViewModel.kt#fun initRearDisplayFeature|initRearDisplayFeature()]]

```kotlin
uiState.surfaceRequest?.let { surface ->  
    CameraPreviewContent(  
        modifier = Modifier.fillMaxSize(),  
        surfaceRequest = surface,  
        autofocusUiState = uiState.autofocusUiState,  
        tapToFocus = viewModel::tapToFocus,  
        detectedPose = uiState.detectedPose,  
        defaultZoomOptions = uiState.zoomOptions,  
        requestFlipCamera = viewModel::flipCameraDirection,  
        canFlipCamera = uiState.canFlipCamera,  
        requestCaptureImage = viewModel::captureImage,  
        zoomRange = uiState.zoomMinRatio..uiState.zoomMaxRatio,  
        zoomLevel = { uiState.zoomLevel },  
        onChangeZoomLevel = viewModel::setZoomLevel,  
        foldingFeature = foldingFeature,  
        shouldShowRearCameraFeature = viewModel::shouldShowRearDisplayFeature,  
        toggleRearCameraFeature = { viewModel.toggleRearDisplayFeature(activity) },  
        isRearCameraEnabled = uiState.isRearCameraActive,  
        cameraSessionId = uiState.cameraSessionId,  
    )  
}
```

* `reuqestCaptureImage` : [[feature-camera-base-CameraViewModel.kt#fun captureImage|viewModel::captureImage]]

### 파라미터
#### 

## private fun CameraPermissionGrant

```kotlin
@Composable  
private fun CameraPermissionGrant(  
    launchPermissionRequest: () -> Unit,  
    showRationale: Boolean,  
    modifier: Modifier = Modifier,  
)
```


## fun StatelessCameraPreviewContent

```kotlin
@Composable  
fun StatelessCameraPreviewContent(  
    viewfinder: @Composable (Modifier) -> Unit,  
    canFlipCamera: Boolean,  
    requestFlipCamera: () -> Unit,  
    detectedPose: Boolean,  
    defaultZoomOptions: List<Float>,  
    zoomLevel: () -> Float,  
    onAnimateZoom: (Float) -> Unit,  
    requestCaptureImage: () -> Unit,  
    modifier: Modifier = Modifier,  
    foldingFeature: FoldingFeature? = null,  
    shouldShowRearCameraFeature: () -> Boolean = { false },  
    toggleRearCameraFeature: () -> Unit = {},  
    isRearCameraEnabled: Boolean = false,  
)
```


## private fun CameraPreviewContent
```kotlin
@Composable  
private fun CameraPreviewContent(  
    surfaceRequest: SurfaceRequest,  
    autofocusUiState: AutofocusUiState,  
    tapToFocus: (Offset) -> Unit,  
    cameraSessionId: Int,  
    canFlipCamera: Boolean,  
    requestFlipCamera: () -> Unit,  
    detectedPose: Boolean,  
    defaultZoomOptions: List<Float>,  
    zoomRange: ClosedFloatingPointRange<Float>,  
    zoomLevel: () -> Float,  
    onChangeZoomLevel: (zoomLevel: Float) -> Unit,  
    requestCaptureImage: () -> Unit,  
    modifier: Modifier = Modifier,  
    foldingFeature: FoldingFeature? = null,  
    shouldShowRearCameraFeature: () -> Boolean = { false },  
    toggleRearCameraFeature: () -> Unit = {},  
    isRearCameraEnabled: Boolean = false,  
)
```