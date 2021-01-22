## [MSA] API Gateway - (2)

[API Gateway개념](https://n1tjrgns.tistory.com/284)



#### 간단한 API Gateway 구축하기

**환경**

JAVA : 8

SpringBoot : 2.2.5 (글 작성 시점 Spring Cloud Gateway Generally Available 버전)

Gradle : 6.7.1

IDE : Intellij

- 여러 프로젝트를 관리해야 하기 때문에 멀티모듈로 시도해보았다.



#### build.gradle

```groovy
plugins {
    id 'org.springframework.boot' version '2.2.5.RELEASE' //spring cloud gateway에서 사용하는 ga(Generally Available)
    id 'io.spring.dependency-management' version '1.0.9.RELEASE'
    id 'java'
}



configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}



//전체에 적용하고 싶다면 allprojects를 사용하면 된다.
//내부 모듈에만 적용하고 싶다면 subprojects
//allprojects와 subprojects에는 plugins를 쓸 수 없어 apply plugin을 사용해야 한다.
allprojects {
    apply plugin : 'java'
    apply plugin : 'org.springframework.boot'
    apply plugin : 'io.spring.dependency-management'

    group = 'com.msa'
    version = '0.0.1-SNAPSHOT'
    sourceCompatibility = '1.8'

    repositories {
        mavenCentral()
    }

    dependencies {
        //spring cloud gateway를 위한 라이브러리
        compile group: 'org.springframework.cloud', name: 'spring-cloud-starter-gateway', version: '2.2.5.RELEASE'

        // netflix에서 Circuit Breaker Pattern을 구현한 라이브러리리
        compile group: 'org.springframework.cloud', name: 'spring-cloud-starter-netflix-hystrix', version: '2.2.5.RELEASE'
        
        compileOnly 'org.projectlombok:lombok'
        annotationProcessor 'org.projectlombok:lombok'
    }

}

test {
    useJUnitPlatform()
}

```



#### application.yml

Spring Cloud Gateway는 크게 3 부분으로 구성된다.



**Route :** 특정 Request에 대한 목적 URI / predicates와 하나 이상의 filter로 구성된다.

**Predicate :** 일치하는 조건을 나타냄 / path 또는 request header에 특정 값을 포함될 수 있다.

**filter :**  Spring WebFilter의 인스턴스 / before, after 작업으로 요청이나 응답에 대한 작업을 수행할 수 있다.



아래 설정은 Gateway에 대한 설정이다.

```yml
server:
  port: 8080

---
spring:
  cloud:
    gateway:
      default-filters: # 공통 필터
        - name: GlobalFilter
          args:
            baseMessage: Spring Cloud Gateway GlobalFilter
            preLogger: true
            postLogger: true
      routes: #라우팅 설정/ client1, client2 총 두 개의 마이크로 서비스 라우팅
        - id: client1   #조건부에 따라서 http://localhost:8080/client1/ 이 호출된다면 http://localhost:8081/client1/ 서비스가 호출된다.
          uri: http://localhost:8081/
          predicates:
            - Path=/client1/**
          filters: # 각 서비스 호출 전 호출되는 필터
            - name: Client1Filter
              args:
                baseMessage: Spring Cloud Gateway Client1Filter
                preLogger: true
                postLogger: true
        - id: client2
          uri: http://localhost:8082/
          predicates:
            - Path=/client2/**
          filters:
            - name: Client2Filter
              args:
                baseMessage: Spring Cloud Gateway Client2Filter
                preLogger: true
                postLogger: true

#게이트웨이 프로세스 추적
logging:
  level:
    org.springframework.cloud.gateway: DEBUG
    reactor.netty.http.client: DEBUG


```

API가 gateway를 타게된다면, 공통 filter를 거치고  라우팅 설정에 따라서 URI와 적합한 predicates에 맞게 라우팅되게된다.

그 후 각각에 해당하는 개별 filter 설정을 거치게된다.



----



#### CommonFilter

Filter 어떠한 방식으로 요청/응답에 대한 작업을 할지 구현을 해주어야한다.

```java
@Slf4j
@Component
public class CommonFilter extends AbstractGatewayFilterFactory<CommonFilter.ConfigField> {

    public CommonFilter(){
        super(ConfigField.class);
    }
	
    //application.yml에 filter args 인자값에 대한 생성자를 생성해줘야한다.
	@Getter @Setter
    public static class ConfigField{
        private String baseMessage; //로그 항목에 포함될 사용자 지정 메세지
        private boolean preLogger; //요청을 전달하기 전 필터 기록여부를 나타내는 플래그
        private boolean postLogger; //프록시 된 서비스에서 응답을 받은 후 필터 기록여부를 나타내는 플래그
    }
    
    @Override
    public GatewayFilter apply(ConfigField configField) {
        ...
    }
}


```

`AbstractGatewayFilterFactory`  

- Gateway를 구현하기 위해서, GatewayFilterFactory를 구현해야하는데, 이때 사용하는 추상 클래스다.좀 더 세분화 된 사용자 지정 filter 작업을 할 수 있다.



AbstractGatewayFilterFactory의 apply 메소드를 오버라이딩해 어떠한 내용을 적용시킬지 보면,

```java
	@Override
    public GatewayFilter apply(ConfigField configField) {
        return ((exchange, chain) -> {
            log.info("CommonFilter :"+ configField.toString());
            log.info("CommonFilter Message : "+ configField.getBaseMessage());

            if (configField.isPreLogger()){
                log.info("CommonFilter START : " + exchange.getRequest());
            }
            return chain.filter(exchange).then(Mono.fromRunnable(() ->{
                if (configField.isPostLogger()){
                    log.info("CommonFilter END : " + exchange.getResponse());
                }
            }));
        });
    }
```

return문의 `exchange`가 실제 요청/응답에 대한 값을 담고 있어서 이를 활용해 구현을 하게된다.

요청에 대한 작업 : `(exchange, chain) -> {}`

응답에 대한 작업 : `Mono.fromRunnable(() -> {})` 



CommonFilter, Client1Filter, Client2Filter 모두 동일한 방식으로 구현했다.

----

#### BeforeFilter

[이전 글](https://n1tjrgns.tistory.com/284)에서 filter 작업에 우선 순위를 부여할 수 있고, batch처럼 before, after 작업을 할 수 있다고 공부했다.



> 어떻게?



```java
@Slf4j
@Component
public class BeforeFilter implements GlobalFilter {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("Global Before Filter executed");
        return chain.filter(exchange); 
                
    }
}    
```

`GlobalFilter`를 상속받아 `Mono<Void> filter` 메소드를 구현하게되면 사전 작업을 수행.



#### PostFilter

```java
@Slf4j
@Configuration
public class PostFilter {

    @Bean
    public GlobalFilter postGlobalFilter() {
        return (exchange, chain) -> {
            return chain.filter(exchange)
                    .then(Mono.fromRunnable(() -> {
                        log.info("Global Post Filter executed");
                    }));
        };
    }
}

```

`Mono.fromRunnable()` 을 사용해 after 작업을 수행할 수 있다.



> 어라 위에서 했던거랑 비슷한데?

그렇다. 2가지 방식을 묶어서 CommonFilter에 작성한 방식처럼 사용할 수 있다.



----

#### 우선순위

filter 우선순위는 생각보다 간단하다.

`Ordered` 라는 인터페이스를 상속받아 `getOrder` 함수를 구현해주면 된다. (숫자가 작을수록 우선순위가 높다.)



#### BeforeOrderedFilter

```java
@Slf4j
@Component
public class BeforeOrderedFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("Order number -1 BeforeOrderedFilter "); //before 로직
        return chain.filter(exchange)
                .then(Mono.fromRunnable(() -> {
                    log.info("BeforeOrderedFilter END "); //after 로직
                }));
    }

    //우선순위
    @Override
    public int getOrder() {
        return -1;
    }
}
```



위에서 작성한 BeforeFilter 클래스가 그 다음 필터 역할로 올 수 있도록 Ordered를 추가해보자

#### BeforeFilter

```java
@Slf4j
@Component
public class BeforeFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("Global Before Filter executed");
        return chain.filter(exchange)
                .then(Mono.fromRunnable(() -> {
                    log.info("BeforeFilter END "); //after 로직
                }));
    }

    @Override
    public int getOrder() {
        return 0;
    }
}
```



-----

이제 어플리케이션을 실행시킨 후 호출을 하게 되면 어떤 순서로 동작할까?

실제로 동작 시켜보기 전에 한 번 예측해보자.



#### 에러 발생

```
Parameter 0 of method modifyRequestBodyGatewayFilterFactory in org.springframework.cloud.gateway.config.GatewayAutoConfiguration required a bean of type 'org.springframework.http.codec.ServerCodecConfigurer' that could not be found.
```

spring-cloud-starter-gateway 의존성이 spring-boot-starter-web와 충돌하여 발생하는 문제였다.

gateway에서는 스프링 부트 웹을 사용하지 않고 웹 플럭스를 사용해야 충돌이 발생하지 않는다.

`spring-boot-starter-webflux`



localhost:8080/client1/ping 을 호출해보도록 하겠다.

![image-20210115141120645](../AppData/Roaming/Typora/typora-user-images/image-20210115141120645.png)

gateway에 대한 로그를 설정해두었기 때문에 보다 자세하게 확인이 가능하다.

client1에 매칭이 되었고, 요청정보에 담긴 Exchange 정보와 route 될 정보 등등에 대한 정보를 확인할 수 있다.



```
BeforeOrderedFilter - BeforeFilter - CommonFilter - ClientFilter
```

우선순위를 가진 필터 - 공통 필터 - 개별 필터의 순서대로 적용이 잘 된것을 확인할 수 있다.



마지막 해당 Controller는 아직 구현을 하지 않아서 에러가 발생하였다.

다음에는 Client1을 구현해보자.



#### 참고

https://spring.io/projects/spring-cloud-gateway

https://www.baeldung.com/spring-cloud-custom-gateway-filters

https://medium.com/@niral22/spring-cloud-gateway-tutorial-5311ddd59816

























#### 테스트 에러

멀티모듈로 인증서버만 h2를 사용하는데 왜 다른 모듈에서 h2 의존성을 찾지?

```
Description:

Failed to configure a DataSource: 'url' attribute is not specified and no embedded datasource could be configured.

Reason: Failed to determine a suitable driver class


Action:

Consider the following:
	If you want an embedded database (H2, HSQL or Derby), please put it on the classpath.
	If you have database settings to be loaded from a particular profile you may need to activate it (no profiles are currently active).

```

의존성 주입중 spring-boot-start-data-jpa가 원인



jpa

하지만 나는 allproject안에 data-jpa를 넣어놔서 다른곳에서도 발생했던것



에러

```
Parameter 0 of method modifyRequestBodyGatewayFilterFactory in org.springframework.cloud.gateway.config.GatewayAutoConfiguration required a bean of type 'org.springframework.http.codec.ServerCodecConfigurer' that could not be found.
```

***이유 :***
종속성 충돌, spring-boot-starter-web이있는 spring-cloud-starter-gateway 및 spring-boot-starter-webflux 종속성 충돌

***해결 :\***
부정적인 spring-boot-starter-web 및 spring-boot-starter-webflux 종속



gateway는 starter-web말고 starter-webflux를 사용해야함



**참고**

https://spring.io/projects/spring-cloud-gateway

https://www.baeldung.com/spring-cloud-custom-gateway-filters

https://medium.com/@niral22/spring-cloud-gateway-tutorial-5311ddd59816

