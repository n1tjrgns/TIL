## Api 설계를 위한 사전 준비

#### URI 정의

- URI만 보고도 직관적으로 이해할 수 있도록, 최대 2 depth 정도로만 작성

  /dogs

  /dogs/1234

  

- get/set 등 행위를 붙이는 것은 좋지 않은 예다.

  /getDogs => X

  

- 리소스간의 관계 표현 방법
  
  1. 서브 리소스로 표현
  
  /"리소스명"/"리소스id"/"관계가 있는 다른 리소스명"
  
  /dogs/{puppy}/owner/{n1tjrgns} 
  
  
  
  2. 서브 리소스에 관계를 명시
  
  /users/{userid}/likes/devices



 

@GetMapping 같은 어노테이션 위에 @ApiOperation이라는 어노테이션

- `@ApiOperation` - 하나의 엔드포인트 단위로 메타데이터를 표현할 때 사용합니다.

  `value` - 엔드포인트명

  `notes` - 설명

  ex: `@ApiOperation(value = "회원 가입", notes = "회원을 생성합니다.")`



#### 버전관리

- **@GetMapping("/v1.0/codes") 이것처럼 v 로 버전 관리**

```
{servicename}/{version}/{REST URL}
example) api.server.com/v2.0/codes
```



#### 오류 처리는 명확히, 에러 스택은 비공개

```
200 - OK
400 - Bad Request
500 - Internal Server Error
201 - Created
304 - Not Modified
404 - Not Found
401 - Unauthorized
403 - Forbidden
```



#### 페이징

- 페이스북 페이징 API example

```
/record?offset=100&limit=25 // 100번째 레코드에서부터 25개 레코드 출력
```



#### 검색(전역검색, 지역검색)

- 검색은 주로 Http Get에서 Query String에 검색 조건을 정의하는 경우가 일반적
- 하지만 검색조건이 많을 경우 검색 조건의 경계가 모호해짐

`/users?name=cho //이름이 cho인 사람을 검색`

`/users?name=cho&offset=100&limit=25 //이름이 cho인 사람을 검색?? 페이징 처리???`  



그래서 분간을 하기위해 URLEncode `%3d`를 사용해 나타내기도 한다.

`/users?name%3Dcho,region%3Dseoul&offset=100&limit=25`

이런식으로 검색 조건에만 URLEncode를 사용해 표현하고 구분자 `,`를 사용하면 검색 조건은 다른 Query String과 분리된다.

위 검색 조건은 서버에 의해 토큰 단위로 파싱되어야 한다.



또한 검색 범위에 대해 고려 할 필요가 있는데, users 리소스가 정의되어 있을 때 `id='n1tjrgns'`에 대한 전역 검색은 `search`  URI를 사용한다.

**전역검색**

`/search?id=n1tjrgns  or  /search?id%3Dn1tjrgns`



**지역검색** 

`/users?id=n1tjrgns  or  /users?id%3Dn1tjrgns`



#### HATEOAS를 이용한 링크 처리

- 하이퍼미디어의 특징을 사용해 HTTP Response에 다음 Action이나 관계되는 리소스에 대한 HTTP Link를 함께 리턴해준다.
- HATEOAS를 API에 적용시 `Self-Descriptive` 특성이 증대되어 API에 대한 가독성이 증가되는 장점이 있지만, 응답 메세지가 다른 리소스 URI에 대한 의존성을 가지기 때문에 구현이 다소 까다롭다.
- Spring에서 HATEOAS를 지원해주기 때문에 찾아봐야한다.



ex) 페이징 처리시 리턴 할 때 전 후 페이지에 대한 링크 제공

```
{  

   [  

      {  

         "id":"user1",

         "name":"n1tjrgns"

      },

      {  

         "id":"user2",

         "name":"n1tjrgns2"

      }

   ],

   "links":[  

      {  

         "rel":"pre_page",

         "href":"http://xxx/users?offset=6&limit=5"

      },

      {  

         "rel":"next_page",

         "href":"http://xxx/users?offset=11&limit=5"

      }

   ]

}
```



ex) 연관된 리소스에 대한 디테일한 링크 표시

```
{  

   "id":"n1tjrgns",

   "links":[  

      {  

         "rel":"friends",

         "href":"http://xxx/users/terry/friends"

      }

   ]

}
```



#### 스프링 부트 HATEOAS 적용

[스프링 hateoas 예제](https://spring.io/guides/gs/rest-hateoas/)



의존성 추가

```
//메이븐
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-hateoas</artifactId>
</dependency>

//그래들
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-hateoas'
}
```



json 데이터 전달을 위해 라이브러리 추가

```
<dependency>
  <groupId>com.jayway.jsonpath</groupId>
  <artifactId>json-path</artifactId>
  <scope>test</scope>
</dependency>
```





#### RDBMS에 적용 시키기 어려움

DB에서는 여러 컬럼을 묶어 하나의 PK처럼 사용할 수 있다.

하지만 HTTP URI는 `/`에 따라 계층 구조를 가지기 때문에 표현이 매우 부자연스러워진다.



ex) 세대주의 주민번호, 사는 지역, 본인 이름 일 때

`userinfo/{주민번호}/{지역}/{본인 이름}`은 다소 이상한 의미가 부여될 수 있다.



구글에서는 `Alternative Key(AK)`를 사용하는 방법으로 해결했다.

의미를 가지지 않은 `Unique Value`를 Key로 잡아서 DB Table에 Ak필드로 잡아 사용하는 방법이다.

하지만 AK필드를 추가하기 위해 전체적인 DB설계에 대한 변경이 필요하기 때문에, REST를 위해 전체 시스템의 아키텍쳐에 변화를 준다는 점이 있다.



그리하여 `몽고DB`같은 JSON Document를 그대로 넣을 수 있는 디비를 사용하는 것이 좋다.

하나의 도큐먼트를 하나의 REST 리소스로 취급하여 REST 리소스 구조에 매핑하기 수월하다.







-----



### Swagger 

[참고1](https://steemit.com/kr-dev/@igna84/spring-boot-web-swagger), [참고2](https://heowc.tistory.com/23)

- API의 자동 문서화 오픈소스 라이브러리
- 서버가 실행 될 때, Restcontroller 어노테이션을 읽어서 내부 API를 분석하고 HTML 문서를 작성해 서비스를 제공해준다.


