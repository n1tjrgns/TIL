# JPA - 엔티티 매핑

1. ##### @Entity

JPA를 사용해서 테이블과 매핑할 클래스에는 @Entity 어노테이션을 붙인다. @Entity가 붙은 클래스는 JPA가 관리하는 것으로 엔티티라 부른다.

##### 1.1 속성

- name : JPA와 사용할 엔티티 이름을 지정, 사용하지 않을 경우 클래스이름이 그대로 적용

##### 1.2 주의사항

- 기본 생성자 필수
- final, enum, interface, inner 클래스는 사용할 수 없음
- 저장할 필드에 final 사용할 수 없음

그리고 기억해야할 것은 자바에서는 기본 생성자가 없는 경우 자동 생성해주지만, 만약 파라미터가 있는 생성자가 클래스에 존재할 경우 기본 생성자를 자동으로 생성해주지 않기 때문에 직접 작성해줘야한다.



2. ##### @Table

@Table 어노테이션은 엔티티와 매핑할 테이블을 지정하고, 생략시 매핑한 엔티티 이름을 테이블 이름으로 사용한다.

##### 2.1 속성

- name : 매핑할 테이블 이름
- catalog : catalog 기능이 있는 데이터베이스에서 catalog를 매핑
- schema : schema 기능이 있는 데이터베이스에서 schema를 매핑
- uniqueConstraint : DDL 매핑 시 유니크 제약조건을 만든다. 스키마 자동생성 기능을 사용해 DDL을 만들 때 사용



3. ##### 데이터베이스 스키마 자동생성

JPA는 데이터베이스 스키마를 자동 생성하는 기능을 제공한다. 클래스의 매핑정보를 보면 어떤 테이블에 어떤 칼럼을 사용하는 지 알 수 있다.

```java
@Entity // 클래스와 테이블 매핑
@Table(name = "MEMBER") // 매핑할 테이블 정보 명시
public class Member {
    
    @Id // 기본키 매핑
    @Column(name = "ID") // 필드를 컬럼에 매핑
    private String id;
    
    @Column(name = "NAME")
    private String userName;
    
    private Integer age; // 매핑 정보가 없을 경우 필드명이 컬럼명으로 매핑
    
    //회원 타입 구분
    @Enumerated(EnumType.STRING)
    private RoleType roleType;
    
    //날짜 타입 매핑
    @Temporal(TemporalType.TIMESTAMP)
    private Date createdDate;
    
    //날짜 타입 매핑
    @Temporal(TemporalType.TIMESTAMP)
    private Date lastModifiedDate;
    
    @Lob // 길이 제한 없음
    private String description;
    
    //getter, setter
}
```



##### 스키마 자동 생성 옵션 설정

persistence.xml에서 아래와 같은 속성 추가시 데이터베이스 테이블을 자동 생성해주고, 콘솔에 DDL을 출력해준다.

```xml
<property name="hibernate.hbm2ddl.auto" value="create" />
<property name="hibernate.show_sql" value="true" />
```

##### hibernate.hbm2ddl.auto의 속성

- create : 기존 테이블을 삭제하고 새로 생성, DROP + CREATE
- create - drop : 어플리케이션을 종료할 때 생성한 DDL을 제거, DROP + CREATE + DROP
- update : DB 테이블과 엔티티 매핑정보를 비교해서 변경사항만 수정
- validate : DB 테이블과 엔티티 매핑정보를 비교해서 차이가 있으면 경고를 남기고 어플리케이션 실행x, DDL 수정 x
- none : 자동생성 기능을 사용하지않으려면 속성을 제거하거나, 유효하지 않은 옵션값을 주면 됨.



##### hbm2ddl 에서 주의해야할점

DDL 을 수정하는 옵션(create, create-drop, update)는 운영서버에서 사용하면 절대 안된다.

오직 개발서버나 개발단계에서만 사용해야 한다.

- 개발 초기단계 : create, update
- 테스트 서버 : update, validate
- 스테이징, 운영서버 : validate, none



##### 이름 매핑 전략 변경

일반적으로 단어와 단어를 구분할 때 자바는 카멜표기법, DB는 언더스코어를 주로 사용한다.

이러한 차이를 매핑하기 위해서는 @Column.name 속성을 명시적으로 사용해 이름을 지정해준다.

```java
@Column(name = "role_type")
String roleType;
```

 persistence.xml에서 속성을 지정하면 자동으로 카멜표기법을 언더스코어 표기법으로 매핑한다.

```xml
<property name="hibernate.ejb.naming_strategy" 							 	     	value="org.hibernate.cfg.ImprovedNamingStrategy" />
```



4. ##### DDL 생성가능

회원 클래스에서 회원 이름이 필수로 입력되고, 10자를 초과하면 안된다는 제약조건을 추가해보자.

```java
@Column(name = "NAME", nullable = false, length = 10 ) // 필수입력(null 허용 x), 길이는 10
private String userName;
```



유니크 제약조건을 만들어주는 @Table의 uniqueConstraints 속성

```java
@Entity
//유니크 제약조건 추가
@Table(name = "MEMBER", uniqueConstraints = {@UniqueConstraint(
			name = "NAME_AGE_UNIQUE",
			columnNames = {"NAME", "AGE"} )})
public class Member {
    ...
}
```



5. ##### 기본키 매핑

   직접할당 : 기본키를 어플리케이션에서 직접할당

   자동생성 : 대리키 사용방식
   - IDENTITY : 기본키 생성을 데이터베이스에 위임, 데이터베이스에 의존
   - SEQUENCE : 데이터베이스 시퀀스를 사용해 기본키를 할당, 데이터베이스에 의존
   - TABLE : 키 생성 테이블을 사용, 데이터베이스에 의존 x 
   - AUTO : 자동으로 기본키 생성




기본키 생성 전략 설정

persistence.xml에 아래와 같이 속성을 추가해줘야 한다.

```xml
<property name="hibernate.id.new_generator_mappings" value="true" />
```



##### 직접할당

기본키를 직접 할당하는 방법은 아래와 같이 클래스 필드를 @Id로 매핑하고, 엔티티를 저장하기 전에 어플리케이션에서 기본키를 직접 할당해주면 된다.

```java
//기본키 직접 할당 전략
@Entity
@Table(name = "MEMBER_DIRECT")
public class MemberDirect {
    
    @Id // 기본키 매핑
    @Column(name = "NAME")
    private String id;
    
    @Column(name = "NAME")
    private String username;
    
    //getter, setter
}

//기본키 직접 할당 사용 코드
for(int i=0; i<5; i++){
    MemberDirect member = new MemberDirect();
    member.setUsername("doubles" + i);
    member.setId("id00" + i); //기본키 직접 할당
    entityManager.persist(member);
    
    System.out.println("DIRECT member id = " + member.getId());
}
```



##### IDENTITY 전략

IDENTITY 전략은 기본키 생성을 데이터베이스에 위임하는 방식으로 주로 MySql, PostgreSQL, SQL Server, DB2에서 사용한다.

회원 클래스를 아래와 같이 작성하고, id 칼럼값을 지정하지 않고 저장하면 콘솔에서 확인할 수 있듯이 자동으로 기본키가 생성되는 것을 확인할 수 있다.

```java
// 기본키 IDENTITY 전략
@Entity
@Table(name = "MEMBER_IDENTITY")
public class MemberIdentity {
    
    @Id // 기본키 매핑
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "NAME")
    private String username;
    
    //getter, setter 생략
    
    //기본키 IDENTITY 전략 사용 코드
    for(int i=0; i<3; i++){
        MemberIdentity member2 = new MemberIdentity();
         member2.setUsername("doubles" + i);
   		 //기본키를 직접 할당하는 코드가 없음
   		 entityManager.persist(member2);
    }
}
```





##### SEQUENCE 전략

유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트. 시퀀스를 지원하는 Oracle, PostgreSQL, DB2, H2에서 사용 가능



```
//Sequence 전략
@Entity
@SequenceGenerator(
	name = "MEMBER_SEQ_GENERATOR", //식별자 생성기 이름
	sequenceName = "MEMBER_SEQ", //매핑할 데이터베이스 시퀀스 이름
	initialValue = 1, //DDL 생성시 사용, 시퀀스 DDL을 생성할 때 처음 시작하는 수
	allocationSize = 1) // 시퀀스 한번 호출에 증가하는 수

public class MemberSequence {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = 					 					"MEMBER_SEQ_GENERATOR")
    private Long id;
    
    @Column(name="NAME")
    private String username;
    
    //getter, setter
}
```







-----

#### 필드와 컬럼 매핑

- 필드와 컬럼 매핑
  - @Column : 컬럼매핑
  - @Enumerated : 자바 enum 타입 매핑
  - @Temporal : 날짜 타입 매핑
  - @Lob : BLOB, CLOB 타입 매핑
  - @Transient : 특정 필드를 DB에 매핑하지 않음
- 기타
  - @Access : JPA가 엔티티에 접근하는 방식을 지정





##### @Column

**속성**

- name 속성 : 필드와 매핑할 테이블의 컬럼 이름, 기본값은 객체의 필드 이름
- nullable 속성 : null 값의 허용여부를 설정, 기본값 true
- unique 속성 : 한 컬럼에 간단한 유니크 제약조건을 걸 때 사용, 만약 두 컬럼이상일 경우 클래스 레벨에서 사용
- length 속성 - 문자 길이 제약조건, String 타입일 경우만 사용, 기본값 255



**주의사항**

자바의 기본타입의 경우 @Column을 사용하면 nullable = false로 지정하는 것이 안전하다.

그 이유는 자바 기본타입의 경우 null값을 입력할 수 없기 때문에



##### @ Enumerated

자바의 enum 타입을 매핑할 때 사용

```
enum RoleType {
    ADMIN, USER
}

@Enumerated(EnumType.STRING)
private RoleType roleType1;

@Enumerated(EnumType.ORDINAL)
private RoleType roleType2;

//enum 사용
member1.setRoleType(roleType1.ADMIN); //DB에 문자 ADMIN으로 저장
member2.setRoleType(roleType2.ADMIN); //DB에 숫자 0으로 저장
```



























<https://doublesprogramming.tistory.com/260>