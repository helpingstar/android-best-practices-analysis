
## class HomeViewModel

```kotlin
@HiltViewModel  
class HomeViewModel @Inject constructor(configProvider: ConfigProvider) : ViewModel()
```

TODO : configProvider
* `ViewModel()` 을 상속한다.


### @HiltViewModel
* Hilt가 ViewModel을 DI(의존성 주입)를 통해 관리할 수 있게 해주는 애노테이션
* 이 annotation을 통해 ViewModel은 자동으로 Hilt의 DI Container에 등록되고, 필요한 의존성을 주입받을 수 있게 된다.
* 생성자에 `@Inject` 를 함께 사용하여 의존성을 주입받는다.
	* TODO : [[data-ConfigProvider.kt#class ConfigProvider|ConfigProvider]]


## data class HomeState

```kotlin
data class HomeState(  
    val isAppActive: Boolean = true,  
    val videoLink: String? = null,  
    val dancingDroidLink: String? = null,  
)
```


