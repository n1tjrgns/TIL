## JPA 

ORM(Object Relational Mapping) 으로, RDB 데이터 베이스의 정보를 객체지향으로 손쉽게 활용할 수 있도록 도와주는 도구



Object(자바객체)와 Relation(관계형 데이터베이스)둘간의 매핑을 통해 보다 손쉽게 적용할 수 있는 기술을 제공해준다.



쿼리보다 객체에 집중 함으로써, 조금 더 프로그래밍 적으로 활용이 가능하다.



자바 변수는 카멜케이스

디비 변수는 스네이크 케이스를 사용하는 것이 일반적



#### Entity

JPA에서 테이블을 자동으로 생성해주는 기능

`DB Table == JPA Entity`



| Annotation      | 용도                         |
| --------------- | ---------------------------- |
| @Entity         | 해당 Class가 Entity임을 명시 |
| @Table          | 실제 DB 테이블 이름 명시     |
| @Id             | Index primary key 명시       |
| @Column         | 실제 DB column의 이름 명시   |
| @GeneratedValue | PK 식별키 전략 설정          |



#### Repository

따로 쿼리문을 작성하지 않아도 기본적인 CRUD가 가능하게 된다.



#### JpaAuditing





#### API Request와 Response 부분을 따로 정의하는 이유 

- 상황에 맞춰 달라질 수 있는 요청과 응답에 대응하기 쉬워지기 때문에





#### 하드코딩으로 String = "item" 이런 값들은 ENUM 클래스로 관리하여 오타 방지를 하도록 하자.



#### @PostConstruct



#### @Component

- Autowired를 받기위해 Component를 지정해줘야한다는데 이건 무슨 소리지?

- 스프링이 관리를 하기 시작함



#### @SpyBean

- 컨트롤러에 원하는 객체를 주입가능한 테스트 어노테이션



#### @Before

- 테스트케이스를 실행하기전에 우선 실행됨



#### @Modifying

- JPA에서 select 이외의 쿼리를 사용하기 위해 필요한 어노테이션



