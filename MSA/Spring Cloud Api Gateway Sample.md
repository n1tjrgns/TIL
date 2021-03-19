```groovy
plugins {
    id 'org.springframework.boot' version '2.2.5' //spring cloud gateway에서 사용하는 ga(Generally Available)
    id 'io.spring.dependency-management' version '1.0.10.RELEASE'
    id 'java'
}

group = 'com.msa'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {
    compile group: 'org.springframework.cloud', name: 'spring-cloud-starter-gateway' //spring cloud gateway를 위한 라이브러리
    // netflix에서 Circuit Breaker Pattern을 구현한 라이브러리리, 하지만 더 이상 추가기능 제공되지 않고 유지보수만 한다고함
    // 최근에는 resilience4j를 사용
    //compile group: 'org.springframework.cloud', name: 'spring-cloud-starter-netflix-hystrix'
    compile group: 'io.github.resilience4j', name: 'resilience4j-circuitbreaker'

    // Contract Test를 하기 위한 라이브러리, 서비스 제공장
    compile group: 'org.springframework.cloud', name: 'spring-cloud-starter-contract-stub-runner'

    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    compileOnly 'org.projectlombok:lombok'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    runtimeOnly 'com.h2database:h2'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

test {
    useJUnitPlatform()
}

```



#### 용어

- Route(경로) : 게이트웨이의 기본 골격. ID, 목적지 URI, 조건부 집합, 필터 집합으로 구성. 모든 조건자가 충족되는 경우 매칭된다.

- Predicate(조건부) : 각 요청을 처리하기 전에 실행되는 로직.

  헤더와 입력된 값 등 다양한 HTTP 요청이 정의된 기준에 맞는지 확인한다.

- Filter(필터) : HTTP 요청 또는 나가는 HTTP 응답을 수정할 수 있게한다.

  Filter에서 다운스트림 요청 전후에 요청/응답을 수정할 수 있다.



#### application.yml

```yaml
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
      routes: #client1-svc, client2-svc 총 두 개의 마이크로 서비스 라우팅
        - id: client1-svc   #조건부에 따라서 http://localhost:8080/client1/ 이 호출된다면 http://localhost:8081/client1/ 서비스가 호출된다.
          uri: http://localhost:8081/
          predicates:
            - Path=/client1/**
          filters: # 각 서비스 호출 전 호출되는 필터
            - name: Client1Filter
              args:
                baseMessage: Spring Cloud Gateway Client1Filter
                preLogger: true
                postLogger: true
        - id: client2-svc
          uri: http://localhost:8082/
          predicates:
            - Path=/client2/**
          filters:
            - name: Client2Filter
              args:
                baseMessage: Spring Cloud Gateway Client2Filter
                preLogger: true
                postLogger: true




```





#### GlobalFilter

```java
package com.msa.apigatewaysample.filter;

import com.msa.apigatewaysample.ConfigField;
import lombok.extern.slf4j.Slf4j;
import org.springframework.cloud.gateway.filter.GatewayFilter;
import org.springframework.cloud.gateway.filter.factory.AbstractGatewayFilterFactory;
import org.springframework.stereotype.Component;
import reactor.core.publisher.Mono;

@Slf4j
@Component
public class GlobalFilter extends AbstractGatewayFilterFactory<ConfigField> {


    @Override
    public GatewayFilter apply(ConfigField configField) {
        return ((exchange, chain) -> {
            log.info("GlobalFilter :"+ configField.toString());
            log.info("GlobalFilter Message : "+ configField.getBaseMessage());
            
            if (configField.isPreLogger()){
                log.info("GlobalFilter START : " + exchange.getRequest());
            }
            return chain.filter(exchange).then(Mono.fromRunnable(() ->{
                if (configField.isPostLogger()){
                    log.info("GlobalFilter END : " + exchange.getResponse());
                }
            }));
        });
    }
}


```



- AbstractGatewayFilterFactory : Gateway를 구현하기 위해 GatewayFilterFactory를 구현해야 하며, 상속할 수 있는 추상 클래스가  AbstractGatewayFilterFactory 이다.

- exchange : 서비스 요청/응답값을 담고있는 변수. 요청/응답값을 출력하거나 변환할 때 사용.

  요청값은 `(exchange, chain) -> ` 구문 이후에 얻을 수 있고, 서비스로부터 리턴받은 응답값은 `Mono.fromRunnable(() ->)` 구문 이후부터 얻을 수 있다.

- config : application.yml에 선언한 각 `filter의 args(인자값)` 사용을 위한 클래스

- 각각의 필터마다 그에 맞는 동작을 구현해주어야 한다.





