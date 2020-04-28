## Spring Data JPA

### JPA

#### Cascade

- 엔티티의 상태 변화를 전파 시키는 옵션



> 엔티티의 상태??

- transient : JPA가 모르는 상태
- Persistent : JPA가 관리중인 상태(1차 캐시, Dirty Checking, Write, Behind,,) - 코드 상에서 .save 와 같은 행동을 취해야 Jpa가 관리하는 Persistent 상태가 된다.
- Detached : JPA가 더이상 관리하지 않는 상태
- Removed : JPA가 관리하긴 하지만 삭제하기로 한 상태



#### Fetch (성능에 영향)

- 연관관계 엔티티를 어떻게 가져올 것인가
- 지금(Eager), 나중에(Lazy)
- @OneToMany의 기본값 = Lazy
- @ManyToOne의 기본값 = Eager



EntityGraph로 각 칼럼별 fetch 전략을 설정 할 수 있음.

----

### DATA JPA

#### JpaRepository<Entity, Id> 인터페이스

- @Repository가 없어도 빈으로 등록해줌.



#### @EnableJpaRepositories

스프링의 경우는 main 클래스에 해당 어노테이션이 있어야하지만, 부트는 생략가능



ImportBeanDefinitionRegistrar 인터페이스로 프로그래밍을 통해 빈으로 주입이 가능.



-----

### 핵심

데이터베이스의 패러다임과 자바의 패러다임간의 불일치를 ORM을 통해 해결

ORM

JPA 사용법(엔티티작성, 벨류 타입, 관계 맵핑)

JPA 특징(엔티티 상태 변화, Cascade, Fetch, 1차 캐시)

#### 주의할점

- 반드시 발생 쿼리 확인
- 팁
  - logging.level.org.hibernate.SQL=degub
  - logging.level.org.hibernate.type.descriptor.sql=trace



----

#### Projection

엔티티의 일부 데이터만 가져오기

```
select * from user; 가 아닌 select id, email user; 같은 형태
```

 한 테이블에 컬럼이 40개 이상인가.. 그정도가 되면 성능상 차이가 있다고 Java Persistence with Hibernate 책에서 읽은 기억 





#### 스프링 @Transactional

모든 Repository는 실질적 구현체 = SimpleJpaRepository에 이미 Transactional이 적용이 되어있어 Transactional 모드로 동작한다.



RuntimeException, error 발생시 -> 롤백

CheckedException 발생시 -> 롤백 안함



----

#### Auditing

엔티티의 변경이 일어났을 때, 언제, 누구에 의해 변경됐는지를 기록하는 기능



1. 메인에 @EnableJpaAuditing 추가

2. 엔티티 클래스에 @EntityListeners(AuditingEntityListener.class) 추가

3. AuditorAware 구현체 구현

4. @EnableJpaAuditing에 (auditorAwareRef = "accountAuditAware") 추가





Audit 기능을 대체할 수 있는 기능 = @Pre~ , @Post~



----



#### 마무리



Spring Jpa를 학습하고 Jpa를 좀 더 효율적으로 사용하도록 하기 위한 Spring Data Jpa



ORM - 단순히 매핑만 하는게 아니고 패러다임의 mismatch를 해결하기 위함

JPA를 알기위해 필수적인 기능

- Entity 칼럼 속성 설정
- 관계 맴핑 1:N, N:1
- 엔티티의 상태변화 (Transient, Persistent, Detached)
- Fetch 모드

- Repository가 만들어지는 원리





----

#### JPA insert

- jpa는 insert할 때 기본적으로 모든 칼럼에 대해 insert를 한다.
- 그로인해 NOT NULL로 설정된 칼럼은 기본값으로 Null 을 insert하기 때문에 에러가 발생할 수 있다.

- 쿼리에서 제외함으로써 DB에 지정된 default값으로 사용 할 수 있는다

  ```
  @Column(insertable=false, updatable=false)
  private String name;
  ```

  위와 같이 insertable, updatable 옵션을 false로 주면 insert 대상에서 제외된다.

  default 옵션은 true다.





비트 연산

1. selft
2. google
   4. naver



가입 한 

