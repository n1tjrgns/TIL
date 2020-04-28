# Chunk 지향 처리

##### Chunk란?

Spring Batch에서 Chunk는 데이터 덩어리로써 작업 할 때 각 커밋 사이에 처리되는 row 수를 얘기한다. **한번에 하나씩 데이터를 읽어 Chunk라는 덩어리를 만든 뒤, Chunk 단위로 트랜잭션**을 다루는 것을 의미한다.

Chunk 단위로 트랜잭션을 수행하기 때문에 **실패할 경우 해당 Chunk 만큼 롤백**이 되고, 이전에 커밋된 트랜잭션 범위까지만 반영이 된다.



- Reader에서 데이터를 하나 읽어온다.
- 읽어온 데이터를 Processor에서 가공한다.
- 가공된 데이터들을 별도의 공간에 모은 뒤, Chunk 단위만큼 쌓이게 되면 Writer에 전달하고 Writer는 일괄 저장한다.

**Reader와 Processor 에서는 1건씩 다뤄지고, Writer에서는 Chunk 단위로 처리** 된다.



 Java 코드 예제를 보자

```java
for(int i=0; i<totalSize; i+=chunkSize){ //chunkSize 단위로 묶어서 처리
    List items = new ArrayList();
    for(int j=0; i<chunkSize; j++){
        Object item = itemReader.read()
        Object processedItem = itemProcessor.process(item);
        items.add(processedItem);
    }
    itemWriter.write(items);
}
```



그럼 이제 Chunk 지향 처리가 어떻게 되고 있는지 Spring Batch 내부 코드를 보면서 알아보자.



##### Chunk 지향 처리 과정

Chunk 지향 처리의 전체 로직을 다루는 것은 **ChunkOrientedTasklet** 클래스이다.

```java
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.springframework.batch.core.StepContribution;
import org.springframework.batch.core.scope.context.ChunkContext;
import org.springframework.batch.core.step.tasklet.Tasklet;
import org.springframework.batch.repeat.RepeatStatus;

/**
 * A {@link Tasklet} implementing variations on read-process-write item
 * handling.
 *
 * @author Dave Syer
 *
 * @param <I> input item type
 */
public class ChunkOrientedTasklet<I> implements Tasklet {

	private static final String INPUTS_KEY = "INPUTS";

	private final ChunkProcessor<I> chunkProcessor;

	private final ChunkProvider<I> chunkProvider;

	private boolean buffering = true;

	private static Log logger = LogFactory.getLog(ChunkOrientedTasklet.class);

	public ChunkOrientedTasklet(ChunkProvider<I> chunkProvider, ChunkProcessor<I> chunkProcessor) {
		this.chunkProvider = chunkProvider;
		this.chunkProcessor = chunkProcessor;
	}

	/**
	 * Flag to indicate that items should be buffered once read. Defaults to
	 * true, which is appropriate for forward-only, non-transactional item
	 * readers. Main (or only) use case for setting this flag to false is a
	 * transactional JMS item reader.
	 *
	 * @param buffering indicator
	 */
	public void setBuffering(boolean buffering) {
		this.buffering = buffering;
	}

	@Override
	public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {

		@SuppressWarnings("unchecked")
		Chunk<I> inputs = (Chunk<I>) chunkContext.getAttribute(INPUTS_KEY);
		if (inputs == null) {
			inputs = chunkProvider.provide(contribution);
			if (buffering) {
				chunkContext.setAttribute(INPUTS_KEY, inputs);
			}
		}

		chunkProcessor.process(contribution, inputs);
		chunkProvider.postProcess(contribution, inputs);

		// Allow a message coming back from the processor to say that we
		// are not done yet
		if (inputs.isBusy()) {
			logger.debug("Inputs still busy");
			return RepeatStatus.CONTINUABLE;
		}

		chunkContext.removeAttribute(INPUTS_KEY);
		chunkContext.setComplete();

		if (logger.isDebugEnabled()) {
			logger.debug("Inputs not busy, ended: " + inputs.isEnd());
		}
		return RepeatStatus.continueIf(!inputs.isEnd());

	}

}
```



여기서 자세하게 봐야할 코드는 **execute**()이다.

Chunk 단위로 작업하기 위한 전체 코드가 여기 있다고 보면 된다.

해당 부분의 코드만 가져오면

```java
if (inputs == null) {
    		//Reader에서 데이터를 가져오는 부분
			inputs = chunkProvider.provide(contribution);
			if (buffering) {
				chunkContext.setAttribute(INPUTS_KEY, inputs);
			}
		}
		//Processor & Writer를 처리하는 부분 
		chunkProcessor.process(contribution, inputs);

		chunkProvider.postProcess(contribution, inputs);
```

- chunkProvider.provide()로 Reader에서 Chunk Size 만큼 데이터를 가져온다.

- chunkProcessor.process() 에서 Reader로 받은 데이터를 가공(Processor)하고 저장(Writer)한다.



chunkProvider.provide()를 따라가보면 데이터를 어떻게 가져오는지 알 수 있다.

```java
@Override
	public Chunk<I> provide(final StepContribution contribution) throws Exception {

		final Chunk<I> inputs = new Chunk<>();
        //반복자
		repeatOperations.iterate(new RepeatCallback() {

			@Override
			public RepeatStatus doInIteration(final RepeatContext context) throws Exception {
				I item = null;
				try {
                    //Reader.read()
					item = read(contribution, inputs);
				}
				catch (SkipOverflowException e) {
					// read() tells us about an excess of skips by throwing an
					// exception
					return RepeatStatus.FINISHED;
				}
				if (item == null) {
					inputs.setEnd();
					return RepeatStatus.FINISHED;
				}
				inputs.add(item); //반복자가 끝날때 까지 inputs에 추가
				contribution.incrementReadCount();
				return RepeatStatus.CONTINUABLE;
			}

		});

		return inputs;

	}
```

inputs가 CunkSize만큼 쌓일때까지 read()를 호출한다.

또한 read()의 내부를 보면 실제로는 ItemReader.read를 호출한다.

```java
@Nullable
	protected I read(StepContribution contribution, Chunk<I> chunk) throws SkipOverflowException, Exception {
		return doRead();
	}
```

```java
protected final I doRead() throws Exception {
    try{
        listener.beforeRead();
        I item = itemReader.read();
        if(item != null) {
            listener.afterRead(item);
        }
        return item;
    }
    catch (Exception e){
        if(logger.isDebugEnabled()){
            logger.debug(e.getMessage() + " : " + 					 													e.getClass().getName());
        }
        listener.onReadError(e);
        throw e;
    }
}
```

ItemReader.read에서 1건씩 데이터를 조회해 ChunkSize 만큼 데이터를 쌓는 것이 provide()가 하는 일이다.

쌓인 데이터는 어떻게 가공되고 저장될까??



##### SimpleChunkProcessor.Class

Processor와 Writer 로직을 담고 있는 것은 ChunkProcessor가 담당하고 있다.

```java
/**
 * Interface defined for processing {@link Chunk}s. 
 *
 * @since 2.0
 */
public interface ChunkProcessor<I> {
	
	void process(StepContribution contribution, Chunk<I> chunk) throws Exception;

}
```



이 인터페이스의 실제 구현체로써 기본적으로 사용되는 것이 SimpleChunkProcessor이다.

```java

public class SimpleChunkProcessor<I, O> implements ChunkProcessor<I>, InitializingBean {

	private ItemProcessor<? super I, ? extends O> itemProcessor;

	private ItemWriter<? super O> itemWriter;

	private final MulticasterBatchListener<I, O> listener = new MulticasterBatchListener<>();

	/**
	 * Default constructor for ease of configuration.
	 */
	@SuppressWarnings("unused")
	private SimpleChunkProcessor() {
		this(null, null);
	}

	
	@Override
	public final void process(StepContribution contribution, Chunk<I> inputs) throws Exception {

		// Allow temporary state to be stored in the user data field
		initializeUserData(inputs);

		// If there is no input we don't have to do anything more
		if (isComplete(inputs)) {
			return;
		}

		// Make the transformation, calling remove() on the inputs iterator if
		// any items are filtered. Might throw exception and cause rollback.
		Chunk<O> outputs = transform(contribution, inputs);

		// Adjust the filter count based on available data
		contribution.incrementFilterCount(getFilterCount(inputs, outputs));

		// Adjust the outputs if necessary for housekeeping purposes, and then
		// write them out...
		write(contribution, inputs, getAdjustedOutputs(inputs, outputs));

	}


```

처리를 담당하는 핵심 로직은 process() 이므로 해당 코드를 보자.



Chunk<O> outputs = transform(contribution, inputs);

- Chunk<I> inputs를 파라미터로 받는다.
  - 이 파라미터는 앞부분에서 chunkProvider.provide()에서 받은 ChunkSize만큼 쌓인 item이다.
- transform() 에서는 전달받은 inputs를 doProcess()로 전달하고 변환값을 받는다.
- transform() 을 통해 가공된 대량의 데이터는 write()를 통해 일괄 저장된다.
  - write()는 저장이 될 수 있고, 외부 API로 전송할 수 도 있습니다.
  - 개발자가 ItemWriter를 어떻게 구현했는지에 따라 달라진다.





transfrom()은 반복문을 통해 doProcess() 를 호출한다.

해당 메소드는 ItemProcessor의 process()를 사용한다.

```java
protected Chunk<O> transform(StepContribution contribution, Chunk<I> inputs) throws Exception {
		Chunk<O> outputs = new Chunk<>();
		for (Chunk<I>.ChunkIterator iterator = inputs.iterator(); iterator.hasNext();) {
			final I item = iterator.next();
			O output;
			try {
				output = doProcess(item);
			}
			catch (Exception e) {
				/*
				 * For a simple chunk processor (no fault tolerance) we are done
				 * here, so prevent any more processing of these inputs.
				 */
				inputs.clear();
				throw e;
			}
			if (output != null) {
				outputs.add(output);
			}
			else {
				iterator.remove();
			}
		}
		return outputs;
	}
```





doProcess()를 처리할 때 만약 ItemProcessor가 없다면 item을 그대로 반환하고 있다면 ItemProcessor의 process()로 가공하여 반환한다.

```java
protected final O doProcess(I item) throws Exception {

		if (itemProcessor == null) {
			@SuppressWarnings("unchecked")
			O result = (O) item;
			return result;
		}

		try {
			listener.beforeProcess(item);
			
			//이 부분
			O result = itemProcessor.process(item);
			
			listener.afterProcess(item, result);
			return result;
		}
		catch (Exception e) {
			listener.onProcessError(item, e);
			throw e;
		}

	}
```





그리고 이렇게 가공된 데이터들은 SimpleChunkProcessor의 doWrite()를 호출해 일괄 처리 한다.

```java
protected final void doWrite(List<O> items) throws Exception {

		if (itemWriter == null) {
			return;
		}

		try {
			listener.beforeWrite(items);
			
			//이 부분
			writeItems(items);
			
			doAfterWrite(items);
		}
		catch (Exception e) {
			doOnWriteError(e, items);
			throw e;
		}

	}
```



-----

Page Size vs Chunk Size

Spring Batch의 Reader의 종류 중 PagingItemReader가 있다.

여기서 Page Size를 사용하곤 하는데, Chunk Size와는 개념이 다르다.



Page Size는 한번에 조회할 Item의 양을 나타낸다.

반면에, Chunk Size는 한번에 처리될 트랜잭션 단위를 얘기한다.









**참고**

https://jojoldu.tistory.com/331?category=635883