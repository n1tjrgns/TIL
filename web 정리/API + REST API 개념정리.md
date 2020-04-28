# API(Application Programming Interface)

##### API 란?

- 응용 프로그램에서 사용할 수 있도록 운영체제나 프로그래밍 언어가 제공하는 기능을 제어할 수 있게 만든 인터페이스
- 데이터와 기능의 집합을 제공해 컴퓨터 프로그램간 상호작용을 촉진하며, 서로 정보를 교환 가능하도록 함

-----

##### REST API(Representational State Transfer)

##### REST란?

- REST 기반으로 서비스 API를 구현한 것


- WWW과 같은 분산 하이퍼미디어 시스템을 위한 소프트웨어 아키텍처의 한 형식
  - 기본적으로 웹의 기존 기술과 HTTP 프로토콜을 그대로 활용하기 때문에 웹의 장점을 최대한 활용할 수 있는 아키텍처 스타일
  - 네트워크 상에서 Client와 Server 사이의 통신 방식 중 하나
- 자원을 정의하고 자원에 대한 주소를 지정하는 방법 전반을 일컫음
  - 자원 : 해당 소프트웨어가 관리하는 모든 것, ex) 문서, 그림, 데이터 등
  - 자원의 표현 : 그 자원을 표현하기 위한 이름, ex)DB의 학생 정보가 자원일 때, 'students'를 자원의 표현으로 정한다.
- 데이터가 요청되어지는 시점에서 자원의 상태(정보)를 전달한다 => 일반적으로 JSON 혹은 XML을 통해 주고받는다.
- REST 원리를 따르는 시스템을 **RESTful** 이란 용어로 지칭됨



**HTTP URI(Uniform Resource Identifier)를 통해 자원을 명시하고, Http Method(post, get, put, delete)를 통해 해당 자원에 대한 CRUD Operation을 적용하는 것을 의미**

- 자원 기반 구조 설계 중심에 Resource가 있고 Http Method를 통해 Resource를 처리하도록 설계된 아키텍쳐



##### 디자인 가이드

1. URI는 정보의 자원을 표현해야 한다.
2. 자원에 대한 행위는 HTTP Method(get, post, put, delete)



##### 부연설명

1. 회원정보를 가져오는 URI에 있을 때, GET/member/select/1 (x) => GET/member/show/1 (o)
   - select 같은 행위에 대한 표현이 들어가면 안됨.
2. '/' 구분자는 계층 관계를 나타냄
   - http://restapi.example.com/animals/mammals/whales => URI 경로 마지막에는 '/' 사용 x
3. 불가피하게 긴 경로는 '-'를 사용해 가독성을 높일 수 있음
4. '_' 밑줄은 사용하지 않음
5. 경로는 소문자를 사용
6. 파일 확장자는 포함 x



##### REST의 장단점

- 장점
  - HTTP 프로토콜을 그대로 사용해 별도의 인프라를 구축 할 필요가 없음
  - HTTP 프로토콜의 표준을 최대한 활용해 여러 추가적인 장점을 함께 가져갈 수 있게 해줌
  - HTTP 프로토콜에 따르는 모든 플랫폼에서 사용가능
  - Hypermedia Api의 기본을 충실히 지키면서 범용성을 보장
  - REST API 메시지가 의도하는 바를 명확하게 나타내므로 의도하는 바를 쉽게 파악할 수 있음
  - 여러가지 서비스 디자인에서 생길 수 있는 문제를 최소화함
  - 서버와 클라이언트의 역할을 명확하게 분리
- 단점
  - 표준이 존재하지 않는다.
  - 사용할수 있는 메소드가 4가지 밖에 없다.
    - HTTP Method 형태가 제한적이다.
  - 구형 브라우저가 아직 제대로 지원을 못해주는 부분이 존재
    - PUT, DELETE를 사용하지 못함
    - pushState를 지원하지 않음



##### REST가 필요한 이유

- 어플리케이션 분리 및 통합
- 다양한 클라이언트의 등장
- 최근의 서버 프로그램은 다양한 브라우저와 Android, IOS의 모바일 디바이스에서도 통신할 수 있어야 함
  - 이러한 멀티 플랫폼에 대한 지원을 위해 서비스 자원에 대한 아키텍처를 세우고 이용하는 방법을 모색한 결과 REST에 관심을 가지게 됨



##### REST 특징

- Server-Client(서버-클라이언트 구조)

  - 자원이 있는 쪽이 Server, 자원을 요청하는 쪽이 Client가 된다.

  - REST Server: API를 제공하고 비즈니스 로직 처리 및 저장을 책임진다.

  - Client: 사용자 인증이나 context(세션, 로그인 정보) 등을 직접 관리하고 책임진다.
    서로 간 의존성이 줄어든다.

  - 

- Stateless(무상태)

    - HTTP 프로토콜은 Stateless Protocol이므로 REST 역시 무상태성을 갖는다.

    - Client의 context를 Server에 저장하지 않는다.
        즉, 세션과 쿠키와 같은 context 정보를 신경쓰지 않아도 되므로 구현이 단순해진다.

    - Server는 각각의 요청을 완전히 별개의 것으로 인식하고 처리한다.

    - 각 API 서버는 Client의 요청만을 단순 처리한다.
        즉, 이전 요청이 다음 요청의 처리에 연관되어서는 안된다.
        물론 이전 요청이 DB를 수정하여 DB에 의해 바뀌는 것은 허용한다.

    - Server의 처리 방식에 일관성을 부여하고 부담이 줄어들며, 서비스의 자유도가 높아진다.

- Cacheable(캐시 처리 가능)

    - 웹 표준 HTTP 프로토콜을 그대로 사용하므로 웹에서 사용하는 기존의 인프라를 그대로 활용할 수 있다.

        -  즉, HTTP가 가진 가장 강력한 특징 중 하나인 캐싱 기능을 적용할 수 있다.			

    - 대량의 요청을 효율적으로 처리하기 위해 캐시가 요구된다.

    -  캐시 사용을 통해 응답시간이 빨라지고 REST Server 트랜잭션이 발생하지 않기 때문에 전체 응답시간, 성능, 서버의 자원 이용률을 향상시킬 수 있다.

- Layered System(계층화)

  - Client는 REST API Server만 호출한다.

  - REST Server는 다중 계층으로 구성될 수 있다.

  - API Server는 순수 비즈니스 로직을 수행하고 그 앞단에 보안, 로드밸런싱, 암호화, 사용자 인증 등을 추가하여 구조상의 유연성을 줄 수 있다.
    또한 로드밸런싱, 공유 캐시 등을 통해 확장성과 보안성을 향상시킬 수 있다.
    PROXY, 게이트웨이 같은 네트워크 기반의 중간 매체를 사용할 수 있다.
    Code-On-Demand(optional)
    Server로부터 스크립트를 받아서 Client에서 실행한다.반드시 충족할 필요는 없다.

- Uniform Interface(인터페이스 일관성)
  - URI로 지정한 Resource에 대한 조작을 통일되고 한정적인 인터페이스로 수행한다.
  -  HTTP 표준 프로토콜에 따르는 모든 플랫폼에서 사용이 가능하다.
  - 특정 언어나 기술에 종속되지 않는다.



https://gmlwjd9405.github.io/2018/09/21/rest-and-restful.html

https://meetup.toast.com/posts/92

https://gmlwjd9405.github.io/2018/09/21/rest-and-restful.html