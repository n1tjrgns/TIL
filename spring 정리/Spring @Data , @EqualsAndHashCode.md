# Spring @Data , @EqualsAndHashCode

이번에 확인해볼 부분은 Lombok 라이브러리에서 제공하는 어노테이션이다.



@Data, @EqualsAndHashCode를 보기전에

우선, @Getter 와 @Setter는 각각 접근자와 설정자 메소드를 작성해주는 Lombok  어노테이션으로 유명하다.

또한 생성자를 자동 생성해주는 Lombok 어노테이션에는 

- @**NorgsConstructor** : 파라미터가 없는 기본 생성자 생성
- @**AllArgsConstructor** : 모든 필드 값을 파라미터로 받는 생성자를 만들어 준다.
- @**RequiredArgsConstructor** : final이나 @NonNull 인 필드 값만 파라미터로 받는 생성자를 만들어 준다.



toString() 메소드를 작성하는 일도 @**ToString**어노테이션만 붙여주면 자동으로 생성해 준다

@ToString(exclude = "id")와 같이 exclude를 사용하면 toString() 결과에서 id를 제외시킬 수 있다.

```
//사용할 때
User user = new User();
user.setId("lol");
sysout(user);

//결과
User(id=lol) 식으로 출력이 된다.
```



-----

##### @**EqualsAndHashCode**

##### equals, hashCode 자동 생성

- **equals** :  두 객체의 내용이 같은지, 동등성(equality) 를 비교하는 연산자
- **hashCode** : 두 객체가 같은 객체인지, 동일성(identity) 를 비교하는 연산자



자바 bean에서 동등성 비교를 위해 equals와 hashcode 메소드를 오버라이딩해서 사용하는데,

@**EqualsAndHashCode**어노테이션을 사용하면 자동으로 이 메소드를 생성할 수 있다.

callSuper 속성을 통해 eqauls와 hashCode 메소드 자동 생성 시 부모 클래스의 필드까지 감안할지의 여부를 설정할 수 있다.



**@EqualsAndHashCode(callSuper = true)**로 설정시 부모 클래스 필드 값들도 동일한지 체크하며, false(기본값)일 경우 자신 클래스의 필드 값만 고려한다.



-----

##### @Data

- @Data는 위에서 설명한 @Getter, @Setter, @RequiredArgsConstructor, @ToString, @EqualsAndHashCode를 한번에 설정해주는 어노테이션이다.

이 어노테이션을 사용하면, 모든 필드를 대상으로 접근자, 설정자가 자동 생성되고, final 또는 @NonNull 필드 값을 파라미터로 받는 생성자가 만들어지고, toString, equals, hashCode 메소드가 자동으로 만들어진다.





**참고**

https://anster.tistory.com/160

http://www.daleseo.com/lombok-popular-annotations/

