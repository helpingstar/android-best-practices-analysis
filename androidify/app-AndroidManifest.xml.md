
앱의 진입점
```kotlin
alias(libs.plugins.android.application)
```

를 가진 폴더의 `AndroidManifest.xml`

```xml
<application  
    android:name=".AndroidifyApplication"
```

`<provider>`
* `androidx.core.content.FileProvider`
* `androidx.startup.InitializationProvider`
	* meta-data
		* [[core-network-startup-FirebaseAppInitializer.kt]]
	* meta-data
		* [[core-network-startup-FirebaseAppCheckInitializer.kt]]
	* meta-data
		* [[core-network-startup- FirebaseRemoteConfigInitializer]]


`<activity>`
[[app-base-MainActivity.kt]]