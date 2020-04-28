# spring @Qualifier 어노테이션



스프링에서는 @Autowired를 사용해 의존성을 주입한다.

<bean>을 설정할 때 <context:annotation-config/> 를 사용함으로써 굳이 <bean> 태그 안에 <constructor-arg>나 <property>태그를 추가하지 않아도 스프링의 @Autowired 어노테이션이 적용된 생성자, 필드, 메소드에 대해 의존 자동 주입을 처리한다.



하지만, 만약 동일한 타입을 가진 bean 객체가 두개가 있다면 어떻게 될까?

- 스프링이 어떤 빈을 주입해야 할 지 알 수 없어서 스프링 컨테이너를 초기화하는 과정에서 Exception을 발생시킨다.
  - @Autowired의 주입 대상이 한 개여야 하는데 실제로는 두 개 이상의 빈이 존재해 주입할 때 사용할 객체를 선택할 수 없기 때문이다.
  - 단, @Autowired가 적용된 필드나 설정 메서드의 property 이름과 같은 이름을 가진 빈 객체가 존재할 경우에는 이름이 같은 빈 객체를 주입받는다.



이런 문제를 해결하기 위해 **@Qualifier** 어노테이션을 사용한다.



##### @Qualifier 어노테이션

- 사용할 의존 객체를 선택할 수 있도록 해준다.
  1. 설정에서 bean의 한정자 값을 설정한다.
  2. @Autowired 어노테이션이 적용된 주입 대상에 @Qualifier 어노테이션을 설정한다.
     - 이때 @Qualifier의 값으로 앞서 설정한 한정자를 사용한다.



```
<context:annotation-config/>

<bean id="member1" class="example.Member">
	<qualifier value="m1"/>
</bean>

<bean id="member2" class="example.Member"/>

```

위와 같이 <bean> 태그에 <qualifier> 태그를 자식 태그로 추가해 한정자의 값을 설정한다.

한정자 값은 value 속성을 사용해 'm1'이라고 지정했다.



```
pubic class MemberDao{
    private Member member;
    
    @Autowired
    @Qualifier("m1")
    public void setMember(Member member){
        this.member = member;
    }
}
```

그리고 위와같이 @Autowired 어노테이션을 적용한 곳에 @Qualifier 어노테이션을 적용해준다.

주입할 의존 객체에 위의 설정파일에서 설정한 member1 빈 객체를 사용하고 싶으면, @Qualifier 값으로 value값인 "m1"을 지정하면 된다.

여기서 주의할 점은 @Qualifier에 지정한 한정자 값을 갖는 <bean> 객체가 존재하지 않으면 Exception이 발생한다 - 사실상 이 예외처리는 보기 힘들 것 같다. bean객체도 안만들었는데 qualifier라니

아무튼 @Qualifier에 해당하는 빈 객체를 찾지 못하기 때문에, 스프링 컨테이너를 생성하는데 실패한다.



##### 스프링 @Autowired 어노테이션 적용시 의존 객체를 찾는 순서

1. 타입이 같은 bean 객체를 검색한다. => 1개이면 해당 bean 객체를 사용한다.

   - @Qualifier가 명시되어 있는 경우 같은 값을 갖는 bean 객체여야 한다.

2. 타입이 같은 bean 객체가 두개 이상이면, @Qualifier로 지정한 bean 객체를 찾는다.

   - 찾은경우 그 객체를 사용

3. 타입이 같은 bean 객체가 두개 이상이고, @Qualifier가 없는 경우 이름이 같은 빈 객체를 찾는다.

   - 찾은경우 그 객체를 사용



4. 위 경우 모두 해당되지 않으면 컨테이너가 Exception을 발생시킨다.













https://m.blog.naver.com/PostView.nhn?blogId=sksk3479&logNo=221178451242&proxyReferer=https%3A%2F%2Fwww.google.com%2F