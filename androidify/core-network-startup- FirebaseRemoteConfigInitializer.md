## class FirebaseRemoteConfigInitializer

```kotlin
class FirebaseRemoteConfigInitializer : Initializer<FirebaseRemoteConfig>
```

### fun create

```kotlin
override fun create(context: Context): FirebaseRemoteConfig
```

```kotlin
val configSettings = remoteConfigSettings {  
    minimumFetchIntervalInSeconds = 600  
}
```

* 앱이 **서버에 fetch 요청을 너무 자주 보내지 않도록 제한**하는 역할
* 앱이 실행되거나 사용자가 새로고침을 눌러 fetch를 시도하면:
	- 10분이 지남 : 서버에 fetch 요청이 정상적으로 전송
	- 10분이 안 지남 : **로컬에 저장된 기존 구성**이 그대로 사용되고, fetch는 무시

```kotlin
setConfigSettingsAsync(configSettings)
```

위에서 만든 설정을 `FirebaseRemoteComfig` 에 비동기로 적용

```kotlin
setDefaultsAsync(R.xml.remote_config_defaults)
```

XML에 정의된 기본값을 설정

> Remote Config 값 우선 순위
> 1. 앱에서 서버로부터 fetch한 최신값
> 2. 앱에서 설정된 기본값(`setDefaultsAsync`)
> 3. 기본 값이 없을 경우 `null`

```kotlin
fetchAndActivate()
```

서버에서 최신 구성을 가져오고 바로 반영
### fun dependencies

```kotlin
override fun dependencies(): List<Class<out Initializer<*>?>?>
```

`FirebaseApp`이 먼저 초기화되어야 하므로 `FirebaseAppInitializer`를 의존성으로 선언


