## class CameraViewModel

```kotlin
@HiltViewModel  
class CameraViewModel  
@Inject constructor(  
    application: Application,  
    val localFileProvider: LocalFileProvider,  
    val rearCameraUseCase: RearCameraUseCase,  
) : AndroidViewModel(application)
```

```kotlin
private var _uiState = MutableStateFlow(CameraUiState())  
```

* [[#data class CameraUiState|CameraUiState()]]는 UI 상태를 나타내는 데이터 클래스의 기본값으로 초기화됨
* `_uiState` 내부에서만 상태를 변경할 수 있도록 `private` 로 선언되어 있다.
* `_uiState.update {...} ` 등의 방식으로 상태를 수정

```kotlin
val uiState: StateFlow<CameraUiState>  
    get() = _uiState
```

* 외부(ex. UI Composable, Activity, Fragment)에서 접근 가능한 Read-only `StateFlow`
* `get() = _uiState`를 통해 `_uiState`의 값을 노출, 외부에서는 값 변경 불가
- 이를 통해 **단방향 데이터 흐름(Unidirectional Data Flow)** 을 유지합니다.

```kotlin
private var _foldingFeature = MutableStateFlow<FoldingFeature?>(null)  
val foldingFeature: StateFlow<FoldingFeature?>  
    get() = _foldingFeature
```
* **폴더블(접이식) 기기**에서 **화면 접힘 상태(FoldingFeature)** 를 감지하고 UI에 반영하기 위해 사용

```kotlin
private var surfaceMeteringPointFactory: SurfaceOrientedMeteringPointFactory? = null
```
* 터치로 초점을 맞추는 포인트(Focus Point)를 계산해주는 메타 포인트 생성기
* 앱이 `(0, 0)`에서 `(width, height)` 범위 내에서 `createPoint(float, float)`를 통해 측광 포인트(metering point)를 지정할 수 있는 영역을 정의
* 너비와 높이를 `1.0`로 설정하면, **정규화된 좌표(normalized coordinates)** 를 이용해 포인트를 생성

```kotlin
private var cameraControl: CameraControl? = null
```
* 현재 바인딩된 카메라의 **제어 기능(control)** 을 담당
- 이 객체를 통해 **줌, 초점, 노출, 플래시** 등을 설정
- 카메라가 바인딩 되기 전에 `cameraControl` 객체가 존재하지 않기 떄문에 nullable

```kotlin
private var cameraInfo: CameraInfo? = null
```
* 현재 바인딩된 카메라의 **정보(info)** 를 읽어올 때 사용
- 설정 변경은 할 수 없고, **읽기 전용(read-only)**

```kotlin
private val cameraPreviewUseCase = Preview.Builder().build().apply {  
    setSurfaceProvider { newSurfaceRequest ->  
        _uiState.update { it.copy(surfaceRequest = newSurfaceRequest) }  
        surfaceMeteringPointFactory = SurfaceOrientedMeteringPointFactory(  
            newSurfaceRequest.resolution.width.toFloat(),  
            newSurfaceRequest.resolution.height.toFloat(),  
        )  
    }  
}
```
* `Preview.Builder().build()` : `Preview` 객체를 생성
* `.apply {... }` : 해당 Preview 객체를 바로 구성
	* Preview 객체를 생성하자마자 설정도 같이 하기 위함
	* `apply` 블록 안에서는 생성한 객체 (`Preview`)를 `this` 로 간주하고 설정을 쉽게할 수 있음
* `setSurfaceProvider { newSurfaceRequest -> ... }`
	* `setSurfaceProvider`는 다음과 같은 상황에서 호출된다.
		* 카메라가 처음 바인딩
		* 카메라의 전면/후면을 전환
		* 화면이 회전하면서 프리뷰 해상도나 방향이 변경
		* Preview UseCase를 새로 생성하거나 재바인딩
		* 앱이 백그라운드 갔다가 다시 돌아오면서 Preview가 재요청됨
	* 이렇게 호출될 때 새로운 SurfaceReuqest 인스턴스가 전달된다.
	* SurfaceReuqest를 UI 쪽으로 전달
	* CameraX는 내부적으로 `SurfaceProvider` 를 통해 미리보기를 출력할 Surface를 요청한다.
	* `newSurfaceRequest` : 카메라 미리보기를 그릴 Surface 요청 정보

```kotlin
private val cameraCaptureUseCase = ImageCapture.Builder().build()
```
- 객체만 생성하면 되고, 설정은 필요 없음
```kotlin
private val cameraImageAnalysisUseCase = ImageAnalysis.Builder()  
    .setBackpressureStrategy(ImageAnalysis.STRATEGY_KEEP_ONLY_LATEST)  
    .build()
```
* `setBackpressureStrategy(ImageAnalysis.STRATEGY_KEEP_ONLY_LATEST)` : 지금 들어오는 프레임 중 가장 최신 것만 분석하라고 요청하는 것
* Analaysis 분석 코드가 느리면 프레임이 쌓여 분석 지연이 발생할 수 있기 때문에 이전 프레임들을 버리고 가장 최근 프레임만 전달하라고 요청하는 것이다.

```kotlin
private val cameraUseCaseGroup = UseCaseGroup.Builder()  
    .addUseCase(cameraPreviewUseCase)  
    .addUseCase(cameraCaptureUseCase)  
    .addUseCase(cameraImageAnalysisUseCase)  
    .build()
```
* `UseCaseGroup`은 CameraX에서 **여러 UseCase(Preview, ImageCapture, ImageAnalysis 등)를 묶어서 바인딩**할 수 있도록 도와주는 객체
	* `androidx.camera.core.UseCaseGroup`

```kotlin
private val cameraTypeFlow = MutableStateFlow<CameraSelector?>(null)
```
현재 선택된 **카메라 렌즈(전면 또는 후면)** 정보를 `StateFlow` 형태로 보관하며, 앱이 카메라 방향을 바꿀 수 있게 만듬
```kotlin
private var mediaActionSound: MediaActionSound? = null
```

```kotlin
private var autofocusRequestId: Int = 0
```

### suspend fun bindToCamera

```kotlin
suspend fun bindToCamera() = suspendCancellableCoroutine { cont ->
	val job = viewModelScope.launch { ... }  
	  
	job.invokeOnCompletion { cause ->  
	    if (cause == null) {  
	        cont.resume(Unit)  
	    } else {  
	        cont.resumeWithException(cause)  
	    }  
	}  
	  
	cont.invokeOnCancellation { job.cancel() }
}
```
* `suspendCallableCoroutine {...}` : TODO
* `val job = viewModelScope.launch { ... }`
	* `viewModelScope` : `ViewModel`의 생명주기에 따라 자동으로 관리되는 CoroutineScope
	* 이 Scope에서 시작된 코루틴은 **ViewModel이 `onCleared()` 될 때 자동으로 취소**
* `launch`
	- `launch`는 코루틴을 **백그라운드에서 비동기적으로 실행**합니다.
	- 반환값이 `Job`이므로, 나중에 **작업을 취소하거나 완료 상태를 확인**할 수 있습니다.
* `job.invokeOnCompletion { cause -> if (cause == null) {...} ... else { ... } }`
	* 코루틴을 `launch`로 실행하면 `Job` 객체가 반환
	* 코루틴이 **정상 종료되면 `cont.resume(Unit)`**,  예외가 발생하면 `resumeWithException(...)`을 호출해서   `suspendCancellableCoroutine`을 호출한 쪽에 **정확한 상태를 전달**
		* `cont.resume(...)` 또는 `cont.resumeWithException(...)` 을 호출하면, `suspendCancellableCoroutine`을 호출한 지점으로 값이나 예외가 반환
* `cont.invokeOnCancellation { job.cancel() }`
	* 상위 코루틴이 취소되면 `job.cancel()`을 호출해 하위 작업들도 중단시킨다.
		* ex) 카메라 바인딩 중 앱이 닫히거나 화면이 바뀌는 경우 리소스를 낭비하지 않도록 즉시 취소

```kotlin
launch { runPoseDetection() }
```
* [[#private suspend fun runPoseDetection|runPoseDetection()]]을 **코루틴으로 비동기 실행**하여,   **카메라 영상에서 사람의 자세를 실시간으로 분석**하는 작업을 **메인 작업과 병렬로 수행**
	* `runPoseDetection()` 은 `suspend` 함수이기 때문에, 코루틴 안에서만 호출 가능
	* `launch` 로 감싸지 않으면, [[#suspend fun bindToCamera|bindToCamera()]] 가 [[#private suspend fun runPoseDetection|runPoseDetection()]] 이 끝날 때까지 블로킹되므로 다음과 같은 문제가 발생할 수 있음
		* 카메라 바인딩 지연
		* UI 응답성 저하
	* `launch { ... }` 는 병렬로 실행되며, `bindToCamera()`의 다른 작업을 방해하지 않음


```kotlin
val processCameraProvider = ProcessCameraProvider.awaitInstance(application)
```
* CameraX의 **카메라 서비스 제공자(CameraProvider)** 인스턴스를 **비동기적으로 가져오는 함수**
* `ProcessCameraProvider` : 내부적으로 장치에 있는 카메라를 감지하고, 사용 가능한 카메라 목록을 확인하고, Preview, ImageCapture 등의 **UseCase**를 바인딩할 수 있게 한다.


```kotlin
val availableCameraLenses =  
    listOf(  
        DEFAULT_BACK_CAMERA,  
        DEFAULT_FRONT_CAMERA,  
    ).filter {  
        processCameraProvider.hasCamera(it)  
    }
```
* **기기에서 사용할 수 있는 카메라(후면/전면)를 확인**해서, 실제로 사용할 수 있는 렌즈만 리스트로 만드

```kotlin
_uiState.update { it.copy(canFlipCamera = availableCameraLenses.size == 2) }
```
* 카메라가 **전면 + 후면 둘 다 있는 경우에만** true로 설정
```kotlin
cameraTypeFlow.update { it ?: availableCameraLenses.firstOrNull() }
```
* `cameraTypeFlow` : 현재 선택된 **카메라 렌즈(전면 또는 후면)** 정보
	* `.update { ... }`
		* `StateFlow` 의 값을 업데이트할 때 사용하는 atomic 한 방식의 함수
		* atomic 하다는 것? : 여러 코루틴이 동시에 값을 변경하려 해도 충돌 없이 안전하게 단일 작업으로 처리된다
			* `flow.value = flow.value + 1`
				* 위험 (두 스레드가 동시에 하면 충돌 가능)
			* `flow.update { it + 1 }`
				* 안전 (atomic 하게 처리됨)
			* `update { ... }`는 Kotlin Flow 내부에서 **lock-free CAS(Compare-And-Set)** 알고리즘으로 처리되기 때문에,  **경쟁 상태(race condition) 없이 값이 일관되게 유지**
	* `.update { it ?: availableCameraLenses.firstOrNull() }`
		* 현재 선택된 카메라(`cameraTypeFlow.value`) 가 아직 설정되지 않았다면, 사용 가능한 카메라 목록 중 첫 번째를 기본값으로 설정하라
		* 현재 코드에서는 두 카메라가 모두 사용 가능할 경우 `DEFAULT_BACK_CAMERA` 가 우선한다.
		* 
```kotlin
mediaActionSound = MediaActionSound()
```
* **카메라 셔터 소리나 포커스 완료 소리 등 시스템 제공 미디어 사운드를 재생**하기 위한 Android 클래스*

```kotlin
cameraTypeFlow
	.onCompletion { ... }
	.collectLastest { cameraType -> ... }
```

* `cameraTypeFlow` : 현재 선택된 카메라(전면 / 후면 등)를 나타내는 상태 흐름
	* `.onCompletion` : Flow 가 정상적으로 종료되거나 취소될 때 호출되는 리스너 (finally랑 같은 역할)
		* 여기서는 카메라 세션이 끝나면 (예: 상위 `job` 취소) 리소스 해제를 한다.
	* `.collectLastest` : 가장 최근에 `emit`된 값만 처리하고, 이전 처리중이라면 취소 후 다시 실행하는 연산자

```kotlin
cameraTypeFlow
	.onCompletion {  
	    mediaActionSound?.release()  
	    mediaActionSound = null  
	    cameraControl = null  
	    cameraInfo = null  
}
```
* 카메라 세션 종료 시점에 리소스(사운드, 제어 핸들 등)를 안전하게 해제하고 메모리 정리를 수행

```kotlin
cameraTypeFlow
	.collectLatest { cameraType -> 
		...
	}
```
* 상위에 정의된 F`Flow<CameraSelector?>` (예: 사용자가 앞,뒤 카메라를 전환할 때마다 emit 되는 렌즈 선택 값)를 `collectLatest` 로 구독한다.
* `collectLatest`는 새로운 값이 들어올 때마다(여기서는 `cameraType` 이 바뀔 때마다) **이전 콜백**을 취소하고, 새 콜백을 실행한다.
* 즉, 사용자가 빠르게 앞/뒤 카메라를 토글하더라도 항상 가장 마지막 상태만 처리된다.


| 구분      | Flow                    | StateFlow          | MutableStateFlow       |
| ------- | ----------------------- | ------------------ | ---------------------- |
| 생성 시점   | Cold: collect 시점에 실행    | Hot: 초기값 지정, 즉시 실행 | Hot: 초기값 지정, 즉시 실행     |
| 값 보관    | 없음                      | 최근 1개              | 최근 1개                  |
| 읽기/쓰기   | 읽기 전용(emit은 flow 블록 내부) | 읽기 전용              | 읽기·쓰기                  |
| 구독 시 동작 | 매 collect마다 재실행         | 최초/재구독 시 바로 최신값 전달 | 최초/재구독 시 바로 최신값 전달     |
| 주 사용처   | 일회성 비동기 데이터 처리          | 앱 상태(state) 저장·공유  | ViewModel 등에서 상태 갱신·관리 |


```kotlin
autofocusRequestId++
```
* 내부에서 "이전에 요청한 자동초점 작업"을 구별하기 위한 식별자
* 값이 바뀌면(카메라 타입이 바뀌면) `autofocusRequestId`를 1 증가시켜 "이전에 진행중이던 자동초점 콜백이 완료되더라도 무시" 하도록 한다.

```kotlin
_uiState.update { it.copy(autofocusUiState = AutofocusUiState.Unspecified) }
```
* `CameraUiState`에 들어있는 `autofocusUiState` 값을 `Unspecified`(초점 상태 미정)로 초기화
* TODO : uiState에서 이미 초기화 하는데 bind 할 때 또 초기화하는 이유

```kotlin
if (cameraType != null)
```
`cameraType` : 현재 선택된 카메라(전면 / 후면 등)를 나타내는 상태 흐름
* `cameraTypeFlow.collectLatest {cameraType -> ...}` 를 통해 상위의 `MutableStateFlow<CameraSelector?>`를 구독한다.

```kotlin
processCameraProvider.runWith(cameraType, cameraUseCaseGroup) { camera -> 
	... 
}
```

- `ProcessCameraProvider`의 확장 함수 [[feature-camera-base-CoroutineCameraProvider.kt#suspend fun <R > ProcessCameraProvider.runWith|ProcessCameraProvider.runWith]]을 호출해,
    1. **해당 렌즈(selector)** 와
    2. **Preview·Capture·Analysis 같은 Use-Case들의 모임(useCaseGroup)**
    을 라이프사이클에 바인딩(bind)한 뒤, 바인딩된 `Camera` 객체를 람다 인자로 넘겨받는다.

```kotlin
cameraControl = camera.cameraControl
cameraInfo    = camera.cameraInfo
```
- `CameraControl`: 줌·초점·노출 같은 런타임 제어 API
- `CameraInfo` : 현재 줌 레벨·지원 범위 등 상태 조회 API
* ViewModel Property에 각각 할당해 다른 메서드 (`tabToFocus`, `setZoomLevel` 등)에서 사용할 수 있게 한다.
	* ViewModel Property
		* `private var cameraControl: CameraControl? = null`
		* `private var cameraInfo: CameraInfo? = null`

```kotlin
_uiState.update { it.copy(cameraSessionId = it.cameraSessionId + 1) }
```
* 내부 `cameraSessionId`를 1 증가시켜, Compose나 View 측에서 “새로운 Preview Surface 생성” 같은 갱신 로직을 트리거합니다.
	* `cameraSessionId`를 증분(+=1)하는 이유는 “View 계층에서 카메라 Preview를 완전히 새로 그려야 한다”는 신호를 주기 위함
	* `cameraType`(앞·뒤 카메라 선택)만 바뀌었다고 해서 Compose나 View쪽에서 반드시 **PreviewView**(또는 Surface) 전체를 재생성하지는 않음
	* 이미 화면에 붙어 있는 PreviewView는 내부 SurfaceProvider가 바뀌지 않는 한 "계속 같은 Surface"에서 영상만 교체하려 할 수 있다. 이를 방지하려면 "완전히 새로운 세션" 임을 알려주는 별도의 key 가 필요한데 이 역할을 `cameraSessionId`가 담당한다.
		* 위 문장에서의 '영상' : **실시간 카메라 프리뷰 스트림**(즉, PreviewView 위에 렌더링되는 개별 프레임)

```kotlin
camera.cameraInfo.zoomState.asFlow().collectLatest { zoomState -> ... }
```
* `camera.cameraInfo.zoomState`
	* `LiveData<ZoomState>` 타입으로 현재 줌 상태를 제공한다
* `asFlow()`
	* `LiveData`를 Kotlin Flow로 변환해준다
* `.collectLatest { zoomState -> … }`
	* Flow의 최신 값만 처리하되, 새 값이 들어오면 이전 처리 블록을 취소하고 다시 실행
	* 이 호출 지점에서 코루틴은 종료되지 않고 계속 실행되며, 줌 상태가 바뀔 때마다 람다 블록이 반복 호출된다.
		* `collectLatest { ... }` 자체가 종료되지 않는 반복문이다.
		* 한 번 호출하면 Flow가 끝나거나 코루틴이 최소될 때까지 계속 살아 있다.
		* 내부적으로는 Flow에서 값이 들어올 때마다(여기서는 zoom 상태가 바뀔 때마다) callback(lambda)을 호출하는 형태입니다.

```kotlin
{ zoomState -> 
	_uiState.update {  
	    it.copy(  
	        zoomLevel = zoomState.zoomRatio,  
	        zoomMinRatio = zoomState.minZoomRatio,  
	        zoomMaxRatio = zoomState.maxZoomRatio,  
	    )  
	}
}
```
* 카메라의 현재 줌 상태(zoomState)를 읽어서, 그 값을 화면에 그려줄 수 있도록 `CameraUiState` 에 반영한다.
	* `zoomState.zoomRatio` : 지금 실제 카메라가 적용중인 줌 배율
	* `zoomState.minZoomRatio` : 이 카메라(렌즈)가 지원하는 최소 줌 배율
		* 보통 1f(확대 없음)이거나, 광각 렌즈의 경우 0.6f같은 1배 미만도 가능
	* `zoomState.maxZoomRatio` : 이 카메라가 지원하는 최대 줌 배율


### fun captureImage
```kotlin
fun captureImage()
```
사용자가 촬영 버튼을 눌렀을 때 **사진을 비동기적으로 캡처**하여 파일로 저장하고, 결과 URI를 ViewModel의 UI 상태에 반영하는 역할

```kotlin
viewModelScope.launch {...}
```

* ViewModel에 묶인 코루틴 스코프를 사용해 비동기 작업을 시작한다.
* 이 안에서 모든 I/O와 CameraX 호출이 논블로킹으로 실행된다.

```kotlin
mediaActionSound?.play(MediaActionSound.SHUTTER_CLICK)
```

촬영 시 셔터 음을 재생하기 위한 코드

```kotlin
val file = localFileProvider.getFileFromCache("image${System.currentTimeMillis()}.jpg")
```

* 앱의 캐시 디렉토리에 고유한 파일 이름 `image${System.currentTimeMillis()}.jpg` 으로 `File` 객체를 얻는다.
* 외부 저장소 권한 없이 내부 캐시에 안전하게 이미지를 저장할 수 있다.
* 다음과 같이 저장된다.
* `file:///data/user/0/com.android.developers.androidify/cache/image1751544963723.jpg`
	* `localFileProvider.getFileFromCache(...)` 메서드는,  일반적으로 내부 구현에서 다음과 같은 경로를 반환
		* `File(context.cacheDir, fileName)`
		* `/data/user/0/[패키지명]/cache/[파일명]`
		* 이 경로는 앱 내부 저장소 중 "캐시 디렉터리"에 해당한다.
		* 이 위치에 저장된 파일은 앱이 삭제되면 같이 삭제되며, 안드로이드 시스템이 필요시 공간 확보를 위해 자동 삭제할 수도 있다.
	* `file://` URI는 보안 제한 때문에 Android 7.0 (API 24) 이상에서는 외부 앱에 공유할 수 없다.
	* 공유하려면 반드시 `FileProvider` 를 통해 `content://` 형태의 URI로 변환해야 함

```kotlin
val outputFileOptions = OutputFileOptions.Builder(file).apply {  
    if (cameraTypeFlow.value == DEFAULT_FRONT_CAMERA) {  
        val metadata = ImageCapture.Metadata().apply {  
            // Ensure the captured front camera image is mirrored to match preview  
            isReversedHorizontal = true  
        }  
        setMetadata(metadata)  
    }  
}.build()
```

* `OutputFileOptions`
	* `ImageCapture`에 넘길 출력 옵션을 만든다.
	* 전면 카메라(`DEFAULT_FRONT_CAMERA`)인 경우, `ImageCapture.Metadata.isReversedHorizontal = true` 로 설정하여 Preview와 같은 방향으로(좌우 반전된 상태로) 저장되도록 한다.
	* `setMetaData(metadata)` : `OutputFileOptions.Builder`에 메타데이터를 전달하여, **이미지 저장 시 이 정보를 반영**하게 함
	* `build()`를 호출하는 이유
		* `Builder`는 단순히 설정을 저장해두는 중간 객체일 뿐이다.
		* `build()`를 호출하지 않으면 `ImageCapture.takePicture(...)` 에 전달할 수 있는 완성된 `OutputFileOptions` 객체가 만들어지지 않음.
* Builder Pattern을 사용하는 이유
	* 복잡한 객체 생성을 단순화
		* 어떤 객체는 생성할 때 설정해야 할 항목이 많거나, 조건에 따라 달라질 수 있음. 이런 경우 생성자에 많은 파라미터를 넣는 대신, `Builder`를 통해 **단계적으로 설정**하고 **최종적으로 객체를 생성**할 수 있음
	* 가독성 향상
		* 생성자에 매개변수가 많으면 가독성이 나빠진다. 하지만 Builder를 쓰면 더 **명시적이고 읽기 쉬운 코드**가 된다.
	* 불변 객체(Immutable Object)와 궁합이 좋음
		* 설정이 끝난 뒤에는 객체를 변경하지 않도록 만들 수 있어 **안전하고 일관성 있는 코드** 작성에 도움이 됨
	* 선택적 매개변수 지원
		* 설정하고 싶은 속성만 설정하고 나머지는 기본값으로 둘 수 있어 생성자 오버로딩 없이 유연한 구성이 가능함

```kotlin
try {  
    // Use the suspend version of takePicture() to get the result  
    val outputFileResults = cameraCaptureUseCase.takePicture(outputFileOptions)  
    Log.d("CameraViewModel", "Image captured: ${outputFileResults.savedUri}")  
    _uiState.update { it.copy(imageUri = outputFileResults.savedUri) }  
} catch (exception: ImageCaptureException) {  
    Log.e("CameraViewModel", "Error capturing image $exception")  
    // TODO handle error on screen  
}
```

```kotlin
// Use the suspend version of takePicture() to get the result  
val outputFileResults = cameraCaptureUseCase.takePicture(outputFileOptions)
```
* `cameraCaptureUseCase` : `ImageCapture` 인스턴스, `ImageCapture.Builder().build()` 로 생성
* `takePicture(outputFileOptions)`
	* Kotlin에서 제공하는 suspend 함수 확장 버전
		* `suspend`이기 때문에 `ImageCaptureException`이 발생할 수 있음
			* ex) 저장 실패, I/O 오류, 저장 권한 문제, 저장 위치 접근 실패
			* 따라서 이 코드는 `try/catch` 로 감싸는 것이 일반적이다
	* input : `outputFileOptions` (이미지를 저장할 위치 및 메타데이터 포함)
	* output : `ImageCapture.OutputFileResults` (결과 객체, 저장된 URI 등 포함)
* `outputFileResults.savedUri` 예시 :  `file:///data/user/0/com.android.developers.androidify/cache/image1751544963723.jpg`

### fun tapToFocus

화면을 터치한 좌표를 기준으로 **카메라의 포커스를 이동**시키고, 그 결과를 UI 상태로 업데이트하는 함수

```kotlin
fun tapToFocus(tapCoordinates: Offset)
```

* `Offset` : Jetpack Compose에서 사용하는 좌표 표현용 클래스, UI 상의 터치 위치, 뷰 내부 좌표, 드로잉 위치, 애니메이션 위치 등에서 자주 사용된다.
	- `x: float` : X축 좌표 (왼쪽 → 오른쪽)
	- `y: float` : Y축 좌표 (위 → 아래)

```kotlin
viewModelScope.launch {...}
```
* `viewModelScope` : `ViewModel` 클래스에서 제공되는 CoroutineScope
	* ViewModel이 생성되면 자동으로 만들어지고 ViewModel이 `onCleared` 될 때 자동으로 cancel 된다.
* `.launch { ... }` : 코루틴을 백그라운드에서 재생시킨다.
	* `launch`는 `Job`을 반환하며, 내부에서 실행 중인 작업을 추적하거나 취소할 수 있다.
	* Job을 저장하지 않는 경우 (fire-and-forget)
		*  작업을 **그냥 실행만 하고 잊어버릴 때**.
		- 성공/실패 여부 확인, 취소, 진행 상황 추적 등이 필요 없을 때.
	    - 대표적인 용도: 로그 전송, 토스트 띄우기, 단순 UI 갱신 등
	* `Job`을 저장하는 경우 (작업 관리 목적)
		* 취소할 수 있게 이전 작업 중복 실행 방지, `job.cancel()`
		* 상태 확인 `job.isActive`, `job.isCompleted`, `job.isCancelled`
		* 나중에 기다리기 `job.join()` 으로 종료될 때까지 기다릴 수 있음
		* 여러 작업 동기화 여러 job을 모아서 처리하거나 취소 그룹 관리할 때

```kotlin
val requestId = ++autofocusRequestId
```
* 새로운 오토포커스 요청을 시작하면서 `autofocusRequestId`를 증가시키고, 그 값을 `requestId` 변수에 저장합니다. 이 `requestId`는 **이 코루틴 블록이 시작될 당시의 요청 번호**입니다.
* auto-focus 완료 했을 때도 이 두 값이 같다면 그 사이에 사용자가 또 화면을 탭했다면 `autofocusRequestid` 는 증가해있을 것이고 변하지 않은 `requestId` 와 다른 값을 가지게 될 것이다.
* 

### fun setZoomLevel

```kotlin
fun setZoomLevel(zoomLevel: Float)
```

### fun setCapturedImage

```kotlin
fun setCapturedImage(uri: Uri?)
```

### fun flipCameraDirection

```kotlin
fun flipCameraDirection()
```

### fun initRearDisplayFeature

```kotlin
fun initRearDisplayFeature(activity: ComponentActivity)
```

### fun shouldShowRearDisplayFeature

```kotlin
fun shouldShowRearDisplayFeature()
```

### fun toggleRearDisplayFeature

```kotlin
fun toggleRearDisplayFeature(activity: ComponentActivity)
```

### fun calculateFoldingFeature

```kotlin
fun calculateFoldingFeature(activity: ComponentActivity)
```
TODO
- 디바이스의 접힘 상태(`FoldingFeature`)를 실시간으로 감지하여 상태값으로 저장한다.
- 액티비티가 종료될 때 해당 감지 작업을 중단하도록 lifecycle observer를 등록한다.

### private suspend fun PoseDetector.detectPersonInFrame

```kotlin
private suspend fun PoseDetector.detectPersonInFrame(
    image: Image,
    imageInfo: ImageInfo,
): Boolean
```

TODO

**이미지 프레임 안에 사람이 있는지를 판별**하기 위해,  **ML Kit의 Pose Detection API**를 사용해 **코, 왼쪽 어깨, 오른쪽 어깨**가 **신뢰도 0.7 이상으로 감지**되었는지 확인하는 함수.
### private suspend fun runPoseDetection

```kotlin
@OptIn(ExperimentalGetImage::class)
private suspend fun runPoseDetection()
```

TODO
`runPoseDetection()` 함수는 **ML Kit의 온디바이스 포즈 인식 기능을 사용하여**,  **실시간 카메라 이미지에서 사람의 자세를 감지하고**,  그 결과를 UI 상태(`detectedPose`)에 반영하는 기능을 수행합니다.


### 파라미터
#### rearCameraUseCase

```kotlin
val rearCameraUseCase: RearCameraUseCase
```

## data class CameraUiState

```kotlin
data class CameraUiState(  
    val surfaceRequest: SurfaceRequest? = null,  
    val cameraSessionId: Int = 0,  
    val imageUri: Uri? = null,  
    val detectedPose: Boolean = false,  
    val zoomMaxRatio: Float = 1f,  
    val zoomMinRatio: Float = 1f,  
    val zoomLevel: Float = 1f,  
    val canFlipCamera: Boolean = true,  
    val isRearCameraActive: Boolean = false,  
    val autofocusUiState: AutofocusUiState = AutofocusUiState.Unspecified,  
) {  
    val zoomOptions = when {  
        zoomMinRatio <= 0.6f && zoomMaxRatio >= 1f -> listOf(0.6f, 1f)  
        zoomMinRatio < 1f && zoomMaxRatio >= 1f -> listOf(zoomMinRatio, 1f)  
        zoomMinRatio <= 1f && zoomMaxRatio >= 2f -> listOf(1f, 2f)  
        zoomMinRatio == zoomMaxRatio -> listOf(zoomMinRatio)  
        else -> listOf(zoomMinRatio, zoomMaxRatio)  
    }  
}
```

### Property
*  `surfaceRequest` The current `SurfaceRequest` from the camera preview, or null if not yet available.
* `imageUri` The `Uri` of the captured image, or null if no image has been captured.
* `detectedPose` Indicates whether a pose has been detected, it's true by default.


## sealed interface AutofocusUiState

