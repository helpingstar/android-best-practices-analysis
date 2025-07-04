## class ZoomState

```kotlin
@Stable
class ZoomState(
    initialZoomLevel: Float,
    val zoomRange: ClosedFloatingPointRange<Float>,
    val onChangeZoomLevel: (Float) -> Unit,
)
```

### suspend fun absoluteZoom

```kotlin
suspend fun absoluteZoom(targetZoomLevel: Float)
```

### suspend fun scaleZoom

```kotlin
suspend fun scaleZoom(scalingFactor: Float)
```

### suspend fun animatedZoom

```kotlin
suspend fun animatedZoom(
    targetZoomLevel: Float,
    animationSpec: AnimationSpec<Float> = tween(durationMillis = 500),
)
```
