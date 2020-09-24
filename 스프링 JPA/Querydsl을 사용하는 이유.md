## Querydsl을 사용하는 이유

SQL, JPQL을 코드로 작성할 수 있도록 도와주는 빌더 API

 [공식 레퍼런스 문서](http://www.querydsl.com/static/querydsl/3.4.3/reference/ko-KR/html_single/)

#### SQL, JPQL의 문제점

- 기존 Sql, JPQL은 문자열이기 때문에 Type-safety불가
- 컴파일 시점에 오류를 발견 할 수 없다. 
- JPQL은 정적 쿼리



#### Querydsl의 장점

- IDE의 자동 완성 지원

- 컴파일 시점에 오류 발견 가능
- 문자열이 아닌 코드 작성
- 오픈소스
- 동적 쿼리



#### Querydsl을 사용하는 가장 큰 이유

`동적 쿼리 사용 가능` + `타입 안정성(Type safety)`



