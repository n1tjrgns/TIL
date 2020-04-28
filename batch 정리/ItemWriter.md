# ItemWriter

Reader까지 알아봤으니, Writer를 알아볼 차례이다.



앞에서 설명했듯이 ItemWriter는 item 하나를 작성하지 않고 **Chunk 단위로 묶인 item List**를 다룬다.

```java
import java.util.List;
public interface ItemWriter<T> {

/*
Process the supplied data element. Will not be called with any null items
in normal operation.
@param items items to be written
@throws Exception if there are errors. The framework will catch the
exception and convert or rethrow it as appropriate.
*/
void write(List<? extends T> items) throws Exception;

}
```

ItemReader의 read()는 Item 하나를 반환하는 반면, Writer의 write()는 인자로 ItemList를 받는다.



##### Database Writer

-----

Writer는 Chunk 단위의 마지막 단계이다.

우선 DB영속성과 관련해 마지막에는 항상 Flush()를 해줘야 한다.

- 영속성 : Entity를 영구적으로 저장해주는 환경



Writer가 받은 모든 Item이 처리 된 후, Spring Batch는 **현재 트랜잭션을 커밋**한다.

DB관련 writer는 3가지가 있다.

- JdbcBatchItemWriter
- HibernateItemWriter
- JpaItemWriter



##### JdbcBatchItemWriter

-----

ORM을 사용하지 않는 경우 writer는 대부분 JdbcBatchItemWriter를 사용한다.

JDBC의 Batch 기능을 사용해 한번에 DB로 전달하여 DB내부에서 쿼리들이 실행되도록 한다.

![jdbcitemwriter](C:\Users\Lenovo\Desktop\jdbcitemwriter.PNG)



이로써 어플리케이션과 데이터베이스 간에 데이터를 주고 받는 회수를 최소화해 성능 향상을 꾀할 수 있다.

- 업데이트를 일괄 처리로 그룹화하면 데이터베이스와 어플리케이션간 왕복 횟수가 줄어들어 성능이 향상된다.



JdbcBatchItemWriter 클래스의 write() 메소드 

```java
@SuppressWarnings({"unchecked", "rawtypes"})
	@Override
	public void write(final List<? extends T> items) throws Exception {

		if (!items.isEmpty()) {

			if (logger.isDebugEnabled()) {
				logger.debug("Executing batch with " + items.size() + " items.");
			}

			int[] updateCounts;

			if (usingNamedParameters) {
				if(items.get(0) instanceof Map && this.itemSqlParameterSourceProvider == null) {
					updateCounts = namedParameterJdbcTemplate.batchUpdate(sql, items.toArray(new Map[items.size()]));
				} else {
					SqlParameterSource[] batchArgs = new SqlParameterSource[items.size()];
					int i = 0;
					for (T item : items) {
						batchArgs[i++] = itemSqlParameterSourceProvider.createSqlParameterSource(item);
					}
					updateCounts = namedParameterJdbcTemplate.batchUpdate(sql, batchArgs);
				}
			}
			else {
				updateCounts = namedParameterJdbcTemplate.getJdbcOperations().execute(sql, new PreparedStatementCallback<int[]>() {
					@Override
					public int[] doInPreparedStatement(PreparedStatement ps) throws SQLException, DataAccessException {
						for (T item : items) {
							itemPreparedStatementSetter.setValues(item, ps);
							ps.addBatch();
						}
						return ps.executeBatch();
					}
				});
			}

			if (assertUpdates) {
				for (int i = 0; i < updateCounts.length; i++) {
					int value = updateCounts[i];
					if (value == 0) {
						throw new EmptyResultDataAccessException("Item " + i + " of " + updateCounts.length
								+ " did not update any rows: [" + items.get(i) + "]", 1);
					}
				}
			}
		}
	}
```



##### JdbcBatchItemWriter 예제

```java
@Slf4j
@RequiredArgsConstructor
@Configuration
public class JdbcBatchItemWriterJobConfiguration {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    private final DataSource dataSource;

    private static final int chunkSize = 10;

    @Bean
    public Job jdbcBatchItemWriterJob(){
        return jobBuilderFactory.get("jdbcBatchItemWriterJob")
                .start(jdbcBatchItemWriterStep())
                .build();
    }

    @Bean
    public Step jdbcBatchItemWriterStep() {
        return stepBuilderFactory.get("jdbcBatchItemWriterStep")
                .<Pay, Pay>chunk(chunkSize)
                .reader(jdbcBatchItemWriterReader())
                .writer(jdbcBatchItemWriter())
                .build();
    }

    @Bean
    public JdbcCursorItemReader<Pay> jdbcBatchItemWriterReader(){
        return new JdbcCursorItemReaderBuilder<Pay>()
                .fetchSize(chunkSize)
                .dataSource(dataSource)
                .rowMapper(new BeanPropertyRowMapper<>(Pay.class))
                .sql("SELECT id, amount, tx_name, tx_date_time FROM pay")
                .name("jdbcBatchItemWriter")
                .build();
    }

    @Bean
    public JdbcBatchItemWriter<Pay> jdbcBatchItemWriter(){
        return new JdbcBatchItemWriterBuilder<Pay>()
                .dataSource(dataSource)
                .sql("insert into pay2(amount, tx_name, tx_date_time) values (:amount, :txName, :txDateTime)")
                .beanMapped()
                .build();
    }
}
```

여기서 나오는 JdbcBatchItemWriterBuilder는 아래와 같은 설정값을 가진다.

| Property      | Parameter Type | 설명                                                         |
| ------------- | -------------- | ------------------------------------------------------------ |
| assertUpdates | boolean        | 적어도 하나의 항목이 행을 업데이트하거나 삭제하지 않을 경우 예외를 throw할지 여부를 설정한다. 기본값은 true. Exception:EmptyResultDataAccessException |
| columnMapped  | x              | Key,Value 기반으로 InsertSQL의 Values를 매핑 (ex: Map<String,Object>) |
| beanMapped    | x              | Pojo 기반 InsertSQL의 Values를 매핑                          |



##### columnMapped vs beanMapped

위의 예제의 beabMapped를 columnMapped로 바꾸면 아래와 같다.

```java
new JdbcBatchItemWriterBuilder<Map<String, Object>>() // Map 사용
                .columnMapped()
                .dataSource(this.dataSource)
                .sql("insert into pay2(amount, tx_name, tx_date_time) values (:amount, :txName, :txDateTime)")
                .build();
```

Reader에서 Writer로 넘겨주는 타입이 Map<String, Object>냐 , Pay.class와 같은 Pojo 타입이냐가 다른 점이다.

values(:field) 부분을 보면 Dto의 Getter 혹은 Map의 Key에 매핑되어 값이 할당 된다.



추가로 JdbcBatchItemWriter의 설정에서 주의할 점이 있다.

- JdbcBatchItemWriter의 제네릭 타입은 **Reader**에서 넘겨주는 값의 타입이다.



오해를 하지 않도록 조심하자.

Pay2 테이블에 데이터를 넣은 Writer이지만 선언된 제네릭 타입은 Reader/Processor에서 넘겨준 Pay클래스이다.

추가로 afterPropertiesSet 메소드를 확인해보자.

이 메소드는 InitializingBean 인터페이스에서 갖고 있는 메소드이다.



이 메소드를 확인하는 이유는 ItemWriter의 구현체들이 모드 InitializingBean 인터페이스를 구현하고 있는데, 각각의 Writer들이 실행되기 위해 필요한 필수값들이 제대로 세팅되어있는지를 체크해준다.

Writer를 생성하고 afterPropertiesSet 메소드를 그 아래에서 바로 실행하면 어느 값이 누락되었는지 명확하게 인지할 수 있다!!



-----

##### Custom ItemWriter

Writer는 Custom 해서 구현 할 일이 많다.

예를 들어

- Reader에서 읽어온 데이터를 RestTemplate으로 외부 API로 전달해야 할 때
- 임시저장을 하고 비교하기 위해 싱글톤 객체에 값을 넣어야 할 때
- 여러 Entity를 동시에 save 할 때 (JPA)



이렇게 Spring Batch에서 공식적으로 지원하지 않는 Writer를 사용하고 싶을 때 ItemWriter 인터페이스를 구현한다.

processor에서 넘어온 데이터를 System.out.println 으로 출력하는 Writer를 만든 경우를 보자.

```java
@Slf4j
@RequiredArgsConstructor
@Configuration
public class CustomItemWriterJobConfiguration {
    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    private final EntityManagerFactory entityManagerFactory;

    private static final int chunkSize = 10;

    @Bean
    public Job customItemWriterJob() {
        return jobBuilderFactory.get("customItemWriterJob")
                .start(customItemWriterStep())
                .build();
    }

    @Bean
    public Step customItemWriterStep() {
        return stepBuilderFactory.get("customItemWriterStep")
                .<Pay, Pay2>chunk(chunkSize)
                .reader(customItemWriterReader())
                .processor(customItemWriterProcessor())
                .writer(customItemWriter())
                .build();
    }

    @Bean
    public JpaPagingItemReader<Pay> customItemWriterReader() {
        return new JpaPagingItemReaderBuilder<Pay>()
                .name("customItemWriterReader")
                .entityManagerFactory(entityManagerFactory)
                .pageSize(chunkSize)
                .queryString("SELECT p FROM Pay p")
                .build();
    }

    @Bean
    public ItemProcessor<Pay, Pay2> customItemWriterProcessor() {
        return pay -> new Pay2(pay.getAmount(), pay.getTxName(), pay.getTxDateTime());
    }

    @Bean
    public ItemWriter<Pay2> customItemWriter() {
        return new ItemWriter<Pay2>() {
            @Override
            public void write(List<? extends Pay2> items) throws Exception {
                for (Pay2 item : items) {
                    System.out.println(item);
                }
            }
        };
    }
}
```

코드는 이전에 작성한 코드와 비슷하다.

write()만 @Override 해서 작성하면 구현체 생성이 끝난다.

 