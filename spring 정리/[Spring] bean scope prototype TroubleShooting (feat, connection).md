## [Spring] bean scope prototype TroubleShooting (feat, connection)

빈 생명주기에 관해서 그래도 기본은 알고있다고 생각했는데 큰 오산이었다.

운영에 반영을 했는데 몇 일 뒤 최대 Connection을 초과해서 장애가 발생하였다.



원인은 Connection close가 되지 않아서였다.



> 왜?? 소스를 어떻게 짰길래??



샘플 소스로 확인해보자.



AServiceImpl는 매번 새로운 커넥션을 맺고 사용이 끝나면 자원을 close 하는 역할을 하고 있다.

common.xml에 Impl 클래스를 prototype으로 빈을 등록해놓았다.

```xml
<bean id="ConnectionService" class="com.example.service.impl.AServiceImpl" scope="prototype" destroy-method="close"/>
```



그리고 다른 클래스에서 매번 새로운 인스턴스로 사용하기 위해서 beanFactory에서 getBean을 통해 직접 해당 클래스를 가져왔다.

```java
@Autowired
private AutowireCapableBeanFactory beanFactory;

AService = beanFactory.getBean(AServiceImpl.class);
```



빈 생명주기에 의해 destroy-method=close가 호출되어 AServiceImpl 클래스의 close 메소드가 동작 할 것이라고 생각했다.
하지만, 생각대로 동작하지 않아 커넥션이 초과되었고 장애가 발생하여 우선은 소스에 하드 코딩으로 close를 호출해주었습니다.

```
finally { ... AService.close(); }
```

왜 destroy-method가 동작하지 않았을까??



갓택오버플로우에서 해답을 찾을 수 있었다.

https://stackoverflow.com/questions/23442512/destroy-method-not-invoked-on-beans-registered-as-prototype



#### 요약

Spring은 prototype의 scope의 전체 라이프 사이클을 관리하지 않는다.

초기화 메소드로 생성이 되고나면, 더 이상 관여하지 않는다.



> 왜? 좀 해주지



예를 들어 배치성 작업처럼 일정 주기마다 새로운 prototype scope의 빈을 생성하는 경우, 어플리케이션이 종료될 때 스프링에서 해당 scope들을 모두 destroy 해야 한다면, 일정 주기마다 생성된 scope bean에 대해서 참조를 유지하고 있어야한다.

따라서 메모리 누수를 유발할 수 있기 때문이다..

