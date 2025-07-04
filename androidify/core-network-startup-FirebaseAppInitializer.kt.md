
## class FirebaseAppInitializer

```kotlin
@SuppressLint("EnsureInitializerMetadata") // Registered in :app module  
class FirebaseAppInitializer : Initializer<FirebaseApp>
```

### @SuppressLint("EnsureInitializerMetadata")
* Lint는 `FirebaseAppInitializer` 같은 `Initializer` 클래스가 `AndroidManifest.xml`에 `<meta-data>`로 등록되어 있는지 확인합니다.
* "등록 안 했다고 걱정하지 마, 실제로는 여기에 등록했거든."


### fun create

```kotlin
override fun create(context: Context): FirebaseApp
```

### fun dependencies

```kotlin
override fun dependencies(): List<Class<out Initializer<*>?>?>
```

이 메서드는 `FirebaseAppInitializer`가 **다른 어떤 초기화 클래스(Initializer)에 의존하지 않는다는 것**을 명시합니다.

App Startup은 여러 개의 초기화 클래스(`Initializer`)가 있을 때:

- **어떤 순서로 초기화할지**를 자동으로 결정해야 합니다.
- `dependencies()`를 통해 **"내가 어떤 클래스보다 나중에 실행되어야 해"** 라고 명시할 수 있습니다.

#### `List<Class<out Initializer<*>?>?>
* `*` : 타입 파라미터를 알 수 없거나 상관 없을 때
* `Initialize<*>?` : 타입 파라미터가 어떤 것이든 상관없는 `Initializer` 타입이며, 그 객체는 **null일 수도 있다**.
* `out Initializer<*>?` : **`Initializer` 타입 또는 그 하위 타입**이고, 그 객체는 **null일 수도 있다**.  단, 이 타입은 **읽기 전용(covariant)** 으로만 사용 가능하다.
	* `out` : Kotlin에서 **공변성(covariance)** 을 나타내며, **읽기 전용(consumer)** 으로 사용될 때 타입 안정성을 보장합니다.
* `Class<out Initializer<*>?>?` : **`Initializer` 또는 그 하위 타입을 나타내는 `Class` 객체일 수도 있고, null일 수도 있다.**  또한, `Initializer`의 타입 파라미터는 어떤 것이든 괜찮다.
	* `Class<T>?` : `T` 타입의 Java 클래스 객체일 수도 있고, null일 수도 있다.
* `List<T>` : **타입 `T`의 요소들로 이루어진 읽기 전용 리스트**
	* `MutableList<T>` : Kotlin에서 **요소를 추가하거나 삭제할 수 있는 리스트**입니다.