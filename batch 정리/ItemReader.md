# ItemReader

앞서 정리한 내용에서 각 Step은 Tasklet 으로 처리되며, Tasklet은 ItemReader, Processor, Writer로 구성되어 있음을 알았다.

그렇다면 ItemReader는 어떤 방식으로 동작할까?

그래서 찾아보게 되었다.



![itemreader](C:\Users\Lenovo\Desktop\회사\정리\개념정리\batch 정리\itemreader.PNG)

ItemReader는 말 그대로 데이터를 읽어들인다.

DB말고도 File, Xml, Json 등 다른 데이터 소스도 사용할 수 있다.





이외에도 지원하지 않는 Reader가 필요할 경우 직접 해당 Reader를 만들 수 있다.

- 입력 데이터에서 읽어오기
- 파일에서 읽어오기
- DB에서 읽어오기
- Java Message Service등 다른 소스에서 읽어오기
- 본인만의 커스텀 Reader로 읽어오기



JdbcPagingItemReader를 보면 

ItemReader 외에도 **ItemStream 인터페이스**를 같이 구현 하고있다.



##### ItemStream

ItemStream 인터페이스는 주기적으로 상태를 저장하고 오류가 발생하면 해당 상태에서 복원하기 위한 마커 인터페이스이다.

배치 프로세스의 실행 컨텍스트와 연계해서 **ItemReader의 상태를 저장하고 실패한 곳에서 다시 실행할 수 있게 해주는 역할**을 한다.



ItemStream의 3개 메소드는 다음과 같은 역할을 한다.

- open(), close() 는 스트림을 열고 닫는다.
- update()를 사용하면 Batch 처리의 상태를 업데이트 할 수 있다.









ItemReader 클래스를 보면 read() 메소드만 가지고 있다.


public interface ItemReader<T> {

```java
/**
 * Reads a piece of input data and advance to the next one. Implementations
 * <strong>must</strong> return <code>null</code> at the end of the input
 * data set. In a transactional setting, caller might get the same item
 * twice from successive calls (or otherwise), if the first call was in a
 * transaction that rolled back.
 * 
 * @throws ParseException if there is a problem parsing the current record
 * (but the next one may still be valid)
 * @throws NonTransientResourceException if there is a fatal exception in
 * the underlying resource. After throwing this exception implementations
 * should endeavour to return null from subsequent calls to read.
 * @throws UnexpectedInputException if there is an uncategorised problem
 * with the input data. Assume potentially transient, so subsequent calls to
 * read might succeed.
 * @throws Exception if an there is a non-specific error.
 * @return T the item to be processed
 */
T read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException;
```

}





##### Database Reader

Spring framework의 강점 중 하나는 **개발자가 비즈니스 로직에 집중할 수 있도록 JDBC와 같은 문제점을 추상화** 한 것이다.



그래서 Spring Batch 개발자들은 Spring 프레임워크의 JDBC 기능을 확장했다.



일반적으로 배치 작업은 많은 양의 데이터를 처리해야 한다.

- 보통 실시간 처리가 어려운 대용량 데이터나 대규모 데이터는 배치 어플리케이션 작업을 한다.



Spring의 JdbcTemplate은 분할 처리를 지원하지 않기 때문에(쿼리 결과를 그대로 반환해서) 개발자가 직접 limit, offset을 사용하는 작업이 필요하다.



Spring Batch는 쿼리 결과를 직접 반환하는 Spring의 JdbcTemplate 문제를 해결하기 위해 2개의 타입을 지원한다.



1. Cursor - 실제로 JDBC ResultSet의 기본 기능이다.
   - ResultSet이 open 될 때마다 next() 메소드가 호출 되어 Database의 데이터가 반환된다.
   - 이를 통해 필요에 따라 Database에서 데이터를 Streaming할 수 있다.
2. Paging - 페이지라는 Chunk로 Database에서 데이터를 검색한다.
   - 페이지 단위로 한번에 데이터를 조회해오는 방식이다. 



##### Cursor vs ItemReader

- Cursor 방식은 DB와 커넥션을 맺은 후, Cursor를 한칸씩 옮기면서 지속적으로 데이터를 가져온다.
- Paging 방식은 한번에 10개 or 개발자가 지정한 PageSize 만큼 데이터를 가져온다.

구현체는 다음과 같다.

- Cursor 기반 ItemReader 구현체
  - JdbcCursorItemReader
  - HibernateCursroItemReader
  - StoredProcedureItemReader
- Paging 기반 ItemReader 구현체
  - JdbcPagingItemReader
  - HibernatePagingItemReader
  - JpaPagingItemReader



- CursorItemReader는 Paging과 다르게 Streaming으로 데이터를 처리한다.
- DB와 어플리케이션 사이 통로를 하나 연결해 하나씩 데이터를 가져온다. 





##### CursorItemReader

- Streaming으로 데이터를 처리한다.

- JdbcCursorItemReader는 Cursor 기반의 JDBC Reader 구현체이다.





##### JdbcCursorItemReader 예제코드

JdbcCursorItemReaderJobConfiguration.class

```java
@Slf4j
@RequiredArgsConstructor
@Configuration
public class JdbcCursorItemReaderJobConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    private final DataSource dataSource; // DataSource DI

    private static final int chunkSize = 10;

    @Bean
    public Job jdbcCursorItemReaderJob() {
        return jobBuilderFactory.get("jdbcCursorItemReaderJob")
                .start(jdbcCursorItemReaderStep())
                .build();
    }

    @Bean
    public Step jdbcCursorItemReaderStep() {
        return stepBuilderFactory.get("jdbcCursorItemReaderStep")
                .<Pay, Pay>chunk(chunkSize)
                .reader(jdbcCursorItemReader())
                .writer(jdbcCursorItemWriter())
                .build();
    }

    @Bean
    public JdbcCursorItemReader<Pay> jdbcCursorItemReader() {
        return new JdbcCursorItemReaderBuilder<Pay>()
                .fetchSize(chunkSize)
                .dataSource(dataSource)
                .rowMapper(new BeanPropertyRowMapper<>(Pay.class))
                .sql("SELECT id, amount, tx_name, tx_date_time FROM pay")
                .name("jdbcCursorItemReader")
                .build();
    }
	
    
    //reader를 수행하기 위한 Writer 추가 
    private ItemWriter<Pay> jdbcCursorItemWriter() {
        return new ItemWriter<Pay>() {
            @Override
            public void write(List<? extends Pay> list) throws Exception {
                for (Pay pay : list) {
                    log.info("Current Pay={}", pay);
                }
            }
        };
    }
}
```



Pay.class

```java
@ToString
@Getter
@Setter
@NoArgsConstructor
@Entity
public class Pay {
    private static final DateTimeFormatter FORMATTER = DateTimeFormatter.ofPattern("yyyy-MM-dd hh:mm:ss");

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private Long amount;
    private String txName;
    private LocalDateTime txDateTime;

    public Pay(Long amount, String txName, String txDateTime) {
        this.amount = amount;
        this.txName = txName;
        this.txDateTime = LocalDateTime.parse(txDateTime, FORMATTER);
    }

    public Pay(Long id, Long amount, String txName, String txDateTime) {
        this.id = id;
        this.amount = amount;
        this.txName = txName;
        this.txDateTime = LocalDateTime.parse(txDateTime, FORMATTER);
    }
}



```



 각 코드들의 역할을 확인해보자.

**chunk**

- <Pay, Pay> 에서 첫번째 Pay는 Reader에서 반환할 타입이고, 두번째 Pay는 Writer에 파라미터로 넘어올 타입이다.
- chunkSize로 인자값을 넣은 경우는 Reader & Writer가 묶일 Chunk 트랜잭션 범위이다.



**fetchSize**

- DB에서 한번에 가져올 데이터의 양을 나타낸다.
- Paging과 다른 점은, Paging은 실제 쿼리를 limit, offset을 이용해 분할 처리하는 반면, Cursor는 분할 처리 없이 실행되나 내부적으로 가져오는 데이터는 FetchSize만큼 가져와 read()를 통해 하나씩 가져온다.



**dataSource**

- Database에 접근하기 위해 사용할 Datasource 객체를 할당



**rowMapper**

- 쿼리 결과를 Java 인스턴스로 매핑하기 위한 Mapepr
- 보편적으로 BeanPropertyRowMapper.Class를 많이 사용



**sql**

- Reader로 사용할 쿼리문을 사용하면 된다.



**name**

- reader의 이름을 지정.
- Bean이 아니고 Spring Batch의 ExecutionContext에 저장될 이름.









Reader에서 읽은 데이터에 대해 크게 변경된 로직이 없다면 writer만 구현하면 된다.

**processor는 필수가 아니다.**



ItemReader의 가장 큰 장점은 **데이터를 Streaming 할 수 있다**는 것이다.

read() 메소드가 데이터를 하나씩 가져와 ItemWriter로 데이터를 전달하고, 다음 데이터를 다시 가져온다. 

이를 통해 reader & processor & writer가 Chunk 단위로 수행되고 주기적으로 Commit된다.

- 고성능 배치 처리에서 핵심이 된다.



테스트를 진행하기 위해 local Mysql에 테이블을 만든다.

```mysql
create table pay (
  id         bigint not null auto_increment,
  amount     bigint,
  tx_name     varchar(255),
  tx_date_time datetime,
  primary key (id)
) engine = InnoDB;

insert into pay (amount, tx_name, tx_date_time) VALUES (1000, 'trade1', '2018-09-10 00:00:00');
insert into pay (amount, tx_name, tx_date_time) VALUES (2000, 'trade2', '2018-09-10 00:00:00');
insert into pay (amount, tx_name, tx_date_time) VALUES (3000, 'trade3', '2018-09-10 00:00:00');
insert into pay (amount, tx_name, tx_date_time) VALUES (4000, 'trade4', '2018-09-10 00:00:00');
```



실행하기 전에 Reader의 작동을 확인하기 위해 Log Level을 변경해보자.

application.yml에 코드를 추가한다.

```java
logging:
  level: debug
```

그리고 배치를 실행한다

![Pay test](C:\Users\Lenovo\Desktop\Pay test.PNG)

그랬더니 등록한 데이터가 잘 조회되어 Writer에 명시한대로 데이터를 Print했다.



##### CursorItemReader의 주의사항

- Cursor는 하나의 Connection으로 Batch가 끝날때까지 사용되기 때문에 Batch가 끝나기전에 DB와 어플리케이션의 Conenction이 먼저 끊어질 수 있다.
- 이런 이유로 DB와 SocketTimeout을 충분히 큰 값으로 설정해야 한다.

그래서 Batch 수행 시간이 오래 걸리는 경우에는 PagingItemReader를 사용하는게 좋다고한다.

- 한 페이지를 읽을때마다 Connection을 맺고 끊기 때문에 아물 많은 데이터라도 타임아웃과 부하 없이 수행 될 수 있다.



-----



##### PagingItemReader

- 페이징은 각 쿼리에 시작 행 번호(offset)와 반환 할 행 수(limit)를 지정해야 한다.
- Batch에서는 PazeSize에 맞게 offset과 limit을 지정해준다.

- 각 페이지마다 새로운 쿼리를 실행하기 때문에 페이징시 결과를 정렬하는 것이 중요하다.
- 데이터 결과의 순서가 보장될 수 있도록 order by가 권장된다.



**JdbcPagingItemReader**

예제를 만들어 보자.

마찬가지로 JdbcTemplate 인터페이스를 이용한 PagingItemReader이다.

```java
@Slf4j
@RequiredArgsConstructor
@Configuration
public class JdbcPagingItemReaderJobConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    private final DataSource dataSource; // DataSource DI

    private static final int chunkSize = 10;

    @Bean
    public Job jdbcPagingItemReaderJob() throws Exception {
        return jobBuilderFactory.get("jdbcPagingItemReaderJob")
                .start(jdbcPagingItemReaderStep())
                .build();
    }

    @Bean
    public Step jdbcPagingItemReaderStep() throws Exception {
        return stepBuilderFactory.get("jdbcPagingItemReaderStep")
                .<Pay, Pay>chunk(chunkSize)
                .reader(jdbcPagingItemReader())
                .writer(jdbcPagingItemWriter())
                .build();
    }

    @Bean
    public JdbcPagingItemReader<Pay> jdbcPagingItemReader() throws Exception {
        Map<String, Object> parameterValues = new HashMap<>();
        parameterValues.put("amount", 2000);

        return new JdbcPagingItemReaderBuilder<Pay>()
                .fetchSize(chunkSize)
                .dataSource(dataSource)
                .rowMapper(new BeanPropertyRowMapper<>(Pay.class))
                .queryProvider(createQueryProvider())
                .parameterValues(parameterValues)
                .name("jdbcPagingItemReader")
                .build();
    }

    private ItemWriter<Pay> jdbcPagingItemWriter() {
        return list -> {
            for (Pay pay: list) {
                log.info("Current Pay={}", pay);
            }
        };
    }

    @Bean
    public PagingQueryProvider createQueryProvider() throws Exception {
        SqlPagingQueryProviderFactoryBean queryProvider = new SqlPagingQueryProviderFactoryBean();
        queryProvider.setDataSource(dataSource); // Database에 맞는 PagingQueryProvider를 선택하기 위해 
        queryProvider.setSelectClause("id, amount, tx_name, tx_date_time");
        queryProvider.setFromClause("from pay");
        queryProvider.setWhereClause("where amount >= :amount");

        Map<String, Order> sortKeys = new HashMap<>(1);
        sortKeys.put("id", Order.ASCENDING);

        queryProvider.setSortKeys(sortKeys);

        return queryProvider.getObject();
    }
}
```

**parameterValues**

- 쿼리에 대한 매개 변수 값의 Map을 지정한다.
- `queryProvider.setWhereClause` 을 보면 어떻게 변수를 사용하는지 자세히 알 수 있다.
- where 절에서 선언된 파라미터 변수명과 parameterValues에서 선언된 파라미터 변수명이 일치해야만 합니다.



예제를 보면 JdbcCursorItemReader와 설정이 다른것이 있는데,

바로 **createQueryProvicer**이다.



JdbcCursorItemReader에서는 String 타입으로 쿼리를 생성했지만, PagingItemReader에서는 **PagingQueryProvider**를 통해 쿼리를 생성한다.



왜 ???

- 각 DB별 Paging을 지원하는 자체적인 전략이 있기 때문에

- providers.put(MYSQL, new MySqlPagingQueryProvider());
- providers.put(ORACLE, new OraclePagingQueryProvider());



하지만 이는 DB마다 Provider 코드를 바꿔야하는 불편함이 있어서 Spring Batch에서는 **SqlPagingQueryProviderFactoryBean을 통해 Datasource 설정값을 보고 작성된 Provider중 하나를 자동으로 선택**하도록 한다.



소스를 실행해보자.

amount >= amount 인 항목에 대해서만 Print문이 찍혔다.

![1552984384792](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1552984384792.png)

나는 여기서 Debug가 찍히지 않았다.

구글링을 해서 이것저것 해보았으나 잘 되지 않았다. 오히려 에러만 늘어났다.

정상적으로 디버그가 되었다면 쿼리 로그에 LIMIT 10이 들어가 있는 것을 볼 수 있다고 한다.

JdbcPagingItemReader에서 선언된 fetchSize에 맞게 자동으로 쿼리에 추가해줬기 때문이다.

만약 조회할 데이터가 10개 이상이였다면 offset으로 적절하게 다음 fetchSize만큼 가져올 수 있다.



##### PagingItemReader 주의사항

정렬(Order)가 무조건 포함되어 있어야 한다.



##### ItemReader 주의사항

JpaRepository, ListItemReader, QueueItemReader에 사용하면 안된다.





- ItemReader가 어디에서 어떤 방식으로 데이터를 읽느냐에 따라 Batch의 성능을 좌우한다.





**참고**

 https://jojoldu.tistory.com/336