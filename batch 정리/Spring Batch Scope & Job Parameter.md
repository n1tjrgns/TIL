# Spring Batch Scope & Job Parameter

##### JobParameter와 Scope

Spring Batch는 외부 혹은 내부에서 파라미터를 받아 여러 Batch 컴포넌트에서 사용할 수 있게 지원하고 있다. 이 파라미터를 **Job Parameter**라고 한다.

Jop Parameter를 사용하기 위해선 항상 Spring Batch 전용 Scope를 선언해야 한다.

크게 @StepScope와 @JobScope 2가지가 있다.

사용법은 아래와 같이 SpEL로 선언해서 사용하면 된다.

```java
@Value("#{jobParameters[파라미터명]}")
```

[SimpleJob 생성해보기](https://n1tjrgns.tistory.com/167)에 가면 @Value를 사용한 예제가 있다.



@JobScope는 Step 선언문에서 사용 가능하고, @StepScope는 Tasklet이나 ItemReader,Processor,Writer에서 사용할 수 있다.



Job Parameter의 타입으로 사용할 수 있는 것은 Double, Long, Date, String이 있다.

LocalDate와 LocalDateTime은 String으로 받아 타입변환을 해서 사용해야한다.



위 링크의 예제에서 보면

```java
  return jobBuilderFactory.get("simpleJob")
                .start(simpleStep1(null))
```

호출하는 쪽에서 null을 할당했다.

- Job Parameter의 할당이 어플리케이션 실행시 하지 않기 때문에 가능하다.

무슨 소리인지 잘 모르겠다.



##### @StepScope & @JobScope 

Spring Batch는 @StepScope와 @JobScope라는 특별한 Bean Scope를 지원한다.

**Spring Bean의 기본 Scope는 singleton**이다.

하지만 아래처럼 SpringBatch 컴포넌트(Tasklet, ItemReader, ItemWriter, ItemProcessor)에 @StepScope를 사용하게 되면

```java
@Bean
@StepScope
public ListItemReader<Integer> simpleWriterReader() {
    List<Integer> items = new ArrayList<>();
    
    for(int i=0; i<100; i++){
        items.add(i);
    }
    
    return new ListItemReader<>(items);
}
```

Spring Batch가 Spring 컨테이너를 통해 지정된 Step의 실행시점에 해당 컴포넌트를 Spring Bean으로 생성한다. 

마찬가지로 @JobScope는 Job 실행시점에 Bean이 생성된다.

즉, **Bean의 생성 시점을 지정된 Scope가 실행되는 시점으로 지연시킨다.**



이렇게 Bean의 생성시점을 늦추는 이유는 이유는 뭘까 ??



Bean의 생성시점을 어플리케이션 실행 시점이 아닌, Step이나 Job의 실행시점으로 지연시키면서 얻는 장점은 크게 2가지가 있다.



1. JobParameter의 LateBinding이 가능하다. 

Job Parameter를 StepContext 또는 JobExecutionContext 레벨에서 할당시킬 수 있다.

​	(이게무슨 소리일까)

꼭 어플리케이션이 실행되는 시점이 아니라도 Controller나 Service 같은 비즈니스 로직 처리 단계에서 Job Parameter를 할당시킬 수 있다. 



2. 동일한 컴포넌트를 병렬 혹은 동시에 사용할 때 유용하다.

Step안에 Tasklet이 있고, 이 Tasklet은 멤버 변수와 이 멤버 변수를 변경하는 로직이 있다고 가정할 때, @StepScope 없이 Step을 병렬로 실행시키면 서로 다른 Step에서 하나의 Tasklet을 두고 계속 상태를 변경하려 할 것이다. 

하지만 @StepScope를 사용함으로써 각각의 Step에서 별도의 Tasklet을 생성하고 관리하기 때문에 서로의 상태를 침범할 일이 없다.



##### Job Parameter

Job Parameter는 @Value를 통해 사용가능하다.

또한 Job Parameter는 Step이나 Tasklet, Reader 등 Batch 컴포넌트 Bean의 생성 시점에 호출할 수 있지만, 정확히는 Scope Bean을 생성할 때 가능하다.

즉, @StepScope, @JobScope Bean을 생성할때만 Job Parameter가 생성되기 때문에 사용할 수 있다.

아래 예제를 보자.

메소드를 통해 Bean을 생성하지 않고, 클래스에서 직접 Bean을 생성해보겠다.

Job과 Step의 코드에서 @Bean과 @Value('#{jobParameters[파라미터명]}') 을 제거하고 SimpleJobTasklet을 생성자 DI 로 받도록 하겠다.

```java
@Bean
    public Job simpleParameter(){
        log.info(">>>>> definition simpleparameter");
        return jobBuilderFactory.get("simpleParameter")
                .start(simpleStep1())
                .next(simpleStep2())
                .build();
    }

    private final SimpleJobTasklet tasklet1;

    //@Bean
    //@JobScope
    public Step simpleStep1() {
        log.info(">>>>>> simpleStep11111111");
        return stepBuilderFactory.get("simpleStep1")
                .tasklet(tasklet1)
                .build();
    }
```



SimpleJobTasklet은 아래와 같이 @Component와 @StepScope로 Scope가 Step인 Bean으로 생성한다. 

그리고 이 상태에서  @Value("#{jobParameters[requestDate]}")를 Tasklet의 멤버변수로 할당한다.

```java
@Slf4j
@Component   // StepScope Bean
@StepScope   // StepScope Bean
public class SimpleJobTasklet implements Tasklet {

    @Value("#{jobParameters[requestDate]}")
    private String requestDate;

    @Override
    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
        log.info(">>>>>> this is step1");
        log.info(">>>>> requestDate = {}", requestDate);
        return RepeatStatus.FINISHED;
    }
}
```

위의 코드는 메소드의 파라미터로 JobParameter를 할당받지 않고, 클래스의 멤버 변수로 JobParameter를 할당 받도록 한다.

![1552956356907](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1552956356907.png)

실행 결과 정상적으로 JobParameter를 받아왔다.

SimpleJobTasklet Bean이 @StepScope로 생성되었기 때문이다.



반면에, 이 SimpleJob Bean을 일반 singleton Bean으로 생성할 경우

```
'jobParameters' cannot be found on object of type 'org.springframework.beans.factory.config.BeanExpressionContext' - maybe not public or not valid?
```

 에러를 볼 수 있다.



Bean을 메소드나 클래스 어느 것을 통해서 생성해도 무방하나 Bean의 Scope는 Step이나 Job이어야 한다!! 반드시!!

- JobParameters를 사용하기 위해 꼭 @StepScope, @JobScope 로 Bean을 생성하자



##### JobParameter  vs 시스템 변수

job parameter를 써야하는 이유에 대해 알아보겠다.

- 무조건 Job Parameter를 써야하는가?
- Spring Boot에서 사용하던 환경변수나 시스템 변수를 사용하면 안되나?
- CommandLineRunner를 사용하면 java jar application.jar -D로 시스템 변수를 지정할 수 있지않나?



아래 코드를 보자.



##### 웹 서버에서 Batch 수행 예제

- 외부에서 넘겨주는 파라미터에 따라 Batch를 다르게 작동할 때

```
@Slf4j
@RequiredArgsConstructor
@RestController
public class JobLauncherController{
    private final JobLauncher jobLauncher;
    privatee final Job job;
    
    @GetMapping("/launchjob")
    public String hadle(@RequestParam("fileName") String fileName) throws 																Exception{
        try{
            JobPameters jobParameters = new JobParametersBuilder()
            	.addString("input.file.name",fileName)
            	.addLong("time",System.currentTimeMillis())
            	.toJobParameters();
            jobLauncher.run(job, jobParameters);
           
        } catch (Exception e){
            log.inf(e.getMessage());
        }
        
        return "Done";
    } 
}
```

예제를 보면 Controller에서 Request Parameter로 받은 값을 Job Parameter로 생성한다.

Job Parameter를 각각의 Batch 컴포넌트들이 사용되면서 변경이 심한 경우에도 쉽게 대응 할 수 있다.

- 웹 서버에서 Batch를 관리하는 것은 권장하지 않는다.
- 실제 운영 환경에서는 어떻게 관리해야 하는지 좀 더 알아보자

-----

##### SpEL

Spring 3.0부터 나온 스프링 전용 표현식 언어이다.







**참고**

http://www.egovframe.go.kr/wiki/doku.php?id=egovframework:rte2:ptl:spel

https://jojoldu.tistory.com/330?category=635883