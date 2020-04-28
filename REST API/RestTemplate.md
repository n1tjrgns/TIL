## RestTemplate

스프링 프레임워크에서는 REST 서비스의 Endpoint를 호출 할 수 있도록 `동기, 비동기` REST Client를 제공한다.



#### RestTemplate

- 스프링 3.0 부터 지원하며, REST API 호출이후 응답을 받을 때 까지 기다리는 동기 방식이다.

- 여러 Template 클래스와 동일한 원칙에 따라 설계되어 단순한 방식의 호출로 복잡한 작업을 쉽게 하도록 제공한다. (HTTP 프로토콜의 메소드에 맞는 메소드 제공)



| 메소드          | HTTP    | 설명                                                         |
| --------------- | ------- | ------------------------------------------------------------ |
| getForObject    | GET     | 주어진 URL 주소로 HTTP GET 메소드 객체 결과를 반환받는다.    |
| getForEntity    | GET     | 주어진 URL 주소로  HTTP GET 메소드 결과를 ResponseEntity로 반환 받는다 |
| postForLocation | POST    | POST 요청을 보내고 결과로 헤더에 저장된 URI를 반환받는다     |
| postForObject   | POST    | POST 요청을 보내고 객체로 결과를 반환받는다                  |
| postForEntity   | POST    | POST 요청을 보내고 결과로 ResponseEntity로 반환받는다        |
| delete          | DELETE  | 주어진 URL 주소로 HTTP DELETE 메소드를 실행한다.             |
| headForHeaders  | HEADER  | 헤더의 모든 정보를 얻을 수 있으면 HTTP HEAD 메소드를 사용한다 |
| put             | PUT     | 주어진 URL 주소로 HTTP PUT 메소드를 실행한다                 |
| patchForObject  | PATCH   | 주어진 URL 주소로 HTTP PATCH 메소드를 실행한다               |
| optionsForAllow | OPTIONS | 주어진 URL 주소에서 지원하는 HTTP 메소드를 조회한다          |
| exchange        | any     | HTTP 헤더를 새로 만들 수 있고 어떤 HTTP 메소드도 사용가능하다 |
| execute         | any     | Request/Response콜백을 수정 할 수 있다.                      |







참고

 https://advenoh.tistory.com/46 