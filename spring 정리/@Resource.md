##### @Resource

- 어플리케이션에서 필요로 하는 자원을 자동 연결할 때 사용.
- 스프링 설정파일에 등록되어있는 빈 객체의 name 속성을 통해 자동으로 주입.

- 스프링 설정파일에 태그 필요

  ```java
  <context:annotation-config/> 
  ```



```java
@Resource(name="sessionTemplateForUser")
private  SqlSessionTemplate sqlSession;
```

나의 경우 DB를 여러개 사용할 때, 각각의 name을 지정해 연결을 해줬었다.

