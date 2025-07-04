
## FirebaseAppCheckInitializer

```kotlin
class FirebaseAppCheckInitializer : Initializer<FirebaseAppCheck>
```

### fun create

```kotlin
override fun create(context: Context): FirebaseAppCheck
```

- `Firebase.appCheck` (KTX 확장 프로퍼티)를 통해 `FirebaseAppCheck` 싱글턴 객체를 가져옵니다.    
- `installAppCheckProviderFactory(...)` 를 호출해 Play Integrity 기반의 App Check 제공자 팩토리를 등록합니다.
- `FirebaseAppCheck` 객체를 반환하여, 이후 App Startup 프레임워크가 관리하도록 합니다.

### fun dependencies

```kotlin
override fun dependencies(): List<Class<out Initializer<*>?>?>
```

이 이니셜라이저가 실행되기 전에 반드시 먼저 실행되어야 하는 다른 Initializer(들)을 명시합니다.
`FirebaseAppInitializer::class.java` 를 리스트에 담아 반환  
→ 이 클래스가 먼저 실행된 후에야 `Firebase.appCheck` 를 안전하게 사용할 수 있기 때문에 의존성을 설정한 것