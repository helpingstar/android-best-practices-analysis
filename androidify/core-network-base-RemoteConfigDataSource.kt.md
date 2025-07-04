
## interface RemoteConfigDataSource


## class RemoteConfigDataSourceImpl

```kotlin
@Singleton  
class RemoteConfigDataSourceImpl @Inject constructor() : RemoteConfigDataSource
```

* `RemoteConfigDataSource` 인터페이스를 구현한 클래스
* `Impl` 은 이 클래스가 인터페이스의 실제 동작을 정의하고 있다는 뜻이다.

### @Singleton
* 이 클래스의 인스턴스를 앱 전체에서 하나만 생성해서 재사용하겠다.


### @Inject
* 생성자 주입(Constructor Injection)을 위한 annotation
* `@inject constructor()`
	* DI(Dependency Injection) 프레임워크가 이 클래스를 자동으로 생성(주입)할 수 있게 만든 생성자 표시
	* 

생성자 주입
* 필요한 의존성을 생성자에서 받아서 객체를 만드는 방식
* 클래스 안에서 직접 필요한 걸 만드는 게 아니라, 외부에서 주입한다.
* 테스트가 쉽고, 결합도는 낮아지고, 재사용성이 높아진다.