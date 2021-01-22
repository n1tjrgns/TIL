## [MSA] API Gateway - (1)

[회고](https://n1tjrgns.tistory.com/283) 글에서 적었듯이 올해는 MSA에 대해서 공부를 해보려한다.

그 중에 첫번째로 API Gateway에 대해 공부를 해보자.



#### API Gateway란?

MSA는 간단하게 말해서 `Monolithic`구조의 서비스를 여러개의 서비스로 쪼개어 관리하는 아키텍쳐다.

- 수 많은 API 호출이 이뤄질텐데, 매번 따로따로 공통된 로직(ex : 인증/인가)을 구현하고 관리해야한다면?

  이는 리소스 낭비임에 분명하다.



> API Gateway를 도입하자!



**특징**

1. API Gateway는 API 서버 앞단에서 모든 Request 엔드포인트를 단일화 해주는 역할을 한다.
   - 그로인해 API 통신량이 줄어들어 대기시간을 줄일 수 있고, 효율성이 높아진다.



2. 모든 요청이 들어오는 통로이기 때문에, 공통 부분(ex: 인증/인가)을 처리하기 좋다.

   - 트래픽을 모니터링하기에도 좋다.

   

3. URI에 따라 서비스 엔드포인트로 `동적 라우팅`이 가능하다.

4. 하지만 트래픽이 급증하였을 때 `Scale-out` 에 대한 처리가 제대로 되지 않는다면, 병목지점이 되어 장애로 이어질 수 있다.

5. 해당 서비스의 API를 바로 호출하는 것이 아닌, gateway를 거치기 때문에 지연이 생길 수 있다.



#### Netflix Zuul vs Spring Cloud Gateway

초창기에는 Zuul을 많이 사용했다.

시간이 지나고 Spring Web Flux가 출시되고 점점 비동기, 논블로킹에 대한 처리 방식이 중요해지면서 Zuul2까지 나오게되었다.

하지만 Zuul은 Spring의 취지(?)와는 맞지 않아 Spring에서는 Spring Cloud Gateway를 만들었다.

마찬가지로 비동기, 논블로킹 방식을 지원하고 무엇보다도 Spring에서 만들었기 때문에 호환성이 좋다.

또한 Spring Cloud Gateway는 Netty 기반으로 동작하기 때문에 Servlet, WAR로 빌드된 경우 동작하지 않는다.



#### 동작 방식

![image-20210113163311476](../AppData/Roaming/Typora/typora-user-images/image-20210113163311476.png)

1. 클라이언트가 Gateway에 요청을 보낸다.
2. Gateway Handler Mapping이 요청 경로와 일치한다면, Gateway Web Handler로 보낸다.
   - 특정한 filter chain을 통해 요청을 실행한다.
3. filter가 점선으로 되어있는데, 그 이유는 필터가 proxy 요청이 전송되기 전과 후(batch의 before, after step) 로직을 실행할 수 있기 때문이다.



#### Spring Cloud CircuitBreaker GatewayFilter Factory

Spring Cloud CircuitBreaker API를 사용해 CircuitBreaker 를 래핑한다.

` spring-cloud-starter-circuitbreaker-reactor-resilience4j `를 추가하면 CircuitBreaker 필터를 활성화 할 수 있다.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: circuitbreaker_route
        uri: https://n1tjrgns.tistory.com
        filters:
        - CircuitBreaker=n1tjrgnsCircuitBreaker
```

 [Resilience4J 참고 문서](https://cloud.spring.io/spring-cloud-circuitbreaker/reference/html/spring-cloud-circuitbreaker.html)



#### Global Filter

API Gateway를 타는 모든 URI 경로에 적용되는 공통 필터

요청이 경로와 일치하면, filtering web handler는 모든 GlobalFilter 인스턴스를 추가하고 각각의 경로별 GatewayFilter를 filterChain에 추가한다.

결합된 필터 체인은 `org.springframework.core.Ordered` 인터페이스 별로 정렬되며, `getOrder()` 메소드를 구현해 설정할 수 있다.

우선 순위가 높은 필터가 before에 실행되고 낮은 단계는 after에 실행된다.

```java
@Bean
public GlobalFilter customFilter() {
    return new CustomGlobalFilter();
}

public class CustomGlobalFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("custom global filter");
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return -1;
    }
}
```



간단하게 API Gateway를 구현하기 위해 몇가지 정보를 알아보았다.

이제 실습을 통해 구현해보자.



#### 참고

https://cloud.spring.io/spring-cloud-gateway/reference/html/



