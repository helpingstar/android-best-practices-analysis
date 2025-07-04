## suspend fun \<R\> ProcessCameraProvider.runWith

```kotlin
suspend fun <R> ProcessCameraProvider.runWith(
    cameraSelector: CameraSelector,
    useCases: UseCaseGroup,
    block: suspend CoroutineScope.(Camera) -> R
): R
```

## internal class CoroutineLifecycleOwner

```kotlin
internal class CoroutineLifecycleOwner(
    coroutineContext: CoroutineContext
) : LifecycleOwner
```