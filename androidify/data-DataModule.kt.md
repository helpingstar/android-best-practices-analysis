
## internal object DataModule

```kotlin
@Module  
@InstallIn(SingletonComponent::class)  
internal object DataModule
```

* `internal object`
	* `internal`
		* 같은 Gradle 모듈 내에서는 어디에서든 접근할 수 있지만,  다른 모듈(라이브러리 A → 애플리케이션, 혹은 멀티모듈 프로젝트의 다른 서브모듈)에서는 보이지 않는다.
			* 현재 프로젝트에서의 모듈들
		* API 노출 최소화 : 외부에 불필요하게 노출될 필요가 없는 구현 세부 사항(DataModule 같은 DI 설정)을 숨길 때 유용
	* `object`
		* 싱글턴 객체 선언 : 이 클래스는 딱 한 번 생성되는 singleton
		* 정적(Static) 멤버로 접근 가능 : 내부에 선언된 `@Provide` 함수나 프로퍼티는 `DataModule.provide<XXX>` 형태로 호출(또는 reflection/Annotation processing) 할 수 있다.
		* 인스턴스 관리 불필요 : 별도의 생성자 호출 없이, 컴파일 타임에 곧바로 싱글턴 인스턴스가 준비된다.

```kotlin
// settings.gradle.kts
rootProject.name = "Androidify"  
include(":app")  
include(":feature:camera")  
include(":feature:creation")  
include(":feature:home")  
include(":feature:results")  
include(":data")  
include(":core:network")  
include(":core:util")  
include(":core:theme")  
include(":core:testing")  
include(":benchmark")
```
위에 나열된 각각의 항목이 Gradle Module(subproject)를 가리킨다.

* `include(":feature:camera")` : 프로젝트 루트 기준으로 `feature/camera` 디렉터리를 하나의 모듈로 등록하라
### @Module

Hilt/Dagger에게 이 객체 안에 `@Provides`나 `@Binds`로 표시된 의존성 공급 함수들이 있다고 알려준다.

### @InstallIn(SingletonComponent::class)

이 모듈이 앱 전체 생명주기(SingletonComponent)에 바인딩되도록 지정한다. 여기서 제공되는 의존성들은 기본적으로 앱이 살아있는 한 싱글톤으로 유지된다.


### fun provideConfigProvider

```kotlin
@Provides  
fun provideConfigProvider(remoteConfigDataSource: RemoteConfigDataSource): ConfigProvider =  
    ConfigProvider(remoteConfigDataSource)
```

#### @Provides

Dagger/Hilt 의 **프로바이더(provider) 메서드**임을 표시하는 역할