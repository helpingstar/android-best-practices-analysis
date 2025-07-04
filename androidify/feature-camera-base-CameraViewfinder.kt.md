## internal fun CameraViewfinder

```kotlin
@Composable
internal fun CameraViewfinder(
    surfaceRequest: SurfaceRequest,
    autofocusUiState: AutofocusUiState,
    tapToFocus: (tapCoords: Offset) -> Unit,
    onScaleZoom: (zoomScaleFactor: Float) -> Unit,
    modifier: Modifier = Modifier,
)
```

```kotlin
val onScaleCurrentZoom by rememberUpdatedState(onScaleZoom)  
val currentTapToFocus by rememberUpdatedState(tapToFocus)  
val coordinateTransformer = remember { MutableCoordinateTransformer() }
```

| 기능                     | `remember`                                    | `rememberUpdatedState`             |
| ---------------------- | --------------------------------------------- | ---------------------------------- |
| 상태 저장                  | ✅ 예                                           | ❌ 직접 저장하지는 않음                      |
| recomposition 시 새로 만듦? | ❌ 아니요, 재사용함                                   | ✅ 새로 람다 들어오면 업데이트함                 |
| 대상                     | 값, 객체                                         | 함수, 람다                             |
| 언제 사용?                 | 값 한 번만 만들고 계속 쓸 때                             | 콜백을 최신 상태로 유지할 때                   |
| 예시                     | `remember { MutableCoordinateTransformer() }` | `rememberUpdatedState(tapToFocus)` |

* TODO : onScaleCurrentZoom
* TODO : currentTapToFocus
* `MutableCoordinateTransformer` : `CamearX`의 프리뷰 화면은 종종 회전, 확대, 비율 왜곡 등이 적용되어 UI 좌표계와 일치하지 않는다. 사용자가 화면을 탭한 좌표 `(x, y)` 를 단순히 카메라 포커스 좌표로 사용하면 정확하지 않다.
	* `MutableCoordinateTransformer` : Preview에서 사용하는 내부 좌표계와 UI(Compose)의 좌표계 간의 변환이 가능해진다.
	* 
```kotlin
CameraXViewfinder(  
    surfaceRequest = surfaceRequest,  
    coordinateTransformer = coordinateTransformer,  
    modifier = modifier  
        .pointerInput(coordinateTransformer) {  
            detectTapGestures { tapCoords ->  
                with(coordinateTransformer) {  
                    currentTapToFocus(tapCoords.transform())  
                }  
            }        }        .transformable(  
            rememberTransformableState(  
                onTransformation = { zoomChange, _, _ ->  
                    onScaleCurrentZoom(zoomChange)  
                },  
            ),  
        ),  
)
```

* `CameraXViewfinder` : Jetpack Compose에서 CameraX를 이용해 **카메라 프리뷰를 보여주면서**,  **사용자의 터치 및 핀치(줌) 제스처를 처리하는 UI 구성 코드**
	* `surfaceRequest = surfaceRequest`
		* `CameraXViewFinder` 에게 CameraX가 제공한 `SurfaceRequest`를 넘겨준다.
			* `SurfaceRequest` : CameraX가 "나 여기다 영상 그릴 수 있게 Surface 좀 줘" 하고 요청한 것
			* 내부에서 이 요청을 처리해 Surface를 만들고 CameraX에 연결
	* `coordinateTransformer = coordinateTransformer`
		* 좌표 변환을 담당하는 객체
		* `remember { MutableCoordinateTransformer() }` 로 만들어진 걸 전달한다.
		* `CameraXViewfinder` 내부에서 **Surface 크기/회전 등을 기준으로 이 변환 정보를 자동으로 업데이트**
	* `modifier`
		```kotlin
		.pointerInput(coordinateTransformer) {
		    detectTapGestures { tapCoords ->
		        with(coordinateTransformer) {
		            currentTapToFocus(tapCoords.transform())
		        }
		    }
		}
		
		```
	* `pointerInput` : 제스처 입력을 감지할 수 있게 해주는 Modifier
		* Kotlin에서 **마지막 파라미터가 함수 타입이면** `{}` 블록을 마지막 괄호 바깥으로 쓸 수 있음 따라서 `onTap` 에서의 행동을 정의한다.
		```kotlin
		// TapGestureDetector.kt
		...
		suspend fun PointerInputScope.detectTapGestures(  
		    onDoubleTap: ((Offset) -> Unit)? = null,  
		    onLongPress: ((Offset) -> Unit)? = null,  
		    onPress: suspend PressGestureScope.(Offset) -> Unit = NoPressGesture,  
		    onTap: ((Offset) -> Unit)? = null  
		)
		...
		```

		* `detectTapGestures { tapCoords -> ...}`
			* 사용자의 `onTab` 을 감지하고 해당 좌표를 전달받는다, 그러면 `tabCoords` 는 그 위치의 `Offset(x,y)` 값을 가진다.
		* `with(coordinateTransformer) { currentTapToFocus(tapCoords.transform()) }`
			* TODO : with 문 이해, Kotlin의 **확장 함수(Extension Function)** 개념
			* 사용자가 탭한 화면 좌표(`tapCoords`)를 카메라 좌표계로 변환한다.
			* 변환된 좌표를 카메라에 전달하여 해당 위치에 초점을 맞춘다.
			* `currentTabToFocus` : [[feature-camera-base-CameraViewModel.kt#fun tapToFocus|tapToFocus]]
	* `transformable(...)`
		* pinch zoom, rotation, pan 같은 제스처를 인식할 수 있게 함
		* `Modifier.transformable(...)`을 뷰 컴포넌트에 붙이면, 해당 뷰가 위와 같은 제스처를 감지하고 처리할 수 있게 됨
		* `onTransformation = { zoomChange, _, _ ->  onScaleCurrentZoom(zoomChange) }`
			* `onTransformation` : 사용자 제스처가 일어날 때마다 호출되는 콜백