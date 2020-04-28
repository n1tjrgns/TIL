## @Bean vs @Component 

`@Component`와 `@Bean` 둘 다 Spring Container에 Bean을 등록하도록 하는 어노테이션으로 알고있다. 그렇다면 둘의 차이점은 뭘까? 궁금해졌다.



#### @Bean

- 개발자가 컨트롤이 불가능한 `외부 라이브러리들을 Bean으로 등록하고 싶은 경우` 사용된다.

  (ObjectMapper의 경우 ObjectMapper Class에 @Component를 선언할 수 없으니, ObjectMapper의 인스턴스를 생성하는 메소드를 만들고 해당 메소드에 @Bean을 선언해 등록한다.)



> ObjectMapper란?
>
> 기본 POJO 또는 범용 JSON Tree Model에서 JSON을 읽고 쓰는 기능과 변환 수행을 위한 기능을 제공한다.
>
> 또한 다양한 스타일의 JSON 컨텐츠와 함께 작동하고 다형성 및 객체 동일성과 같은 고급 객체 개념을 지원하도록 Customizing 할 수 있다.

Java Object와 JSON 파싱을 쉽게 해주는 클래스이다.



#### @Component

- 직접 컨트롤 가능한 class를 Bean으로 등록하기 위한 어노테이션이다.



> 개발자가 생성한 Class에 @Bean선언이 가능할까?
>
> - No





@Bean과 @Componet는 각자 선언할 수 있는 타입이 정해져있어, 해당 용도 이외에는 컴파일 에러를 발생시킨다.

**Bean**

```
@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Bean {

	...
}
```

Target이 Method로 지정되어있어 메소드에만 선언 가능하다.



**Component**

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Indexed
public @interface Component {
	...
}	
```

Target이 TYPE로 지정되어있어 Class위에서 선언 가능하다.



#### 추가

- @Configuration 내부의 @Bean 메소드는 `싱글톤`을 보장받지만 @Component 내부 메소드는 그렇지 않다.

