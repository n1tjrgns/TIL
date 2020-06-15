## 스프링 AOP with (프록시 패턴)

> 인프런 백기선님의 예제로 배우는 스프링 입문 (개정판) 강좌를 참고해 정리한 글 입니다.



내가 알기로 AOP는 공통 로직을 줄이기 위해 사용한다고 알고있다.

> 어떻게?? 찾아보자. 





#### 대표적인 스프링 AOP 예제

- `@Transactional` : AOP 기반으로 만들어짐

  원래 쿼리를 수행하기전에, setAutoCommit = false 

  쿼리 수행 후 commit or rollback 코드를 넣어준다.

- 비즈니스 로직에 집중할 수 있게 해주기 위해서



#### 다양한 AOP 구현 방법

- 컴파일 A.java ---(AOP)--> A.class (AspectJ)

  컴파일 시점에 특정 코드를 삽입해줌

- 바이트코드 조작 A.java -> A.class ---(AOP)--> 메모리

  class loader가 클래스를 읽어와서 메모리에 올리는 시점에 (AOP)삽입
  
- 프록시 패턴(스프링 AOP)



#### 프록시 패턴

- 간단하게 말해서 기존 코드를 건드리지 않고 새 기능을 추가하는 것이다.



>  예제를 통해 배워보자



1. 결제 방식을 위한 Payment 인터페이스

```
public interface Payment {

    void pay (int amount);
}

```

   

2. Payment를 사용 할 클라이언트 Store 코드

```
public class Store {

    Payment payment;

    //생성자를 통한 의존성 주입
    //store 클래스는 payment 없이는 동작 할 수 없다. 강제성성
   public Store(Payment payment) {
        
       this.payment = payment;
    }

    public void buySomething(int amount){
        
       payment.pay(100);
    }

}
```

여기서 Store 생성자를 Payment로 지정함으로써, Store를 만들 때 Payment가 반드시 필요하도록 구현한다.



3. Cash 클래스

```
public class Cash implements Payment{

    @Override
    public void pay(int amount) {
        System.out.println(amount + "현금결제");
    }
}
```



4. StopWatch 기능을 추가한 CashPerf 클래스

```
public class CashPerf implements Payment{

    Payment cash = new Cash();

    @Override
    public void pay(int amount) {
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();

        cash.pay(amount);

        stopWatch.stop();
        System.out.println(stopWatch.prettyPrint());
    }
}
```



5. StoreTest 클래스

```
class StoreTest {

    @Test
    public void testPay(){
        //CashPerf를 추가함으로써 기존 코드를 건드리지 않고도 추가 테스트가 가능
        Payment cashPerf = new CashPerf();
        Store store = new Store(cashPerf);
        store.buySomething(100);
    }
}
```



`Payment cashPerf = new CashPerf();`

- new Cash 대신에 StopWatch 기능을 추가한 CashPerf 클래스로 선언한다.

- 생성된 `cashPerf` 객체를 `Store`에 넘겨준다.



> Store 코드를 건드리지 않고 StopWatch 기능을 추가 할 수 있게 되었다.



----

#### @AOP 적용 예제



> 시간 측정을 위한 어노테이션을 생성해보자.



#### @LogExecutionTime

```
@Target(ElementType.METHOD) //어노테이션 사용 위치 선언
@Retention(RetentionPolicy.RUNTIME) //어노테이션 정보를 어느시점까지 유지할지
public @interface LogExecutionTime {

}

```



#### LogAspect

```
@Component //빈으로 등록되어야 하기 때문에
@Aspect
public class LogAspect {

    //로거 생성
    Logger logger = LoggerFactory.getLogger(LogAspect.class);

    //Around 어노테이션으로 joinPoint를 받는데
    //joinpoint는 해당 어노테이션이 붙은 메소드를 가리킴킴
    @Around("@annotation(LogExecutionTime)")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable{
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();
		
		//해당 메소드를 실행
        Object proceed = joinPoint.proceed();

        stopWatch.stop();
        logger.info(stopWatch.prettyPrint());
		
		//리턴
        return proceed;
    }

}
```

- Around라는 어노테이션을 사용하면 joinPoint라는 파라미터를 받을 수 있다. (Around 안에 속성으로 달린 어노테이션에 대해서)

- 스프링 내부 메커니즘에 의해서 위의 프록시 예제에서 했던 일들을 어노테이션 몇개로 간단하게 사용 할 수 있다.