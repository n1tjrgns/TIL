

# 실전에서 사용하는 Spring Batch(Next, Flow)

예제를 보다보면 Job에는 코드가 별로없다.

실제로 Batch 비즈니스 로직을 처리하는 기능은 Step에 구현되어 있다.

아래 예제에서는 log.info가 비즈니스 로직에 해당된다.



- Step에서는 Batch로 실제 처리하고자 하는 기능과 설정을 모두 포함하는 장소다.



Job 내부에서 Step들간에 순서 혹은 처리 흐름을 제어할 필요가 있는데, 제어하기 위한 방법을 알아보자.

-----



##### Next

StepNextJobConfiguration라고 새로운 클래스를 만들고 코드를 아래와 같이 작성했다.



```java
@Slf4j
@Configuration
@RequiredArgsConstructor
public class StepNextJobConfiguration {
    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job stepNextJob(){
        return jobBuilderFactory.get("stepNextJob")
                .start(step1())
                .next(step2())
                .next(step3())
                .build();
    }

    @Bean
    public Step step1() {
        return stepBuilderFactory.get("step1")
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                        log.info(">>>>>>>>>> This is Step1");
                        return RepeatStatus.FINISHED;
                    }
                })
                .build();
    }

    @Bean
    public Step step2() {
        return stepBuilderFactory.get("step2")
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                        log.info(">>>>>>>>>> This is Step1");
                        return RepeatStatus.FINISHED;
                    }
                })
                .build();
    }

    @Bean
    public Step step3() {
        return stepBuilderFactory.get("step3")
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                        log.info(">>>>>>>>>> This is Step1");
                        return RepeatStatus.FINISHED;
                    }
                })
                .build();
    }
}
```

위의 코드를 보면 알 수 있듯이 next()는 순차적으로 Step들을 연결시킬 때 사용된다.



순차적으로 호출이 되는지 실행을 해보자

실행하기전에

Job Parameter를 version=1 로 수정을 했다.

![1552885928196](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1552885928196.png)

하지만 기존에 만들었던 Job 까지 전부 실행이 되어버렸다.



StepNextJob만 실행되도록 설정을 변경해보자.

yml 설정 파일에 아래 코드를 추가한다.



```java
spring:
	batch:
  		job:
    		names: ${job.name:NONE}
```

Spring Batch가 실행될때, Program arguments로 job.name 값이 넘어오면 해당 값과 일치하는 job만 실행시킨다.

job.name:NONE 을 보면, job.name이 있으면 값을 할당하고, 없으면 NONE 을 할당하겠다는 의미이다.

NONE이 할당되면 어떤 배치도 실행하지 않겠다는 의미이다.

값이 없을 때 모든 배치가 실행되지 않도록 막는 역할을 한다.



job.name을 배치 실행시 Program arguments로 넘기도록 IDE의 실행환경도 수정해줘야한다.

![1552886379242](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1552886379242.png)

그 후 실행을 하게 되면

![1552886451483](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1552886451483.png)

지정한 stepNextJob만 수행되었다.

-----



##### 조건별 흐름 제어(Flow)

Next가 순차적으로 Step의 순서를 제어한다는 것을 확인했다.

중요한 점은, **앞의 step에서 오류가 나면 나머지 뒤에 있는 step들은 실행되지 못한다**는 것이다.



그렇다면, 정상일 경우 Step B, 오류발생시 Step C로 수행하고 싶으면 어떻게 할까 ?



이런 경우를 대비해 Spring Batch Job에서는 조건별로 Step을 사용할 수 있다.



코드를 작성해 보자.

```java
Slf4j
@RequiredArgsConstructor
@Configuration
public class StepNextConditionalJobConfiguration {
    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job stepNextConditionalJob() {
        return jobBuilderFactory.get("stepNextConditionalJob")
                .start(conditionalJobStep1())
                .on("FAILED") // FAILED 일 경우
                .to(conditionalJobStep3()) // step3으로 이동한다.
                .on("*") // step3의 결과 관계 없이
                .end() // step3으로 이동하면 Flow가 종료한다.
                .from(conditionalJobStep1()) // step1로부터
                .on("*") // FAILED 외에 모든 경우
                .to(conditionalJobStep2()) // step2로 이동한다.
                .next(conditionalJobStep3()) // step2가 정상 종료되면 step3으로 이동한다.
                .on("*") // step3의 결과 관계 없이
                .end() // step3으로 이동하면 Flow가 종료한다.
                .end() // Job 종료
                .build();
    }

    @Bean
    public Step conditionalJobStep1() {
        return stepBuilderFactory.get("step1")
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                        log.info(">>>>>This is stepNextConditionJob Step1");
                        /*ExitStatus를 FAILED로 지정한다.
                        * 해당 status를 보고 Flow가 진행된다. */
                        contribution.setExitStatus(ExitStatus.FAILED);
                        return RepeatStatus.FINISHED;
                    }
                })
                .build();
    }

    @Bean
    public Step conditionalJobStep2() {
        return stepBuilderFactory.get("conditionalJobStep2")
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                        log.info(">>>>>This is stepNextConditionJob Step2");
                        
                        return RepeatStatus.FINISHED;
                    }
                })
                .build();
    }

    @Bean
    public Step conditionalJobStep3() {
        return stepBuilderFactory.get("conditionalJobStep3")
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                        log.info(">>>>>This is stepNextConditionJob Step3");

                        return RepeatStatus.FINISHED;
                    }
                })
                .build();
    }
}
```

위의 코드를 보면 step1의 성공여부에 따라 결과값이 달라진다.

- step1 실패 : step1 -> step3
- step1 성공 : step1 -> step2 -> step3



- on()

  - 캐치할 ExitStatus 지정
  - '*' 인 경우 모든 ExitStatus가 지정된다.

- to()

  - 다음으로 이동할 Step을 지정한다.

- from()

  - 일종의 **이벤트 리스너** 역할
  - 상태값을 보고 일치하는 상태라면 to() 에 포함된 step을 호출한다.

  - step1의 이벤트 캐치가 FAILED로 되있는 상태에서 추가로 이벤트 캐치하려면 from을 써야만 한다.
  - end는 FlowBuilder를 반환하는 end와 FlowBuilder를 종료하는 end 2개가 있다.
  - on('*')뒤에 있는 end는 FlowBuilder를 반환하는 end
  - build() 앞에 있는 end는 FlowBuilder를 종료하는 end
  - 반환하는 경우의 end 사용시 계속 from을 사용해 이어갈 수 있다.



**on() 이 캐치하는 상태값이 BatchStatus가 아닌 ExitStatus라는 점을 주의해야한다.**

그래서 분기처리를 위해 상태값 조정이 필요하다면 ExitStatus를 조정해야한다.

```java
 contribution.setExitStatus(ExitStatus.FAILED);		
```

이 부분을 조정해줘야 한다.

조정한 값에 따라 해당되는 Job만 실행된 모습을 볼 수 있다.

1. contribution.setExitStatus(ExitStatus.FAILED);		

![1552889385493](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1552889385493.png)

2. //contribution.setExitStatus(ExitStatus.FAILED);		

![1552889452275](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1552889452275.png)



-----

##### Batch Status VS Exit Status

BatchStatus는 Job 또는 Step의 실행 결과를 Spring에 기록할 때 사용하는 Enum이다.

BathStatus로 사용되는 값은 COMPLETED, STARTING, STARTED, STOPPING, STOPPED, FAILED, ABANDONED, UNKNOWN 이 있고, 단어의 뜻와 같다.



Exit Status는 Step의 실행 후 상태를 얘기한다. 얘는 Enum이 아니다.

사용되는 값은 COMPLETED, FAILED, STOPPED, EXECUTING, NOOP, UNKNOWN이 있다.

Spring Batch는 기본적으로 ExitStatus의 ExitCode는 Step의 BatchStatus와 같도록 설정되어 있다. 

하지만 서로가 달라야하는 상황이 필요하다면??

커스텀을 해야한다.

```
.start(step1())
    .on("FAILED")
    .end()
.from(step1())
    .on("COMPLETED WITH SKIPS")
    .to(errorPrint1())
    .end()
.from(step1())
    .on("*")
    .to(step2())
    .end()
    
```

위 stpe1의 실행 결과는 3가지가 될 수 있다.

- step1이 실패하고, Job 또한 실패한다.
- step1이 성공적으로 수행되어 step2가 실행된다.
- step1이 성공적으로 완료되며, COMPLETED WITH SKIPS의 exit코드로 종료된다.



위의 COMPLETED WITH SKIPS는 ExitStatus에는 없는 코드이기 때문에

원하는대로 처리하기 위해 별도의 로직이 필요하다.



```
public class SkipCheckingListener extends StepExecutionListenerSupport {
    public ExitStatus afterStep(StepExecution stepExecution){
        String exitCode = stepExecution.getExitStatus().getExitCode();
        if(!exitCode.equals(ExitStatus.FAILED.getExitCode()) && 					stepExecution.getSkipCount() > 0){
            return new ExitStatus("COMPLETED WITH SKIPS");
        }
        else{
            return null;
        }
    }
}
```

위 코드는 StepExecutionListener에서 먼저 Step이 성공적으로 수행됐는지 확인하고, StepExecution의 skip횟수가 0보다 클 경우 COMPLETED WITH SKIPS의 exitCode를 갖는 ExitStatus를 반환한다.



-----

##### Decide

위에서 진행했던 방식은 2가지 문제점이 있다.

- Step이 담당하는 역할이 2개 이상이 된다.
  - 실제 해당 Step이 처리해야할 로직외에도 분기처리를 시키기 위해 ExitStatus 조작이 필요하다.
- 다양한 분기 로직 처리의 어려움
  - ExitStatus를 커스텀하게 고치기 위해선 Linstener를 생성하고 Job Flow에 등록하는 등 번거로움이 존재한다.



그래서 Spring Batch에서 Step들의 Flow 속 분기만 담당하는 타입이 있다.



**JobExecutionDecider**

DeciderJobConfiguration 클래스를 만들어 보자.

```java
@Slf4j
@RequiredArgsConstructor
@Configuration
public class DeciderJobConfiguration {
    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job deciderJob(){
        return jobBuilderFactory.get("deciderJob")
                .start(startStep())
                .next(decider()) // 홀수 | 짝수 구분
                .from(decider()) //decider의 상태가
                .on("ODD") //ODD 라면
                .to(oddStep()) //oddStep으로 간다.
                .from(decider())
                .on("EVEN") //EVEN이라면
                .to(evenStep()) //evenStep으로 간다.
                .end() //builder 종료
                .build();
    }

    @Bean
    public Step startStep() {
        return stepBuilderFactory.get("startStep")
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                        log.info(">>>>> STARTTTTTT ! ");
                        return RepeatStatus.FINISHED;
                    }
                }).build();
    }

    @Bean
    public Step evenStep() {
        return stepBuilderFactory.get("evenStep")
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                        log.info(">>>>> 짝수 ! ");
                        return RepeatStatus.FINISHED;
                    }
                }).build();
    }

    @Bean
    public Step oddStep() {
        return stepBuilderFactory.get("oddStep")
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
                        log.info(">>>>> 홀수 ! ");
                        return RepeatStatus.FINISHED;
                    }
                }).build();
    }

    @Bean
    public JobExecutionDecider decider() {
        return new OddDecider();
    }


    public static class OddDecider implements JobExecutionDecider {

        @Override
        public FlowExecutionStatus decide(JobExecution jobExecution, StepExecution stepExecution) {
            Random rand = new Random();

            int randomNumber = rand.nextInt(50) + 1;
            log.info("랜덤숫자 : {}", randomNumber);

            if (randomNumber % 2 == 0 ){
                return new FlowExecutionStatus("EVEN");
            } else{
                return new FlowExecutionStatus("ODD");
            }

        }
    }
}
```

위 Batch의 Flow는 다음과 같다

- startStep -> OddDecider에서 홀수 or 짝수를 구분 -> 그에 맞는 oddStep or evenStep진행



- start()
  - Job Flow()의 첫 번째 Step을 시작한다.
- next()
  - startStep 이후에 decider를 실행한다.
- from()
  - from은 이벤트 리스너 역할을한다.
  - decider의 상태 값을 보고 일치하면 to()에 포함된 step을  호출한다.



분기 로직에 대한 모든 일은 OddDecider가 전담하고 있다.

아무리 복잡한 분기로직이 필요하더라도 Step과는 명확히 **역할과 책임이 분리**된채 진행할 수 있다.



Decider 구현체만 따로 살펴보자

```java
 @Bean
    public JobExecutionDecider decider() {
        return new OddDecider();
    }


    public static class OddDecider implements JobExecutionDecider {

        @Override
        public FlowExecutionStatus decide(JobExecution jobExecution, StepExecution stepExecution) {
            Random rand = new Random();

            int randomNumber = rand.nextInt(50) + 1;
            log.info("랜덤숫자 : {}", randomNumber);

            if (randomNumber % 2 == 0 ){
                return new FlowExecutionStatus("EVEN");
            } else{
                return new FlowExecutionStatus("ODD");
            }

        }
    }	
```

JobExecutionDecider 인터페이스를 구현한 OddDecider이다.

랜덤하게 숫자를 생성해 홀수/짝수에 맞는 서로 다른 상태값을 반환한다.



- 주의 : Step으로 처리하는 것이 아니기 때문에 ExitStatus가 아닌 FlowExecutionStatus로 상태를 관리한다.

반환되는 상태를 from(), on()에서 사용하는 것을 알 수 있다.

![1552896285148](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1552896285148.png)

실행 결과 반환되는 상태에 맞는 step이 실행되는 것을 확인 할 수 있었다.



참고**

https://jojoldu.tistory.com/328?category=635883