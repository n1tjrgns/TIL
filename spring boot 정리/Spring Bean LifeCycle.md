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



##### 참고

 https://sehun-kim.github.io/sehun/springbean-lifecycle/#4 

 https://jeong-pro.tistory.com/167 