## [MSA] API Gateway - (3)

저번 글에 이어서 Client1을 작성해보자



간단하게 Request를 받을 컨트롤러만 작성하여 데이터를 확인해보자.



#### Client1Controller

```java
@Slf4j
@RestController
@RequestMapping("/client1")
public class Client1Controller {

    @GetMapping(value = "/ping")
    public Mono<String> getData(ServerHttpRequest request, ServerHttpResponse response){
        log.info("getData Method");
        HttpHeaders headers = request.getHeaders();

        headers.forEach((k,v) ->{
            System.out.println(k + " : " + v);
        });

        ResponseCookie.ResponseCookieBuilder builder = ResponseCookie.from("client1","client1CookieValue");
        ResponseCookie cookie = builder.build();
        response.addCookie(cookie);
        Mono<String> data = Mono.just("Hello from Reactive getData Method");
        return data;
    }
}
```

컨트롤러를 구현했으니 이제 제대로 라우팅이 되는지 실행해서 확인해보자.



#### 테스트

`http://localhost:8080/client1/ping`

![client 200 ok return](client 200 ok return.JPG)

8080 서버로 요청을 보냈는데, 8081 서버에서 응답이 제대로 온 것으로 보아 라우팅이 성공적으로 진행됐다.

`Mono.just("Hello from Reactive getData Method");` Mono에 담은 데이터가 body에 담겨 응답값으로 전달되었다. 



> Mono가 정확히 뭘까? Flux와 함께 다음 포스트에 정리하자.



#### Client1Controller log

![client 200 ok log](client 200 ok log.JPG)

컨트롤러 헤더에 담긴 key, value 값이 보여진다.



> 그럼 저번에 작성한 Filter는 제대로 우선순위에 맞게 동작이 되었을까?



#### 게이트웨이 로그

![gateway 200 ok filter log](gateway 200 ok filter log.JPG)

**필터 시작**

```
BeforeOrderedFilter - BeforeFilter - CommonFilter - ClientFilter 
```



**필터 종료**

```
ClientFilter - CommonFilter - BeforeFilter - BeforeOrderedFilter
```

역순으로 잘 종료된 모습을 확인 할 수 있다.



게이트 웨이를 구현함으로써, Request를 단일화 할 수 있고, 공통 로직(인증/인가)를 한 번만 처리하면 된다고하였다.

Mono, Flux에 대해 학습하고 인증 과정까지 구현해보자.

