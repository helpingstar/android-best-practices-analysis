
```kotlin
@file:OptIn(ExperimentalSerializationApi::class)
```
파일 전체에서 실험적(Experimental) API를 사용할 수 있게 허용

## sealed interface NavigationRoute

```kotlin
sealed interface NavigationRoute
```

- `sealed class`나 `sealed interface`는 **같은 파일 내에서만** 구현체를 만들 수 있음
- 즉, **컴파일 타임에 모든 구현체를 알 수 있기 때문에** `when` 구문에서 `else` 없이 안전하게 처리할 수 있음.
	- 컴파일 타임에 모든 가능한 라우트를 알 수 있다.

## data object Home

```kotlin
@Serializable  
data object Home : NavigationRoute
```
* `data object` : 단일 인스턴스만 존재하는 싱글톤 객체이면서도 데이터 클래스의 장점을 가짐
* `Home` 화면은 매개변수가 필요 없는 단순한 화면이므로 단일 객체로 충분하다.


## data class Create

```kotlin
@Serializable  
data class Create(val fileName: String? = null, val prompt: String? = null) : NavigationRoute
```

* Create 화면은 매개변수(fileName, prompt)를 받아야 한다. data class는 여러 인스턴스를 생성할 수 있고, 각각 다른 매개변수를 가질 수 있기 때문에 `data class`
## object Camera

```kotlin
@Serializable  
object Camera : NavigationRoute
```
## object About

```kotlin
@Serializable  
object About : NavigationRoute
```




