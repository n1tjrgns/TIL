# ItemProcessor

##### ItemProcessor는 필수가 아니다.

Writer 부분에서도 충분히 구현 가능하다!



근데 왜 쓰냐??



Reader, Writer와 별도의 단계로 분리되어있기 때문에 **비즈니스 코드를 분리하기 위해** 사용한다.



따라서 배치 어플리케이션에서 비즈니스 로직을 추가할 때는 가장 먼저 Processor를 고려하자



일반적으로 ItemProcessor를 사용하는 방법은 2가지가 있다.

- 변환
  - Reader에서 읽은 데이터를 원하는 타입으로 변환해서 Writer에 넘겨 줄 수 있다.
- 필터
  - Reader에서 넘겨준 데이터를 Writer로 넘겨줄 것인지 결정할 수 있다.
  - null을 반환하면 Writer에 전달되지 않는다.



-----



##### 기본 사용법

ItemProcessor 인터페이스는 두 개의 제네릭 타입이 필요하다.

```java
package org.springframework.batch.item;

public interface ItemProcessor<I, O> {

  O process(I item) throws Exception;

}
```

- I
  - ItemReader에서 받을 데이터 타입
- O
  - ItemWriter에 보낼 데이터 타입



Reader에서 일은 데이터가 ItemProcessor의 process를 통과해서 Writer에 전달된다.

구현해야 할 메소드는 process 하나다.



-----



##### 변환

Reader에서 읽은 타입을 변환해 Writer에 전달해준다.

Teacher라는 도메인 클래스를 읽어와 Name 필드 (String 타입)을 Writer에 넘겨주도록하는 예제를 보자.

```java
@Slf4j
@RequiredArgsConstructor
@Configuration
public class ProcessorConvertJobConfiguration {

    public static final String JOB_NAME = "ProcessorConvertBatch2";
    public static final String BEAN_PREFIX = JOB_NAME + "_";

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    private final EntityManagerFactory emf;
    private final DataSource dataSource;
    //("#{jobParameters[requestDate]}") String requestDate

    @Value("${chunkSize:1000}")
    private int chunkSize;

    @Bean(JOB_NAME)
    public Job job() throws  Exception{
        return jobBuilderFactory.get(JOB_NAME)
                .preventRestart()
                .start(step())
                .build();
    }

    @Bean(BEAN_PREFIX+"step")
    @JobScope
    public Step step() throws Exception{
        return stepBuilderFactory.get(BEAN_PREFIX+"step")
                .<Teacher, String>chunk(chunkSize)
                .reader(reader())
                .processor(processor())
                .writer(writer())
                .build();
    }

    @Bean
    public JdbcPagingItemReader<Teacher> reader() throws Exception{

        Map<String, Object> parameterValues = new HashMap<>();
        parameterValues.put("id", 2);


        return new JdbcPagingItemReaderBuilder<Teacher>()
                .fetchSize(chunkSize)
                .dataSource(dataSource)
                .rowMapper(new BeanPropertyRowMapper<>(Teacher.class))
                .queryProvider(createQueryProvider())
                .name("jdbcPagingItemReader")
             .build();

    }

    @Bean
   // @ConditionalOnMissingBean //프로젝트 상에서 동명의 스프링 빈이 정의되었을 시 쓰지않고
    //정의된 스프링 빈을 사용하고, 없을 시 자동 등록한 빈을 쓰게 유도함.
    public PagingQueryProvider createQueryProvider() throws Exception {
        SqlPagingQueryProviderFactoryBean queryProvider = new SqlPagingQueryProviderFactoryBean();
        queryProvider.setDataSource(dataSource);
        queryProvider.setSelectClause("id, name");
        queryProvider.setFromClause("from teacher");
        //queryProvider.setWhereClause("where id >= :id");

        Map<String, Order> sortKeys = new HashMap<>(1);
        sortKeys.put("id", Order.ASCENDING);

        queryProvider.setSortKeys(sortKeys);

        return queryProvider.getObject();
    }

    @Bean
    public ItemProcessor<Teacher, String> processor(){
        return new ItemProcessor<Teacher, String>() {
            @Override
            public String process(Teacher teacher) throws Exception {
                return teacher.getName();
            }
        };
    }

    private ItemWriter<String> writer(){
        return new ItemWriter<String>() {
            @Override
            public void write(List<? extends String> items) throws Exception {
                for (String item : items) {
                    log.info("Teacher Name={}", item);
                }
            }
        };
    }
}

```

나는 Jpa를 사용하지 않기 때문에 JpaPagingItemReader로 구현되어 있던 부분을

JdbcPagingItemReader를 사용해서 대체해보았다.



ItemProcessor에서 Reader에서 읽어올 타입이 Teacher이고 Writer에 넘겨줄 타입이 String 이기 때문에 제네릭 타입이 <Teacher, String>이 된다.

```java
@Bean
    public ItemProcessor<Teacher, String> processor(){
        return new ItemProcessor<Teacher, String>() {
            @Override
            public String process(Teacher teacher) throws Exception {
                return teacher.getName();
            }
        };
    }
```



chunksize 앞에 선언될 타입 역시 Reader와 Writer 타입을 따라가야해서 아래와 같이 선언된다.

```java
 .<Teacher, String>chunk(chunkSize)
```



코드를 실행해보니 결과 값이 잘 나오는 것을 볼 수 있다.

![processConvert 결과](C:\Users\Lenovo\Desktop\processConvert 결과.PNG)

Teacher 클래스가 Processor를 거치면서 String으로 잘 전환되었다.



-----

##### 필터

**Writer에 값을 넘길지 말지를 Processor에서 판단**

Teacher의 id가 짝수일 경우 필터링하는 예제



```java
@Slf4j
@RequiredArgsConstructor
@Configuration
public class ProcessorNullJobConfiguration {

    public static final String JOB_NAME = "processorNullBatch";
    public static final String BEAN_PREFIX = JOB_NAME + "_";

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    private final DataSource dataSource;


    @Value("${chunkSize:1000}")
    private int chunkSize;

    @Bean(JOB_NAME)
    public Job job() throws  Exception{
        return jobBuilderFactory.get(JOB_NAME)
                .preventRestart()
                .start(step())
                .build();
    }

    @Bean(BEAN_PREFIX+"step")
    @JobScope
    public Step step() throws Exception{
        return stepBuilderFactory.get(BEAN_PREFIX+"step")
                .<Teacher, String>chunk(chunkSize)
                .reader(reader())
                .processor(processor())
                .writer(writer())
                .build();
    }

    @Bean
    public JdbcPagingItemReader<Teacher> reader() throws Exception{

        Map<String, Object> parameterValues = new HashMap<>();
        parameterValues.put("id", 2);


        return new JdbcPagingItemReaderBuilder<Teacher>()
                .fetchSize(chunkSize)
                .dataSource(dataSource)
                .rowMapper(new BeanPropertyRowMapper<>(Teacher.class))
                .queryProvider(createQueryProvider())
                .name("jdbcPagingItemReader")
                .build();

    }

    @Bean
    // @ConditionalOnMissingBean //프로젝트 상에서 동명의 스프링 빈이 정의되었을 시 쓰지않고
    //정의된 스프링 빈을 사용하고, 없을 시 자동 등록한 빈을 쓰게 유도함.
    public PagingQueryProvider createQueryProvider() throws Exception {
        SqlPagingQueryProviderFactoryBean queryProvider = new SqlPagingQueryProviderFactoryBean();
        queryProvider.setDataSource(dataSource);
        queryProvider.setSelectClause("id, name");
        queryProvider.setFromClause("from teacher");
        //queryProvider.setWhereClause("where id >= :id");

        Map<String, Order> sortKeys = new HashMap<>(1);
        sortKeys.put("id", Order.ASCENDING);

        queryProvider.setSortKeys(sortKeys);

        return queryProvider.getObject();
    }

    @Bean
    public ItemProcessor<Teacher, String > processor(){
       return new ItemProcessor<Teacher, String>() {
           @Override
           public String process(Teacher teacher) throws Exception {
               boolean isIgnoreTarget = teacher.getId() % 2 == 0L;
               if (isIgnoreTarget) {
                   log.info(">>>>>>>> Teacher name = {}, isIgnoreTarget={}", teacher.getName(), isIgnoreTarget);
                   return null;
               }
               return String.valueOf(teacher);
           }
       };
    }

    private ItemWriter<String> writer(){
        return new ItemWriter<String>() {
            @Override
            public void write(List<? extends String> items) throws Exception {
                for (String item : items) {
                    log.info("Teacher Name={}", item);
                }
            }
        };
    }
}
```

앞서 말했듯이 ItemProcessor는 Process에 해당하는 부분만 수정을 해주면 된다.

그래서 @Override process() 메소드만 수정을 해주었다.

id 값을 2로 나눴을 때 나누어 떨어지면 null 을 return해 writer로 보내지 않고

log.info만 출력이 되게했다. 

배치를 실행시켰더니

![null 필터링 성공](C:\Users\Lenovo\Desktop\null 필터링 성공.PNG)

이와 같이 id값이 5, 7, 9인 대상에대해서는 Print가 되었고, 그렇지 않은 대상은 log.info가 출력되었다. 



이와 같은 방식으로 Writer에 넘길 데이터를 제어 할 수 있다.







**참고**

https://jojoldu.tistory.com/347















createQueryProvider라는 bean을 생성하는 과정에서 에러 

```
Description:

The bean 'createQueryProvider', defined in class path resource [com/example/batch/seokhun/processor/ProcessorConvertJobConfiguration.class], could not be registered. A bean with that name has already been defined in class path resource [com/example/batch/seokhun/Reader/JdbcPagingItemReaderJobConfiguration.class] and overriding is disabled.

Action:

Consider renaming one of the beans or enabling overriding by setting spring.main.allow-bean-definition-overriding=true


```



**에러** **원인**

다른곳에서 이미 해당 bean을 생성해서 중복이 되는현상

  Bean Overriding을 활성화 하기 위해 application.yml에 spring.main.allow-bean-definition-overriding: true 옵션 추가.

