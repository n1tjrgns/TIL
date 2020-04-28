#### 마이크로서비스 @PathVariable, @RestController, @EnableEurekaClient, @ HystrixCommand,  @EnableCircuitBreaker

-----

요즘들어 스프링 부트에 관심이 많아졌다.

워낙 업무에 레거시한 소스들을 많이 보다보니 더욱 더 최신 소스를 갈망하는 것 같다...



스프링 부트로 프로젝트를 해보거나, 뭔가 제대로 만들어본 적이 없어서 이참에 공부를 하여 클론 코딩을 해보려 한다.



그러기 위해서 부트를 제대로 공부해보자.



지인이 추천해준 `스프링 마이크로서비스 코딩 공작소`라는 책을 구매해 공부 할 예정이다.

스프링 부트를 공부하면서 `REST`작성 방법도 같이 공부 할 예정이다.



또한 책 이름에 맞게 나는 마이크로서비스를 위한 구현 방식을 공부하고, 그에 대한 코드 작성법을 공부 할 것이다.



##### 스프링 부트를 사용한 전형적인 HELLO WORLD 예제

```
@SpringBootApplication //스프링 부트 서비스의 진입점을 지정
@RestController //이 클래스의 코드를 스프링 RestController 클래스로 노출하도록 지정
@RequestMapping(value = "hello")
public class MicroserviceApplication {

    public static void main(String[] args) {
        SpringApplication.run(MicroserviceApplication.class, args);
    }

    //두 매개변수 first와 last를 입력받는 GET 기반의 REST 엔드포인트 노출
    @RequestMapping(value = "/{firstName}/{lastName}", method = RequestMethod.GET)
    //@PathVariable을 통해 url에 전달된 first, lastName 매개변수를 hello 함수에 전달에 두 변수에 매핑해준다.
    public String hello(@PathVariable("firstName") String firstName ,@PathVariable("lastName") String lastName){
        return String.format("{\"message\":\"Hello %s %s \"}", firstName, lastName);
    }
}

```



위 방식은 단순하게 PathVariable 어노테이션을 사용해서 url에 전달된 값을 변수로 매핑해 받는 예제이다.



##### 스프링 클라우드를 사용하는 Hello World 예제

```
@SpringBootApplication //스프링 부트 서비스의 진입점을 지정
@RestController //이 클래스의 코드를 스프링 RestController 클래스로 노출하도록 지정
@RequestMapping(value = "hello")
@EnableCircuitBreaker //서비스가 히스트릭스와 리본 라이브러리를 사용한다
@EnableEurekaClient //유레카 서비스 디스커버리 에이전트에 서비스 자신을 등록하고
                    // 서비스 디스커버리를 사용해 원격 서비스의 위치를 '검색'하도록 지정	
	
	//helloRemoteServiceCall 메소드에 대한 호출을 히스트릭스 회로 차단기로 감싼다.
    @HystrixCommand(threadPoolKey="helloThreadPool")
    public String helloReomoteServiceCall(String firstName, String lastName){
        ResponseEntity<String> restExchange = restTemplate.exchange("http://logical-service-id/name/[ca]{firstName}/{lastName}",
                HttpMethod.GET,
                null,
                String.class,
                firstName, lastName);

        return restExchange.getBody();
    }

    @RequestMapping(value = "/{firstName}/{lastName}", method = RequestMethod.GET)
    public String hello(@PathVariable("firstName") String firstName , @PathVariable("lastName") String lastName){
        return helloReomoteServiceCall(firstName,lastName);
    }
```



위 코드에서 처음보는 어노테이션이 있다.

`@EnableCircuitBreaker와 EnableEurekaClient`

웹 서비스가 복잡해지고 마이크로서비스 개념이 도입되면서, 서비스 요청을 안전하게 처리하기 위한 Circuit Breaker의 도입을 고려하는 사이가 많아졌다고 한다.



@EnableEurekaClient는 마이크로서비스 자신을 유레카 서비스 디스커버리 에이전트에 등록하고 서비스 디스커버리를 사용해 코드에서 원격 REST 서비스의 엔드포인트를 검색할 것을 지정한다.



@HystrixComman는 두 가지 작업을 수행하는데,

1. helloRemoteServiceCall 메소드가 호출될 때 직접 호출되지 않고 히스트릭스가 관리하는 스레드 풀에 위임한다. 호출이 너무 오래 걸리면 ( default : 1초 ) 히스트릭스가 개입해 호출을 중단시킨다. => `회로 차단기 패턴의 구현`
2.   히스트릭스가 관리하는 helloTreadPool이라는 스레드 풀을 만든다. helloRemoteServiceCall 메소드에 대한 모든 호출은 이 스레드 풀에서만 발생하며, 수행 중인 다른 원격 서비스 호출과 격리된다.



끝으로, RestTemplate을 사용해 클래스로 호출하려는 서비스의 논리적 서비스 ID를 전달할 수 있다.

```
ResponseEntity<String> restExchange = restTemplate.exchange("http://logical-service-id/name/[ca]{firstName}/{lastName}"
```



내부적으로, RestTemplate 클래스는 유레카 서비스에 접속해 1개 이상의 logical service id서비스 인스턴스에 대한 물리적 위치를 검색한다.

=> 서비스 소비자의 코드는 해당 서비스 위치를 전혀 알 필요가 없다.



또한 RestTemplate 클래스는 넷플릭스의 리본 라이브러리도 사용한다.

클라이언트가 서비스를 호출할 때마다 로드 밸런서를 거치지 않고 모든 서비스 인스턴스를 `라운드로빈`방식으로 호출한다. 

로드 밸런서를 제거해 클라이언트 측에 옮김으로써 어플리케이션 인프라스트럭쳐에서 `장애점(로드밸런서 고장)`하나를 제거하는 셈이다.



##### 번외)

대형 사이트들은 내부/외부적으로 다양한 서비스 자원에 대해 분산처리를 하고 있다.

하나의 웹 서비스 요청을 위해 다양한 내부 서비스 모듈들이 필요하다.



> 그런데 하나의 의존서비스에 문제가 발생한다면?

바로 매출하락으로 이어지는 결과가 발생한다.



##### 장애의 원인

1. 서비의 지연 혹은 과부하 상황
2. 네트워크 문제



##### 장애를 극복하기 위한 방법 - Resiliency (탄력성, 회복성)

##### Resiliency Patterns

- TimeOut (시간만료)
- Circuit Breaker (차단기) : 넷플릭스에서 공개한 오픈소스
- Bulkheads (칸막이 벽)
- Load Shedding (부하분배, 부하제한)
- Fail Fast (빠른 실패)
- HandShakin (상호동의)
- Graceful Degradation (점진적 기능저하)
- Asynchronous Calls (비동기 호출)

...



