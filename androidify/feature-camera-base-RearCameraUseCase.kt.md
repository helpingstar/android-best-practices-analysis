## class RearCameraUseCase

```kotlin
@OptIn(ExperimentalWindowApi::class)
@ViewModelScoped
class RearCameraUseCase @Inject constructor(@ApplicationContext context: Context) :
    WindowAreaSessionCallback {
```

### fun init

```kotlin
fun init(activity: ComponentActivity)
```

### fun shouldDisplayRearCameraButton

```kotlin
fun shouldDisplayRearCameraButton(): Boolean
```

### fun isRearCameraActive

```kotlin
fun isRearCameraActive(): Boolean
```

### fun toggleRearCameraDisplay

```kotlin
fun toggleRearCameraDisplay(activity: ComponentActivity)
```

### override fun onSessionEnded

```kotlin
override fun onSessionEnded(t: Throwable?)
```

### override fun onSessionStarted

```kotlin
override fun onSessionStarted(session: WindowAreaSession)
```
