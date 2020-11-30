## Spring Bean LifeCycle

솔직하게 말해서 기존에는 LifeCycle이 왜 중요한지 체감을 못했다.

하지만, JPA를 공부하다보니 Entity의 LifeCycle이 엄청 중요했다. 그 Entity가영속성 상태인지 아닌지가 중요했다.

그래서 LifeCycle에 대한 공부를 하려한다.



#### Spring Bean의 생명주기

1. ##### Spring Application 시작

2. ##### Bean  설정파일 초기화

   Config 파일이나 web.xml을 사용해 Bean으로 등록할 대상을 찾아 기본 생성자를 호출하여 Bean에 등록한다.

   

3. ##### Bean으로 등록할 객체 초기화

   ```
   //@Component를 사용 할 경우
   @PostConstruct
   public void init() {
   	System.out.println("Init");
   }
   
   //@Bean을 사용 할 경우
   @Bean(initMethod = "init")
   public BeanA beanA() {
   	return new BeanA();
   }
   
   ```

   - Bean의 의존관계를 확인해 (@Autowired, @Resource) 다른 @Bean을 주입해주고, Bean 설정파일에 있는 init 메소드를 호출한다.



4. ##### Bean 준비상태

   모든 Bean의 초기화가 끝나고 사용 가능한 상태



5. ##### Bean 소멸 상태

   ```
   // @Component를 사용할 경우
   @preDestroy
   public void destroy(){
   	System.out.println("destroy");
   }
   
   //@Bean을 사용할 경우
   @Bean(destroyMethod = "destroy")
   public BeanA beanA() {
   	return new BeanA();
   }
   ```

   - 스프링 프로젝트가 종료될 때 Bean 설정파일의 destroy 메소드가 호출된다.



----

#### 의존 관계에 따른 생명주기 변화

##### 의존관계가 없는 경우

- BeanA

```
public class BeanA {
    public BeanA() {
        System.out.println("BeanA : 생성자");
    }

    public void init() {
        System.out.println("BeanA : init");
    }

    public void destroy() {
        System.out.println("BeanA : destroy");
    }
}
```



- BeanB

```
public class BeanB {
    public BeanB() {
        System.out.println("BeanB : 생성자");
    }

    public void init() {
        System.out.println("BeanB : init");
    }

    public void destroy() {
        System.out.println("BeanB : destroy");
    }
}
```



- WebConfig

```
@Configuration // web.xml과 같은 역할
public class WebConfig extends WebMvcConfigurerAdapter {
    @Bean(name = "beanA", initMethod = "init", destroyMethod = "destroy")
    public BeanA getBeanA() {
        return new BeanA();
    }

    @Bean(name = "beanB", initMethod = "init", destroyMethod = "destroy")
    public BeanB getBeanB() {
        return new BeanB();
    }
}
```



- ComponentA

```
@Component
public class ComponentA {
    public ComponentA() {
        System.out.println("ComponentA : 생성자");
    }

    @PostConstruct
    public void init() {
        System.out.println("ComponentA : init");
    }

    @PreDestroy
    public void destroy() {
        System.out.println("ComponentA : destroy");
    }
}
```



- ComponentB

```
@Component
public class ComponentB {
    public ComponentB() {
        System.out.println("ComponentB : 생성자");
    }

    @PostConstruct
    public void init() {
        System.out.println("ComponentB : init");
    }

    @PreDestroy
    public void destroy() {
        System.out.println("ComponentB : destroy");
    }
}
```



- @DiController

```
@RestController
public class DiController {

    public DiController() {
        System.out.println("DiController : 생성자");
    }

    @PostConstruct
    public void init() {
        System.out.println("DiController : init");
    }

    @RequestMapping("/")
    public String index() {
        return "hello world";
    }

    @PreDestroy
    public void destroy() {
        System.out.println("DiController : destroy");
    }
}
```



#### 실행결과

![image-20200507154612613](../AppData/Roaming/Typora/typora-user-images/image-20200507154612613.png)



- 프로젝트 종료시 객체들이 `초기화 된 순서의 역순`으로 destroy() 된다.



----

#### 의존관계가 있는 경우

- Controller에 의존성 주입

```
@RestController
public class DiController {

    private final BeanA beanA;

    private final BeanB beanB;

    private final ComponentA componentA;

    private final ComponentB componentB;

    public DiController(){
        System.out.println("DiController : 생성자");
    }

    @PostConstruct
    public void init() {
        System.out.println("DiController : init");
    }

    @RequestMapping("/")
    public String index() {
        return "hello world";
    }

    @PreDestroy
    public void destroy() {
        System.out.println("DiController : destroy");
    }
}

```

#### 실행결과

![image-20200507183851686](../AppData/Roaming/Typora/typora-user-images/image-20200507183851686.png)





- 스프링 부트에서는 파일 이름 순서대로 Bean을 초기화한다.

  다만, 의존성이 주입되어 의존성이 끼어들게되면 먼저 선언된 의존성부터 생성자가 초기화된다.



----

#### 번외

Bean의 LiceCycle을 고려하지 않고 여러 Bean들을 등록해놓고 @Autowired로 주입받아서 사용하다가는

아직 등록되지 않은 Bean을 사용한다는 에러를 만날 수 있다.



그 이유는

Spring에서는 주로 xml을 사용해 bean을 등록하는데, 자동적으로 위에서 아래로 bean을 스캔해 생성해준다.



> bean 생성 순서가 바뀌는 상황이라면?

- Spring이 알아서 순서를 바꿔 참조되는 Bean을 먼저 생성해주기 때문에 문제가 되지 않는다.



하지만 스프링부트에서 bean을 등록하면 신기하게도, `패키지 디렉토리 순서(알파벳)대로 스캔하`면서 bean을 생성한다.



> 해결 방법은?

#### ` @PostConstruct` 어노테이션 사용 

- Bean이 완전히 주입된 후 작동하기 때문에 NullPointerException이 발생하지 않는다.





##### 참고

 https://sehun-kim.github.io/sehun/springbean-lifecycle/#4 

 https://jeong-pro.tistory.com/167 