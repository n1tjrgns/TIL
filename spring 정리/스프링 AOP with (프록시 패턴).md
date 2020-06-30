## 스프링 AOP with (프록시 패턴)

> 인프런 백기선님의 예제로 배우는 스프링 입문 (개정판) 강좌를 참고해 정리한 글 입니다.



내가 알기로 AOP는 공통 로직을 줄이기 위해 사용한다고 알고있다.

> 어떻게?? 찾아보자. 



#### AOP란?

- 어플리케이션 전체에 걸쳐 사용되는 기능을 재사용하도록 지원

  핵심적인 기능에서 부가적인 기능을 분리하여 `Aspect`라는 모듈로 만들어 설계하고 개발하는 방법이다.

  각각의 클래스를 부가기능의 관점에서 바라보았을 때 공통된 요소를 추출하는것(로깅, 트랜잭션, 보안)

  가로 영역의 공통된 부분을 잘라냈다고 하여, AOP를 `크로스 컷팅(Cross-Cutting)`이라고 부르기도 한다.



#### 장점

- 어플리케이션 전체에 흩어진 공통 기능이 하나의 장소에서 관리된다.
- 다른 서비스 모듈이 본인의 목적에만 충실하고 그외 사항은 신경쓰지 않아도 된다.



#### 용어

`타겟`

부가기능을 부여할 대상



`애스펙트 (Aspect)`

객체지향 모듈을 오브젝트라 부르는것과 비슷하게, 부가기능 모듈은 애스펙트라고 부른다.

`핵심기능에 부가되어 의미를 갖는` 특별한 모듈이다.

부가될 기능을 정의한 `어드바이스`와 어드바이스를 어디에 적용할지를 결정하는 `포인트컷`을 함께 가진다.



`어드바이스 (Advice)`

실질적으로 부가기능을 담은 구현체

Aspect가 '무엇'을 '언제' 할지에 대한 정의를 담고있다.

- @Before : 어드바이스 타겟 메소드가 호출되기 전에 어드바이스 기능을 수행
- @After : 타겟 메소드의 결과에 관계없이(성공, 예외 상관없이) 타겟 메소드가 완료되면 어드바이스 기능 수행
- @AfterReturning (정상적 반환 이후) : 타겟 메소드가 성공적으로 결과값을 반환 후에 어드바이스 기능을 수행
- @AfterThrowing (예외 발생 이후) : 타겟 메소드가 수행 중 예외를 던지게 되면 어드바이스 기능을 수행
- @Around (메소드 실행 전 후) : 어드바이스가 타겟 메소드를 감싸서 타겟 메소드 호출 전, 후 어드바이스 기능을 수행 -> 스프링 배치의 beforeJob, afterJob과 비슷한 것 같다.



`포인트컷 (PointCut)`

부가기능이 적용될 대상(메소드)를 선정하는 방법

어드바이스를 적용할 조인포인트를 선별하는 기능을 정의한 모듈이다.



`조인포인트 (JoinPoint)`

어드바이스가 적용될 수 있는 위치를 얘기한다.

Spring에서는 `메소드 조인포인트만 제공`한다.



`프록시 Proxy`

타겟을 감싸서 타겟의 요청을 대신 받아주는 랩핑(Wrapping) 오브젝트

클라이언트에서 타겟을 호출하면, 타겟을 감싸고 있는 프록시가 호출되어, 타겟 메소드 실행전에 선처리, 타겟 메소드 실행 후 후처리를 실행시키도록 구성되어 있다.

![image-20200630190256879](../AppData/Roaming/Typora/typora-user-images/image-20200630190256879.png)

- 프록시는 호출을 가로챈 후, 어드바이스에 등록된 기능을 수행 후 타겟 메소드를 호출한다.



`인트로덕션 (Introduction)`

타겟 클래스에 코드 변경없이 신규 메소드나 멤버변수를 추가하는 기능



`위빙 (Weaving)`

지정된 객체에 Aspect를 적용해 새로운 프록시 객체를 생성하는 과정

A 객체에 트랜잭션 Aspect가 지정되어 있다면, A라는 객체가 실행되기전 커넥션을 오픈하고 실행이 끝나면 커넥션을 종료하는 기능이 추가된 프록시 객체가 생성되고, 이 프록시 객체가 앞으로 A 객체가 호출되는 시점에 사용된다.

이때의 프록시객체가 생성되는 과정을 `위빙`이라고한다.

Spring AOP는 `런타임 시점`에서 프록시 객체가 생성된다.

 

#### 대표적인 스프링 AOP 예제

- `@Transactional` : AOP 기반으로 만들어졌다.

  원래 쿼리를 수행하기전에, setAutoCommit = false 

  쿼리 수행 후 commit or rollback 코드를 넣어준다.

- 비즈니스 로직에 집중할 수 있게 해주기 위해서



#### 다양한 AOP 구현 방법

- 컴파일 A.java ---(AOP)--> A.class (AspectJ)

  컴파일 시점에 특정 코드를 삽입해준다.

- 바이트코드 조작 A.java -> A.class ---(AOP)--> 메모리

  class loader가 클래스를 읽어와서 메모리에 올리는 시점에 (AOP)삽입한다.
  
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



#### 참고

 https://jojoldu.tistory.com/71 