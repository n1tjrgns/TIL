## CORS (Cross-Origin Resource Sharing)

![1558426866361](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1558426866361.png)

slack api 호출하기 위해 ajax로 post 방식으로 요청을 보냈는데 계속해서 위와 같은 에러가 발생했다.

`api 호출방식` 이라고 구글링 후 다른 방식으로 해결하긴 했지만, 위 오류에 대해서도 알고 넘어가려한다.



에러를 보면 `has been blocked by CORS policy` CORS라는 정책에 의해 막혀졌단다.



> CORS가 뭐지??



#### HTTP 접근 제어 (CORS)

처음 전송되는 리소스의 도메인과 다른 도메인으로부터 리소스가 요청될 경우 해당 리소스는 `cross-orgin HTTP` 요청에 의해 요청된다.

나의 경우에는 local에서 테스트를 해볼 때, localhost에서 slack 이라는 서로 다른 도메인에게 리소스를 요청했다. 이 경우 보안 상의 이유로, 브라우저들은 스크립트 내에서 초기화되는 cross-orgin HTTP 요청을 제한하기 때문에 에러가 발생한다.

즉, 자신과 동일한 도메인으로 HTTP 요청을 보내는 것만 가능했다.



> 그럼 외부 API 호출이 불가능한데??

- 당연히 개발자들은 브라우저 벤더사들에게 cross-domain 요청을 할 수 있도록 요청을 했다.



W3C(Web Applications Working Group)은 CORS (Cross-Origin Resource Sharing) 메커니즘을 권하고 있다. 

이는 웹 서버에 보안 cross-domain 데이터 전송을 활성화하는 cross-domain 접근 제어권을 부여한다. 모던 브라우저들은 cross-origin HTTP 요청의 위험성을 완화시키기 위해 (ex: XMLHttpRequest 같은) API 컨테이너 내에서 CORS를 사용한다.



> 어떻게??

CORS 표준은 웹 브라우저가 사용하는 정보를 읽을 수 있도록, 서버에게 알려 줄 수 있는 HTTP 헤더를 추가해줘야 한다.



#### CORS 요청의 종류

CORS 요청은 Simple/Preflight, Credential/Non-Credential의 4가지 조합이 존재한다.

브라우저가 요청 내용을 분석해 4가지 방식중 해당하는 방식을 찾아 서버에 요청을 날리기 대문에, 프로그래머가 목적에 맞는 방식으로 코딩해야한다.



1. ##### Simple Request

서버에 1번 요청하고, 1번 회신 받는다.

- GET, POST, HEAD 중 한가지 방식을 사용한다.
- POST 방식일 경우 Content-type이 아래 셋 중 하나여야 한다.
  - application/x-www-form-urlencoded
  - multipart/form-data
  - text/plain
  - 커스텀 헤더를 전송하지 말아야 한다.
    - 사용가능한 헤더는 Accept, Accept-Language, Content-Languate, Content-Type가 있다.



2. ##### Preflight Request

Simple Request 조건에 해당하지 않으면 브라우저는 Preflight Request 방식으로 요청한다.

- GET, POST, HEAD 외의 다른 방식으로도 요청을 보낼 수 있다.
- application/xml 처럼 다른 Content-type으로 요청을 보낼 수 있다.
- 커스템 헤더를 사용할 수 있다.



Preflight Request는 예비 요청과 본 요청으로 나뉘어 전송한다.

`서버에 예비 요청 > 예비 요청에 응답 > 본 요청 > 본 요청에 응답`

하지만, 위 과정을 개발자가 전부 처리해야 하는 것은 아니다.

Access-Control-계열의 Response Header를 정해주면, OPTIONS 요청으로 오는 예비 요청과 GET, POST, HEAD, PUT, DELETE등으로 오는 본 요청의 처리는 서버가 알아서 처리한다.



3. ##### Credential

HTTP Cookie와 HTTP Authentication 정보를 인식할 수 있게 해주는 요청

요청 시 `xhr.withCredentials = true` 를 지정해서 Credential 요청을 보낼 수 있고, 서버는 Response Header에 반드시 `Access-Control-Allow-Credentaials: true``를 포함해야 하고, Access-Control-Allow-Origin `헤더의 값에는`* `가 오면 안되고 http://foo.examp 같은 구체적 도메인이 와야한다. 



4. ##### Non-Credential

CORS 요청은 기본적으로 Non-Credential 요청이므로 ,  `xhr.withCredentials = true` 를 지정하지 않으면 Non-Credential 요청이 된다.



-----



#### CORS 관련 HTTP Response Headers

서버에서 CORS 요청을 처리할 때 지정하는 헤더



##### Access-Control-Allow-Origin

Access-Control-Allow-Origin 헤더의 값으로 지정된 도메인으로부터의 요청만 서버의 리소스에 접근할 수 있게 한다.

```
Access-Control-Allow-Origin : <origin> | *
```

`<origin>`에는 요청 도메인의 URI를 지정한다.

*는 모든 도메인으로부터의 접근을 허용한다는 뜻이다.

위에서 정리한 3번의 경우에는 *를 사용할 수 없다.



##### Access-Control-Expose-Headers

기본적으로 브라우저에게 노출이 되지 않지만, 브라우저 측에서 접근할 수 있게 허용해주는 헤더를 지정한다.

기본적으로 브라우저에 노출되는 HTTP Response Header는 6가지가 있다.

- Cache-Control
- Content-Language
- Content-Type
- Expires
- Last-Modified
- Pragma



Response Header에 지정하여 회신하면 기본적으로 접근할 수 없었던 헤더정보를 알 수 있게 된다.

```
Access-Control-Expose-Headers: Content-Length, X-My-Custom-Header, X-Another-Custom-Header
```



##### Access-Control-Max-Age

Preflight Request의 결과가 캐쉬에 얼마나 오랫동안 남아있는지 나타낸다.

```
Access-Control-Max-Age: <delta-seconds>
```



##### Access-Control-Allow-Credentials

Request with Credential 방식이 사용될 수 있는지 지정한다.

```
Access-Control-Allow-Credentials: true | false
```



##### Access-Control-Allow-Methods

예비 요청에 대한 Response Header에 사용되며, 서버의 리소스에 접근할 수 있는 HTTP Method 방식을 지정한다.

```
Access-Control-Allow-Methods: <method>[, <method>]*
```



##### Access-Control-Allow-Headers

예비 요청에 대한 Response Header에 사용되며, 본 요청에서 사용할 수 있는 HTTP Header를 지정한다.

```
Access-Control-Allow-Headers: <field-name>[, <field-name>]*
```

-----



#### CORS 관련 HTTP Request Headers

클라이언트가 서버에 CORS 요청을 보낼 때 사용하는 헤더로, 브라우저가 자동으로 지정하며, XMLHttpRequest를 사용하는 개발자가 직접 지정해 줄 필요가 없다.



##### Origin

Cross-site 요청을 날리는 요청 도메인 URI를 나타내며, access control이 적용되는 모든 요청에 Origin 헤더는 반드시 포함된다.

```
Origin: <origin>
//<origin> 은 서버이름(포트포함)만 포함되며 경로 정보는 포함되지 않는다.
```



##### Access-Control-Request-Method

예비 요청을 보낼 때 포함되어, 본 요청에서 어떤 HTTP Method를 사용할 지 서버에 알려준다.

```
Access-Control-Request-Method: <method>
```



##### Access-Control-Request-Headers

예비 요청을 보낼 때 포함되어, 본 요청에서 어떤 Http Header를 사용할지 서버에 알려준다.

```
Access-Control-Request-Headers: <field-name>
```



#### XDomainRequest

XDomainRequest(XDR)은 W3C 표준은 아니며, IE 8, 9 에서 비동기 CORS 통신을 위해 MS에서 만든  객체다.

- setRequestHeader가 없다.
- XDR과 XHR을 구분하려면 obj.contentType을 사용한다.
- http와 https 프로토콜만 가능하다.



[해당 브라우저의 CORS 정보를 확인 사이트 ](https://tools.geekflare.com/) 에서는 해당 사이트의 CORS Information을 확인해 볼 수 있으니, 궁금하면 참고하자.

#### 

#### 마무리

- CORS를 사용하면 AJAX로도 다른 도메인의 자원을 사용할 수 있다.
- CORS를 사용하려면
  - 클라이언트에서 Access-Control-** 류의 HTTP Header를 서버에 보내야 한다.
  - 서버에서도 Access-Control-** 류의 HTTP Header를 클라이언트에 회신하게 되어 있어야 한다.



> AJAX로는 다른 도메인의 자원을 사용할 수 없었던게 아니라, 내가 못한거였다 ㅎㅎㅎㅎㅎ



##### 참고

<https://developer.mozilla.org/ko/docs/Web/HTTP/Access_control_CORS>

<https://homoefficio.github.io/2015/07/21/Cross-Origin-Resource-Sharing/>