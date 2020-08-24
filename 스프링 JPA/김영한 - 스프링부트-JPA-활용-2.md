## 김영한 - 스프링부트-JPA-활용-2

### JPA 극한의 성능 최적화



aop와 proxy에 대해서도 더 알아야할듯



@RestController = @Controller + @ResponseBody



#### jpa api를 작성 할 때 별도의 dto 클래스를 사용해야 하는 이유

#### V1 코드

```
 @PostMapping("/api/v1/members")
 public CreateMemberResponse saveMemberV1(@RequestBody @Valid Member member) {
     Long id = memberService.join(member);
     return new CreateMemberResponse(id);
 }
 
 @Data
 class CreateMemberResponse {
     private Long id;
     public CreateMemberResponse(Long id) {
     	this.id = id;
     }
 }
```



위와 같은 회원가입 api가 있다고 해보자.

- 등록 V1: 요청 값으로 Member 엔티티를 직접 받는다. 



문제점 

* 엔티티에 프레젠테이션 계층을 위한 로직이 추가된다. 

  엔티티에 API 검증을 위한 로직이 들어간다. (@NotEmpty 등등) 

  실무에서는 회원 엔티티를 위한 API가 다양하게 만들어지는데(소셜로그인),
  한 엔티티에 각각의 API를 위 한 모든 요청 요구사항을 담기는 어렵다. 

  엔티티가 변경되면 API 스펙이 변한다. 

  

결론 

- API 요청 스펙에 맞추어 별도의 DTO를 파라미터로 받는다. 



#### 수정된 V2 코드

```
@PostMapping("/api/v2/members")
    public CreateMemberResponse saveMemberV2(@RequestBody @Valid CreateMemberRequest request){
        Member member = new Member();
        member.setName(request.getName());

        Long id = memberService.join(member);
        return new CreateMemberResponse(id);

    }

    @Data
    static class CreateMemberRequest{

       @NotEmpty
        private String name;
    }
```



-----



조회하고 싶지 않은 필드가 있다면 해당 필드에 @JsonIgnore를 사용해주면 제외된다.

하지만 이것은 1차원적인 방법

매번 줬다뺐다 할 수는 없음



-----

#### API 개발 고급



#### V1

#### XXXToOne 관계 성능 최적화

Order에 갔더니 연관관계인 Member로 감
Member에 갔더니 다시 또 연관관계 Order로 감 == 무한루프프
해결하려면 양방향 연관관계 매핑된 필드에 대해 전부 @JsonIgnore를 해줘야한다.



무한루프는 해결 됐지만, 

```
com.fasterxml.jackson.databind.exc.InvalidDefinitionException: No serializer found for class org.hibernate.proxy.pojo.bytebuddy.ByteBuddyInterceptor and no properties discovered to create BeanSerializer
```


   새로운 에러가 발생함.

```

    @GetMapping("/api/v1/simple-orders")
    public List<Order> ordersV1(){
        List<Order> all = orderRepository.findAllByCriteria(new OrderSearch());

        return all;
    }
```



#### 위의 에러 발생 이유

```
	@ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id")
    private Member member;
```

현재 LAZY 로딩으로 되어있는데, 그로인해 하이버네이트에서 Order에 대한 데이터만 가져온다.

그 때 member 엔티티는

```
private Member member = new ProxyMember(); 
```

위와 같이 가짜 프록시멤버를 가져온다. 이 프록시가 에러에서 튀어나온 `ByteBuddyInterceptor`인데 

```
private Member member = new ByteBuddyInterceptor();
```

처럼 눈에는 안보이지만 저렇게 동작이된다.



Jackson 라이브러리에서 `ByteBuddy`를 처리 할 수 없어서 발생하는 에러이다.

- 이 경우 `Hibernate5Module`라는걸 추가해서 해결 할 수 있음.

```
[{"id":4,"member":null,"orderItems":null,"delivery":null,"orderDate":"2020-05-02T15:30:41.326","status":"ORDER","totalPrice":50000},
```

그럼 위와 같이 data가 나오는데 LAZY로딩이기 때문에 order 이외의 데이터는 `null`인 것을 볼 수 있다.



모듈에 옵션을 줘서 해당 데이터도 뽑을 수는 있다.

하지만, 여전히 엔티티가 그대로 노출되기 때문에 문제점이 많다.



항상 `LAZY Loading을 기본으로 사용`하고, `성능 최적화가 필요한 경우 페치 조인(fetch join)`을 사용해라



----

#### V2 : 엔티티 클래스를 DTO로 변환, N+1 문제 발생 

```
//V2 -> DTO로 변환
    @GetMapping("/api/v2/simple-orders")
    public List<SimpleOrderDto> ordersV2(){
        List<Order> orders = orderRepository.findAllByCriteria(new OrderSearch());
        List<SimpleOrderDto> result = new ArrayList<>();
        for (Order o : orders) {
            SimpleOrderDto simpleOrderDto = new SimpleOrderDto(o);
            result.add(simpleOrderDto);
        }
        return result;
    }

    @Data
    static class SimpleOrderDto {
        private Long orderId;
        private String name;
        private LocalDateTime orderDate;
        private OrderStatus orderStatus;
        private Address address;

        public SimpleOrderDto(Order order){
            orderId = order.getId();
            name = order.getMember().getName();
            orderDate = order.getOrderDate();
            orderStatus = getOrderStatus();
            address = order.getDelivery().getAddress();
        }
    }
```

`N + 1 문제 발생`

- order를 조회 1번한다. -> 결과 row가 2개 이상이면 그 횟수 = N
- order -> member 조회시 LAZY Loading 조회 N번
- order -> delivery 조회시 LAZY Loading 조회 N번

만약 order의 조회 결과가 4개면 최악의 경우 ( 1 + 4 + 4) 번 실행된다.



Lazy Loading은 영속성 컨텍스트에서 쿼리가 있는 경우에는 추가 조회를 하지 않기 때문에 같은 id로 select가 일어나면 추가 조회가 발생하지 않는다.

하지만, 늘 최악의 경우로 생각해서 해야한다.



----

#### V3 : N+1 문제를 fetch join을 사용하여 해결

#### 엔티티 조회 후 dto로 변환해서 조회

fetch join 학습 필요

v2와의 차이점은 fetch join을 해서 가져오냐 아니냐의 차이

#### 

애초에 fetch join으로 조회를 전부 다 해서 가져오기 때문에

LAZY Loading이 발생하지 않는다.

```
 //한 번에 필요한 쿼리를 fetch join으로 다 가져옴.
    public List<Order> findAllWithMemberDelivery() {
        return em.createQuery(
                "select o from Order o" +
                        " join fetch o.member" +
                        " join fetch o.delivery", Order.class
        ).getResultList();
    }
```

**결과** : 쿼리 1번 발생..



----

#### V4 : JPA에서 DTO를 바로 조회

dto를 반환해서 사용하면 불필요한 칼럼은 select 하지 않아도 된다.

하지만 칼럼을 지정해버렸기 때문에 딱딱함 -> repo 재사용성이 떨어진다.

성능 측면에서는 v4가 좋지만, 그때그때 상황에 맞게 v3, v4를 쓰면된다.

내가 원하는 칼럼만 select 해올 수 있음.

쿼리가 복잡하기 때문에 별도의 패키지로 분리해서 순수 조회용임을 명시해주는 것이 좋다.

```
select
        order0_.order_id as col_0_0_,
        member1_.name as col_1_0_,
        order0_.order_date as col_2_0_,
        order0_.status as col_3_0_,
        delivery2_.city as col_4_0_,
        delivery2_.street as col_4_1_,
        delivery2_.zipcode as col_4_2_ 
    from
        orders order0_ 
    inner join
        member member1_ 
            on order0_.member_id=member1_.member_id 
    inner join
        delivery delivery2_ 
            on order0_.delivery_id=delivery2_.delivery_id
```



#### 쿼리 방식 선택 권장 순서

1. 엔티티 -> DTO 변환 (필수)
2. 필요시 fetch join으로 성능 최적화 -> 대부분 이슈 해결
3. 그래도 안되면 DTO를 직접 조회하는 방법 사용
4. 최후 방법은 JPA가 제공하는 native SQL이나 스프링 JDBC Template을 사용해 SQL을 직접 사용



-----

#### 컬렉션 조회 최적화

OneToMany의 최적화 조회

V1 엔티티의 직접 노출



v2 : 엔티티 -> DTO 변환

properties가 없다는 오류 -> getter, setter가 없다.

```
static class OrderDto{

        private Long orderId;
        private String name;
        private LocalDateTime orderDate;
        private OrderStatus orderStatus;
        private Address address;
        private List<OrderItem> orderItems; //추가된 부분
```

Dto로 분리를 했지만 `List<OrderItem>` 아직 래핑된 부분이 남아있다.

이렇게 래핑된 부분도 남아있으면 안된다.

완전히 엔티티에대한 의존을 끊어야한다.



하지만 역시 N+1문제가 발생.

fetch 조인으로 최적화가 필요하다

-----

#### V3 : fetch join으로 최적화



>  어 근데 XXXToOne 관계의 경우 fetch join만 사용하면 최적화가 끝났는데OneToMany의 경우 그렇지 않네 ? 왜 ?

실제 DB의 join을 생각해보면 id에 대한 조회 결과값이 여러 개 일 경우 그에 대한 값이 다 출력된다.



```
{
        "orderId": 4,
        "name": "userA",
        "orderDate": "2020-05-03T14:55:32.297",
        "orderStatus": "ORDER",
        "address": {
            "city": "서울",
            "street": "1",
            "zipcode": "1111"
        },
        "orderItems": [
            {
                "itemName": "JPA1 BOOK",
                "orderPrice": 10000,
                "count": 1
            },
            {
                "itemName": "JPA2 BOOK",
                "orderPrice": 20000,
                "count": 2
            }
        ]
    },
    {
        "orderId": 4,
        "name": "userA",
        "orderDate": "2020-05-03T14:55:32.297",
        "orderStatus": "ORDER",
        "address": {
            "city": "서울",
            "street": "1",
            "zipcode": "1111"
        },
        "orderItems": [
            {
                "itemName": "JPA1 BOOK",
                "orderPrice": 10000,
                "count": 1
            },
            {
                "itemName": "JPA2 BOOK",
                "orderPrice": 20000,
                "count": 2
            }
        ]
    },
```

![image-20200503145949592](./image/image-20200503145949592.png)



fetch join으로 실행된 쿼리를 수행하면

```
select
        order0_.order_id as order_id1_7_0_,
        member1_.member_id as member_i1_5_1_,
        delivery2_.delivery_id as delivery1_2_2_,
        orderitems3_.order_item_id as order_it1_6_3_,
        item4_.item_id as item_id2_3_4_,
        order0_.delivery_id as delivery4_7_0_,
        order0_.member_id as member_i5_7_0_,
        order0_.order_date as order_da2_7_0_,
        order0_.status as status3_7_0_,
        member1_.city as city2_5_1_,
        member1_.street as street3_5_1_,
        member1_.zipcode as zipcode4_5_1_,
        member1_.name as name5_5_1_,
        delivery2_.city as city2_2_2_,
        delivery2_.street as street3_2_2_,
        delivery2_.zipcode as zipcode4_2_2_,
        delivery2_.status as status5_2_2_,
        orderitems3_.count as count2_6_3_,
        orderitems3_.item_id as item_id4_6_3_,
        orderitems3_.order_id as order_id5_6_3_,
        orderitems3_.order_price as order_pr3_6_3_,
        orderitems3_.order_id as order_id5_6_0__,
        orderitems3_.order_item_id as order_it1_6_0__,
        item4_.name as name3_3_4_,
        item4_.price as price4_3_4_,
        item4_.stock_quantity as stock_qu5_3_4_,
        item4_.author as author6_3_4_,
        item4_.isbn as isbn7_3_4_,
        item4_.actor as actor8_3_4_,
        item4_.director as director9_3_4_,
        item4_.dtype as dtype1_3_4_ 
    from
        orders order0_ 
    inner join
        member member1_ 
            on order0_.member_id=member1_.member_id 
    inner join
        delivery delivery2_ 
            on order0_.delivery_id=delivery2_.delivery_id 
    inner join
        order_item orderitems3_ 
            on order0_.order_id=orderitems3_.order_id 
    inner join
        item item4_ 
            on orderitems3_.item_id=item4_.item_id
```

orderitem이 n개만큼 더 있다고 하면 그 갯수만큼 데이터의 양이 증가하게 된다.



그럼 어떻게 해결해???

쿼리에 조건을 추가해준다.



기존 쿼리가

```
public List<Order> findAllWithItem() {
        return em.createQuery(
                "select o from Order o" +
                        " join fetch o.member m" +
                        " join fetch o.delivery d" +
                        " join fetch o.orderItems oi" +
                        " join fetch oi.item i", Order.class)
                .getResultList();
    }
```

이런 형태였다면



distinct를 추가해준다.

DB에서는 distinct가 데이터가 완전히 데이터가 일치해야 중복을 제거하지만,

JPA에서는 distinct가 있을 때 루트 엔티티에(여기서는 Order)같은 id값이 있으면 id값을 제거해준다.

DB에서 중복 제거해서 가져오는 것이 아닌, 가져온 데이터에 대한 한 번 더 필터 과정을 거치는 것이다.

```
public List<Order> findAllWithItem() {
        return em.createQuery(
                "select distinct o from Order o" +
                        " join fetch o.member m" +
                        " join fetch o.delivery d" +
                        " join fetch o.orderItems oi" +
                        " join fetch oi.item i", Order.class)
                .getResultList();
    }
```



#### 컬렉션 fetch join시 치명적인 단점

- 페이징을 할 수가 없다.



```
public List<Order> findAllWithItem() {
        return em.createQuery(
                "select o from Order o" +
                        " join fetch o.member m" +
                        " join fetch o.delivery d" +
                        " join fetch o.orderItems oi" +
                        " join fetch oi.item i", Order.class)
                //.setFirstResult(1) 페이징을 할 수가 없다.
                //.setMaxResults(100)
                .getResultList();
    }
```

페이징 조건 추가시 쿼리에 limit, offset 쿼리를 찾아볼 수 없다.

또한 에러가 발생한다.

```
firstResult/maxResults specified with collection fetch;
applying in memory!
```

fetch join을 사용하면 메모리에서 페이징처리를 해버리기 때문에,

데이터 전부를 어플리케이션으로 가져와서 페이징 처리를하면 

>  펑~



1:N의 경우에는 그래서 fetch join을 사용하면 안된다.

컬렉션 페치 조인은 1개만 사용 할 수 있다. 둘 이상 페치 조인을 사용하면 안된다.

-----

#### V3.1 : 페이징과 한계 돌파

- 컬렉션을 fetch join하면 페이징이 불가능하다.
  - 1 : N 조인이 발생해 데이터가 예측 할 수 없이 증가한다.
  - 데이터가 N을 기준으로 row가 생성되기 때문에

- 하이버네이트가 경고 로그를 남기고 모든 DB 데이터를 일어 메모리에서 페이징을 시도한다. -> `장애로 이어질 수 있다.`



#### 한계 돌파

>  페이징 + 컬렉션 엔티티 조회는 도대체 어떻게?



- 먼저 XXXToOne(OneToOne, ManyToOne) 관계를 모두 fetch join한다. ToOne 관계는 row수를 증가시키지 않기 때문에 페이징 쿼리에 영향을 주지않는다.(fetch join을 계속 걸어도된다.)
- 컬렉션은 지연 로딩으로 조회한다.
- 지연 로딩 성능 최적화를 위해 `hibernate.default_batch_fetch_size`, `@BatchSize`를 적용한다.
  -  `hibernate.default_batch_fetch_size` : 글로벌 설정
  - `@BatchSize` 개별 최적화
  - 이 옵션 사용시 컬렉션이나, 프록시 객체를 한꺼번에 설정한 size만큼 IN 쿼리로 조회한다.



`default_batch_fetch_size: 100 # 쿼리 where in 절 안의 조건 갯수 설정`

```
select
        item0_.item_id as item_id2_3_0_,
        item0_.name as name3_3_0_,
        item0_.price as price4_3_0_,
        item0_.stock_quantity as stock_qu5_3_0_,
        item0_.author as author6_3_0_,
        item0_.isbn as isbn7_3_0_,
        item0_.actor as actor8_3_0_,
        item0_.director as director9_3_0_,
        item0_.dtype as dtype1_3_0_ 
    from
        item item0_ 
    where
        item0_.item_id in (
            ?, ?
        )
```

쿼리가 `where in`의 형태로 날라가기 때문에 발생되는 쿼리의 양이 줄어들었다.



글로벌 설정 말고 detail하게 설정하고 싶다면

적용 할 필드 위에 적용하면 된다.



- 컬렉션에 적용 할 때

```
@BatchSize(size = 1000)
@OneToMany(mappedBy = "order", cascade = CascadeType.All)
private List<orderItem> orderItems = new ArrayList<>();
```

  

- 일반 필드에 적용 할 때

```
클래스명 위에
```



보통은 글로벌로 설정하고 사용함.



- IN 쿼리로 바뀌기 때문에 쿼리 호출 수가 `1 + N`에서 `1 + 1`로 최적화 된다.
- 조인보다 DB 데이터 전송량이 최적화 된다.
- 쿼리 호출 수가 약간 증가 할 수는 있다.
  - 쪼개서 호출 하기 때문에
- 컬렉션 fetch join은 페이징 불가능 하지만 이 방법은 페이징 가능하다.



#### 결론

- ToOne 관계는 fetch join해도 페이징에 영향을 주지 않기 때문에, 쿼리수를 줄이고, 나머지는 `hibernate.default_batch_fetch_size`로 최적화 하자.



> 사이즈 크기는 몇으로 설정하는게 좋을까?

100~ 1000개 사이를 선택하자.

어플리케이션에서는 100~ 1000이든 결국 전체 데이터를 로딩해야 하므로 메모리 사용량은 같다.

다만 DB에 부하를 줄 수 있기 때문에 사이즈를 조절하는 것이다.



----



#### V4 : JPA에서 DTO 직접 조회

Query: 루트 1번, 컬렉션 N 번 실행 

ToOne(N:1, 1:1) 관계들을 먼저 조회하고, ToMany(1:N) 관계는 각각 별도로 처리한다. 

이런 방식을 선택한 이유는 다음과 같다. 

ToOne 관계는 조인해도 데이터 row 수가 증가하지 않는다.

 ToMany(1:N) 관계는 조인하면 row 수가 증가한다. 

row 수가 증가하지 않는 ToOne 관계는 조인으로 최적화 하기 쉬우므로 한번에 조회하고, ToMany 관계 는 최적화 하기 어려우므로 findOrderItems()같은 별도의 메서드로 조회한다.



-----

#### V5 : JPA에서 DTO 조회 최적화(원하는 컬럼만 조회)

쿼리 : 루트 1번, 컬렉션 1번

ToOne 관계를 먼저 조회하고, 여기서 얻은 식별자 orderId ToMany 관계인 orderItem을 한꺼번에 조회

Map을 사용해 매칭 성능을 O(1)로 최적화



-----

#### V6 : 플랫 데이터 최적화

쿼리는 1번이다.

하지만 조인으로 인해 DB에서 어플리케이션에 전달하는 데이터에 중복 데이터가 추가되므로 상황에따라 v5보다 더 느릴 수 있다.

어플리케이션에서 추가 작업이 크다

페이징 불가능



----



#### 총 정리

- 엔티티 조회

  - 엔티티를 조회 후 그대로 반환 : V1

  - 엔티티 조회 후 DTO로 변환 : V2

  - fetch join으로 쿼리 수 최적화 : V3

  - 컬렉션 페이징과 한계 돌파 : V3.1

    - 컬렉션은 fetch join시 페이징을 불가능

    - ToOne 관계는 fetch join으로 쿼리 수 최적화

    - 컬렉션은 fetch join 대신 지연 로딩을 유지하고, `hibernate.default_batch_fetch_size`, `@BatchSize`로 IN 절을 활용해 최적화



- DTO 직접 조회
  - JPA에서 DTO 직접 조회 : V4
  - 컬렉션 조회 최적화 : 1 : N 관계인 컬렉션은 IN 절을 활용해 메모리에 미리 조회해서 최적화 : V5
  - 플랫 데이터 최적화 : JOIN 결과를 그대로 조회 후 어플리케이션에서 원하는 모양으로 직접 변환 : V6



#### 권장 순서

1. 엔티티 조회 방식으로 우선 접근
   1. fetch join으로 쿼리 수를 최적화
   2. 컬렉션 최적화
      	1. 페이징 필요 : `hibernate.default_batch_fetch_size`, `@BatchSize` 로 최적화
       	2. 페이징 필요 X -> 페치 조인 사용
2. 엔티티 조회 방식으로 해결이 안되면 DTO 조회 방식 사용
3. DTO 조회 방식으로 해결이 안되면 NativeSQL or 스프링 JdbcTemplate 사용



엔티티는 직접 캐싱(Reddis)을 하면안된다.

- 영속성 컨텍스트가 관리하기 때문에
- DTO에 넣어서 캐시해야함.

----

#### OSIV와 성능 최적화

- Open Session In View : 하이버네이트
- OSIV라 부름



어플리케이션을 실행하면 아래와 같은 기본 warning이 발생한다.

```
WARN 16856 --- [  restartedMain] JpaBaseConfiguration$JpaWebConfiguration : spring.jpa.open-in-view is enabled by default. Therefore, database queries may be performed during view rendering. Explicitly configure spring.jpa.open-in-view to disable this warning
```

여기서 `spring.jpa.open-in-view`가 OSIV를 일컫는 옵션이며, 기본값은 `true`다.



#### OSIV옵션 ON 일 때

![image-20200506185204055](./image/image-20200506185204055.png)



#### OSIV 전략

- 기본 전략은 최초 DB 커넥션 시작 시점부터 API 응답이 끝날 때 까지 영속성 컨텍스트와 DB 커넥션을 유지한다.

> 그렇다면 반환 시점은?



- 요청에 대한 응답이 끝날 때 까지 커넥션이 살아있다.
- 그로인해 view template과 api 컨트롤러에서 지연로딩이 가능했다.
- 지연 로딩은 영속성 컨텍스트가 살아있어야 가능하고, 영속성 컨텍스트는 기본적으로 DB 커넥션을 유지한다. (장점)



하지만,

이 전략은 너무 오랜시간동안 DB 커넥션 리소스를 사용하기 때문에, 실시간 트래픽이 중요한 어플리케이션에서는 커넥션이 모자랄 수 있다.

- `장애로 이어질 수 있다.`

- ex) 외부 API를 호출했는데 대기 시간 만큼 커넥션 리소스를 반환하지 못하고, 유지해야한다.



----

#### OSIV 옵션 OFF 일 때

![image-20200506191441936](./image/image-20200506191441936.png)

`spring.jpa.open-in-view : false `



- OSIV 옵션을 false로 하면 `트랜잭션이 끝나는 시점에 영속성 컨텍스트도 닫고, DB 커넥션도 반환한다.`
- 커넥션 리소스를 낭비하지 않는다.
- LAZY Loading이 끊기기 때문에 트랜잭션 안으로 해당 코드를 넣어야하는 단점이 있다.



그 동안 LAZY Loading을 사용해서 작성한 코드를 OSIV 옵션 false로 수정 후 API를 호출해보자.

```
"status": 500,
"error": "Internal Server Error",
"message": "org.hibernate.LazyInitializationException: could not initialize proxy [jpabook.jpashop.domain.Member#1] - no Session"
```



```
@GetMapping("/api/v1/orders")
    public List<Order> orderV1() {
        List<Order> all = orderRepository.findAllByCriteria(new OrderSearch());
        for (Order order : all) {
            order.getMember().getName();
            order.getDelivery().getAddress();

            //강제 초기화
            List<OrderItem> orderItems = order.getOrderItems();
            for (OrderItem orderItem : orderItems) {
                orderItems.stream().forEach(o->orderItem.getItem().getName());
            }
        }
        return all;
    }
```

v1 을 예제로 들자면,

`order.getMember().getName();`

- `getName()`에서 프록시를 초기화해야하는데, 초기화를 할 수가 없다.

- controller에서는 이미 영속성 컨텍스와, DB 커넥션이 끝났기 때문에



> 그럼 어떻게해?

1. 트랜잭션 안에서 Lazy Loading 수행
2. fetch join 사용

3. command와 Query를 분리



OrderService : 핵심 비즈니스 로직

OrderQueryService : 화면이나 API에 맞춘 서비스 (주로 읽기 전용 트랜잭션 사용)



#### 꿀팁

- 실시간 고객서비스 성격의 API는 OSIV를 끄고, ADMIN 페이지 같은 커넥션을 많이 사용하지 않는 곳에서는 OSIV를 켠다.

- ORM 표준 13장 참고



----

#### Spring Data Jpa

- 결국 스프링 데이터 JPA는 JPA를 사용해서 이런 기능을 제공할 뿐이기 때문에, JPA 자체를 잘 이해하고 사용하는것이 중요하다.

- 실무에서 직접 쓰기에는 제약사항이 있기 때문에, 그 한계를 알고 사용해야한다.

