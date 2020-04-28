## Atomic Type vs Synchronized



스프링 REST API 기본 예제를 보고있었는데 `AtomicLong` 이라는 타입으로 변수를 선언해 놓은 부분이 있었다.

```
private final AtomicLong counter = new AtomicLong();
```



> 이게 뭘까??



#### Atomic 변수

- 원자성을 보장하는 변수
- 멀티쓰레드 환경에서 동기화 문제를 `synchronized` 키워드를 사용하여 락을 거는데, 해당 키워드 없이 동기화 문제를 해결하기 위해 고안된 방법이다.



#### 동기화 문제 해결방법

- synchronized
- Atomic
- volatile



#### Atomic VS synchronized 차이점

- synchronized 

  특정 Thread가 `해당 블럭 전체를 lock` 하기 때문에, 다른 Thread는 아무런 작업을 하지 못하고 기다리는 상황 발생 -> 자원의 낭비



- Atomic

   `NonBlocking` 상태로 동기화 문제를 해결 할 수 있다.



#### Atomic의 경우

NonBlocking이 가능한 이유는 바로 `CAS(Compared And Swap)` 알고리즘 덕분이다.



멀티쓰레드, 멀티코어 환경에서는 `각 CPU는 메인 메모리에서 변수값을 참조하지않는다.`

`각 CPU의 캐시 영역`에서 메모리를 참조하게된다.



이때, 메인 메모리에 저장된 값과 CPU 캐시에 저장된 값이 다른 경우(가시성 문제) 사용되는 것이 `CAS알고리즘`이다.



현재 쓰레드에 저장된 값과 메인 메모리에 저장된 값을 비교하여 

- 일치하는 경우 새로운 값으로 교체 
- 일치하지 않는다면 실패 후 재시도

해당 방법을 통해 CPU 캐시에 잘못된 값을 참조하는 가시성문제를 해결 할 수 있다.



#### synchronized의 경우 

- synchronized 진입전 메인 메모리와 CPU 캐시메모리 값을 동기화하여 문제가 없도록 처리한다.





#### AtomicInteger 클래스

```
public class AtomicInteger extends Number implements java.io.Serializable {

	private volatile int value; 
        
        public final int incrementAndGet() { 
	    int current; 
            int next;
            do { 
	        current = get(); 
                next = current + 1; 
            } while (!compareAndSet(current, next));
            return next; 
        } 
        
        public final boolean compareAndSet(int expect, int update) {
        	return unsafe.compareAndSwapInt(this, valueOffset, expect, update); 
        }	
            
}
```

AtomicInteger 클래스의 내부 모습이다.

incrementAndGet() 메소드 안에 있는 compareAndSet 메소드가 바로 `CAS 알고리즘`이다.



compareAndSet을 호출하여, 그 결과값이 성공일 때 까지 while을 통해 무한루프를 돈다.

compareAndSet 내부에서는 compareAndSwapInt를 호출해 `메모리에 저장되어진 값과 현재 CPU에 캐시된 expect 값을 비교해 동일한 경우만 update`를 실행한다.



또한 value 값의 선언을 보면

```
private volatile int value;
```

volatile 키워드로 선언이 되어있다.

> 이건 뭐야??



#### Volatile

해당 키워드가 붙어있는 객체는 CPU캐시가 아닌 메모리에서 값을 참조한다.

> 메모리에서 어차피 직접 참조하는데 왜 CAS알고리즘으로 비교를 또해???



volatile은 `오직 1개의  쓰레드에서 쓰기 작업`을 하고 `다른 쓰레드는 읽기 작업`만을 할 때 `안정성이 보장`된다.



하지만 AtomicIntever는 여러 쓰레드에서 읽기/쓰기 작업을 병행한다.

- 그래서 CAS 알고리즘을 사용해 2중 안전장치를 걸어놓는다.

