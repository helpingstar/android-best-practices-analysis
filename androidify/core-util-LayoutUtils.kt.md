
## fun calculateWindowSizeClass
```kotlin
@Composable  
fun calculateWindowSizeClass(): WindowSizeClass
```

## fun isAtLeastMedium
```kotlin
@Composable  
fun isAtLeastMedium(): Boolean
```

현재 화면의 크기가 중간 크기(Medium) 이상인지 판단하는 함수

```kotlin
val sizeClass = calculateWindowSizeClass()
```
* calculateWindowSizeClass(): 현재 화면의 크기 정보를 계산

```kotlin
return sizeClass.isWidthAtLeastBreakpoint(WindowSizeClass.WIDTH_DP_MEDIUM_LOWER_BOUND)
```
* `WindowSizeClass.WIDTH_DP_MEDIUM_LOWER_BOUND`: Android의 Medium 크기 하한선 (일반적으로 600dp)
* `isWidthAtLeastBreakpoint()`: 현재 화면 너비가 지정된 breakpoint 이상인지 확인