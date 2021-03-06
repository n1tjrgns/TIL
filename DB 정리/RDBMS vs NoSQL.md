## RDBMS vs NoSQL

>  RDB를 써야할까? NoSql을 사용해야할까? NoSql은 어떨때 사용하는걸까?

궁금증으로부터 출발했다.

RDBMS와 NoSQL에 대해서 알아보자.



#### RDB(Relational Database)

![image-20201127161839388](../AppData/Roaming/Typora/typora-user-images/image-20201127161839388.png)

**특징**

- 구조화된 쿼리 언어(Structured Query Language)
- 정해진 `데이터 스키마`에 따라 DB 테이블에 저장된다.
- 관계를 통해 연결된 여러 테이블에 분산된다.
- 관계형 데이터베이스로 대표적으로 MySql, Oracle, PostgreSql 등이 있다.



**장점**

각 테이블에 정의된 구조(필드명, 데이터 유형) 가 있기 때문에 스키마를 준수하지 않는 레코드는 추가할 수 없다.

관계에 의해 데이터를 여러 테이블에 나누어 저장하기 때문에 `중복 데이터가 없다.`

- 복잡한 형태의 쿼리(join)가 가능하다.



**단점**

- 대량의 데이터 입력, 조회시 성능이 저하된다.

- 확장성이 좋지 않다.

  성능을 높이기 위해서 H/W를 고성능으로 교체해야한다. -> 돈이 많이 든다.



----

#### NoSql (NotOnlySql)

![image-20201127162047985](../AppData/Roaming/Typora/typora-user-images/image-20201127162047985.png)

**특징**

- 대표적으로 mongoDB, Redis, hBase 등이 존재

- RDB의 확장성 이슈를 해결하기 위해 나온 모델

  분산 컴퓨팅이 가능해 확장성이 좋다. (Scale Out)

- 스키마가 존재하지 않아 자유롭게 데이터 관리가 가능.

  관계가 존재하지않는다.

- `key - value` 방식으로 데이터 관리

  



**장점**

- 데이터 분산처리에 용이
- 수직, 수평적 확장이 가능(샤딩)
- 속도가 빠르다(`데이터를 컬렉션 형태로 저장`하기 때문에).
- 스키마가 존재하지 않기 때문에 유연하다.



**단점**

- 데이터가 여러 컬렉션에 중복되어 있기 때문에, 수정해야하는 경우 `모든 컬렉션을 update `해줘야한다.
- 스키마가 존재하지 않기 때문에 중복된 데이터가 있을 수 있다.





#### RDB를 사용해야 하는 경우

- 데이터의 `정확성`을 보장해야 하는 경우



#### NoSql을 사용해야 하는 경우

- 데이터 형식이 다양하고, 정확성 보다는 `데이터의 양이 중요`한 경우
- 정확한 데이터 구조를 알 수 없거나 / 확장 될 수 있는 경우
- DB를 수평적으로 확장해야 하는 경우 (`Big Data`)

