



## 김영한 - 스프링부트-JPA-활용-1

타임리프 장점 : natural template -> html 형식을 깨지 않음.

spring.io -> guide에 스프링 예제가 많으니 참고하면 좋음.

resources/static - 순수 정적인 페이지 렌더링



#### 로컬에 h2 설치

h2디비를 설치후 bat/sh 를 실행하면 

```
http://10.0.75.1:8082/login.jsp?jsessionid=419cca829f04e67fd3ea46011489423b
이렇게 뜨는데 localhost로만 바꿔주자, 뒤에 키 값은 유지해야함

localhost:8082/login.jsp?jsessionid=419cca829f04e67fd3ea46011489423b
```



 jdbc:h2:~/jpashop (최소 한번) 

~/jpashop.mv.db 파일 생성 확인 

이후 부터는 jdbc:h2:tcp://localhost/~/jpashop 이렇게 접속 



프로젝트 셋팅 및 jpa 테스트시 #hibernate.dialect' not set h2 에러 해결

```
spring.jpa.hibernate.dialect: org.hibernate.dialect.H2Dialect 
```



h2 mvcc not supported -  H2 1.4.200 버전부터 MVCC 옵션이 제거되었습니다. 



----

#### 설계 및 분석

Address 타입 -> value type 참고

N : N 관계는 운영환경에서는 지양해야한다.

(여기서는 관계를 예제로 보여주기 위해서 있는것)



내장 클래스 @Embeddable 사용

@Setter 는 꼭 필요한 클래스에만 사용하도록 해야한다.



```
@Inheritance(strategy = InheritanceType.SINGLE_TABLE) // 상속 전략
```



1:1 관계인 경우에는 access를 많이하는 엔티티에 fk를 놓는게 좋다고함.

둘 다 상관없긴함



다대다 관계는 joincolumn이 아닌 jointable을 사용해야 함

1:N의 단방향 관계 매핑시에

@JoinColumn을 사용해야한다.

데이터 모델링에서 fk로 조인을 하듯이 joincolumn을 통해서 



- 엔티티를 변경할 때는 Setter 대신 변경 지점이 명확하도록 변경을 위한 비즈니스 메소드를 별도로 제공하자.



```
package com.jpabook.jpashop.domain;

import lombok.Getter;

import javax.persistence.Embeddable;

@Embeddable
@Getter
public class Address { //값 타입의 클래스는 immutable하게 설계되어야함

    private String city;
    private String street;
    private String zipcode;
    
    //new 로 새로 만들어서 값 타입 변경을 방지하기 위해서 jpa 스펙상 protected까지 가능
    protected Address() {
    }
	
	//생성자에서 값을 모두 초기화해서 변경 불가능 클래스를 만드는게 목적
    public Address(String city, String street, String zipcode){
        this.city = city;
        this.street = street;
        this.zipcode = zipcode;
    }
}

```



### 엔티티 설계시 주의사항

#### **가급적 Setter를 지양해라**

- Setter가 모두 열려있으면, 변경 포인트가 너무 많아져서 유지 보수가 어렵다.



#### 모든 연관관계는 지연로딩(LAZY)으로 설정해라(중요)

- 즉시로딩(`Eager`)의 경우 N+1 문제가 발생 할 수 있다.

- 어떤 SQL이 실행될지 추적하기 어렵다.

- 실무에서 모든 연관관계는 지연로딩(`LAZY`)로 설정해야 한다.

- 함께 조회해서 가져오고 싶은 경우 fetch join 또는 엔티티 그래프 기능을 사용해야한다.

- `xxxToOne`어노테이션은 기본값이 Eager이기 때문에 LAZY로 바꿔줘야한다.

  

#### 컬렉션은 필드에서 초기화 하자.

- 필드에서 바로 초기화 하는것이 안전하다.

- `null`에 대해 안전하다.

- 하이버네이트에서는 엔티티를 영속화 할 때, 컬렉션을 감싸서 하이버네이트가 제공하는 내장 컬렉션으로 변경한다. 

  만약 getOrders()처럼 임의의 메소드에서 컬렉션을 잘못 생성하면 하이버네이트 내부 메커니즘에 문제가 발생할 수 있다.

```
Member member = new Member();

System.out.println(member.getOrders().getClass());
em.persist(member);
System.out.println(member.getOrders().getClass());

//출력결과
class java.util.ArrayList
class org.hibernate.collection.internal.PersistentBag
```



#### 테이블, 컬럼명 생성 전략

스프링부트에서 하이버네이트의 기존 네이밍 전략 = `SpringPhysicalNamingStrategy`



스프링 부트 신규 설정(엔티티(필드) -> 테이블(컬럼))

1. 카멜 케이스 -> 언더스코어 (memberId -> member_id)
2. . (점) -> _(언더스코어)
3. 대문자 -> 소문자





#### 양방향 연관관계 메소드 작성

데이터를 양쪽에 저장해야 참조해서 값을 불러올 수 있다.

```
public static void main(String[] args){
        Member member = new Member();
        Order order = new Order();

        member.getOrders().add(order);
        order.setMember(member);
}
```



이런식으로 매번 작성해야 한다면, 번거롭기도하고 실수하기 딱 좋다.

그래서 아예 연관관계 메소드를 작성하자.



```
package com.jpabook.jpashop.domain;

import lombok.Getter;
import lombok.Setter;

import javax.persistence.*;
import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "orders")
@Getter @Setter
public class Order {

    @Id
    @GeneratedValue
    @Column(name = "order_id")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id")
    private Member member;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> orderItems = new ArrayList<>();

    @OneToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @JoinColumn(name = "delivery_id")
    private Delivery delivery;

    private LocalDateTime orderDate; //주문시간

    @Enumerated(EnumType.STRING)
    private OrderStatus status; //주문상태 [ORDER, CANCEL]

    /* 연관관계 메소드*/
    public void setMember(Member member){
        this.member = member;
        member.getOrders().add(this);
    }

    public void addOrderItem(OrderItem orderItem){
        orderItems.add(orderItem);
        orderItem.setOrder(this);
    }

    public void setDelivery(Delivery delivery){
        this.delivery = delivery;
        delivery.setOrder(this);
    }
}

```



----



### 아키텍처

#### 계층형 구조 사용

- controller, web : 웹 계층
- service : 비즈니스 로직, 트랜잭션 처리
- repository : JPA를 직접 사용하는 계층, 엔티티 매니저 사용
- domain : 엔티티가 모여있는 계층, 모든 계층에서 사용



개발 순서 : 서비스, 리포지토리 계층을 개발하고, 테스트 케이스를 작성해서 검증, 마지막 웹 계층에 적용



`extends JpaRepository` 를 상속받아 사용하는 것은 정확히 말하자면

Spring data JPA를 사용하는 것이다.

그래서 이 강의에서는 순수 JPA를 사용해서 구현한다.



SQL과 JPQL의 차이점

sql : 테이블을 대상으로 쿼리 수행

JPQL : 객체를 대상으로 쿼리 수행



opensessioninview



@Transactional(readOnly = true) 리드온리 옵션을 주면

dirty checking이나 그런 부분들을 생략해서 읽기 부분에는 옵션을 주는것이 좋다.

쓰기 부분에 저 옵션을 넣으면 데이터 변경이 없음

default = false

그래서 보통 클래스 위에 readOnly를 주고

나머지 작업이 있는 메소드에는 

`@Transactional을 명시해준다.`





스프링부트에서 별도의 h2디비 설정을 하지 않으면 메모리모드로 돌려서 

jdbc 설정을 별도로 하지 않아도 된다.



```
@NoArgsConstructor(access = AccessLevel.PROTECTED)
```

객체의 생성을 제약함으로써 또다른 생성코드를 발생시키지 않도록 제한하면서 코딩해야한다.



주문 서비스의 주문과 주문 취소 메소드는 `비즈니스 로직이 대부분 엔티티에`있다.

이런식으로 서비스 계층을 단순히 엔티티에 필요한 요청을 위임하는 역할만하고,

엔티티의 비즈니스 로직을 가지고 객체지향의 특성을 활용하는 것을 `도메인 모델 패턴`이라고 한다.

반대로 엔티티에는 비즈니스 로직이 거의 없고, 서비스 계층에서 대부분의 비즈니스 로직을 처리하는 것을 `트랜잭션 스크립트 패턴`이라고 한다.

jpa는 DDD 패턴 스타일



#### jpa에서 동적 쿼리는 어떻게 하는가?

- 검색 기능의 경우 조건에 따라 쿼리가 달라짐

- Querydsl을 사용해라 !!



-----

스프링에서 자동으로 유효성 체크를 지원해줌

```
    @NotEmpty(message = "회원 이름은 필수 입니다.")
    private String name;
```



컨트롤러에서는 @valid를 사용해서 받는다.

```
@PostMapping("/members/new")
    public String create(@Valid MemberForm memberForm){
        
    }
```

없으면

` default message [name]]; default message [회원 이름은 필수 입니다.] ` 이렇게 에러가 찍힌다.



이렇게 bindingresult 라는걸 사용하면

```
public String create(@Valid MemberForm form, BindingResult result){

//에러가 있으면 해당 폼으로 강제로 보내버린다.
if(result.hasErrors()){
            return "members/createMemberForm";
        }
```

valid에서 오류가 생기더라도 그 오류를 담은 상태로 아래 코드가 바인딩되어서 실행된다.

```
<div class="form-group"><label th:for="name">이름</label> <input type="text" th:field="*{name}"
                                                                       class="form-control" placeholder="이름을 입력하세요"
                                                                       th:class="${#fields.hasErrors('name')}? 'form-control fieldError' : 'form-control'">
            <p th:if="${#fields.hasErrors('name')}" th:errors="*{name}">Incorrect date</p></div>
```

화면처리를 추가적으로 해줘야하긴하다.



엔티티는 최대한 핵심로직만 있도록해야한다.

화면을 위해 뿌려질 데이터는 새로 화면로직을 위한 엔티티만 따로 구성해서 사용하자.

----

#### 변경 감지(dirty checking)와 병합(merge)



#### 준영속 엔티티란?

- 영속성 컨텍스트가 더는 관리하지 않는 엔티티를 말한다.
- 객체가 이미 DB에 한번 저장되어 식별자가 존재하는 경우, 기존 식별자를 가지고 있으면 준영속 엔티티로 볼 수 있다.
- jpa가 관리를 하지 않음, `persist 상태가 아니기 때문에` 값을 바꿔도 DB변경이 일어나지 않음



#### 준영속 엔티티를 수정하는 2가지 방법

- 영속성 상태인 객체를 사용하여 값을 저장



```
@Transactional
    public void updateItem(Long itemId, Book param){
        //findItem이 이미 영속성 상태이다.
        Item findItem = itemRepository.findOne(itemId);
        findItem.setPrice(param.getPrice());
        findItem.setName(param.getName());
        findItem.setStockQuantity(param.getStockQuantity());
        
        //set 로직이 끝나면 transactional이 commit된다
        //그럼 flush할때 바뀐값을 dirty checking 해서 최종 값으로 commit을 날림
    }
```



- merge를 사용해서 값을 저장

> merge가 뭐길래?

준영속 상태의 코드를 영속상태로 바꿔벌임



#### 병합 동작 방식

1. merge()가 실행한다.
2. 파라미터로 넘어온 준영속 엔티티의 식별자 값으로 `1차 캐시에서 엔티티를 조회`한다.
3. 만약 1차 캐시에 엔티티가 없으면 DB에서 엔티티를 조회하고, 1차 캐시에 저장한다.
4. 조회한 영속성 엔티티의 값을 채워넣는다. 새로운 값으로
5. 영속 상태인 엔티티를 반환한다.



#### merge의 코드화

```
	@Transactional
    public Item updateItem(Long itemId, Book param){
        //findItem이 이미 영속성 상태이다.
        Item findItem = itemRepository.findOne(itemId);
        findItem.setPrice(param.getPrice());
        findItem.setName(param.getName());
        findItem.setStockQuantity(param.getStockQuantity());
        
        return findItem;
}
```



> 주의사항 !!!

- dirty checking을 사용하면 원하는 속성만 변경할 수 있지만, 병합을 사용 할 경우 모든 속성이 변경된다.

- 병합시 값이 없으면 `null`로 업데이트 해버릴 수 있다.



`그래서 영속성 상태인 객체를 사용하여 값을 저장을 해야한다.`

`또한 dirty checking은 트랜잭션 커밋 시점에 실행된다!!!`



