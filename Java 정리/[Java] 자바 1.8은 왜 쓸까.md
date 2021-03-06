## [Java] 자바 1.8은 왜 쓸까?

나는 단순하게 [람다식](https://n1tjrgns.tistory.com/227)을 사용 할 수 있어서? 라고 밖에 생각이 들지 않는다.

그래서 갑자기 궁금해졌다..!



자바 1.8을 검색했을 때 가장 많이 나오는 단어가 `함수형 프로그래밍` 이었다.



>  함수형 프로그래밍??

자바를 주로하는 나에게 객체지향 프로그래밍도아닌 `함수형 프로그래밍`은 낯선 단어이다.



#### 객체지향 프로그래밍 vs 함수형 프로그래밍

- 객체지향은 `명령형`프로그래밍이고, 함수형은 `선언형` 프로그래밍이다.

- 두 프로그래밍 방식의 가장 큰 차이점은 `문제해결의 관점`이다.

  객체지향 : 데이터를 어떻게 처리할 지에 대해 명령을 통해 풀어 나간다.

  함수형 : 선언적 함수를 통해 무엇을 풀어나갈지 결정한다.



> 흠.. 아직 그래도 어렵다.



#### 함수형 프로그래밍

- 대입문이 없어 변수에 값이 할당되면 그 이후 절대 변하지않는다.

  = `side effect`가 전혀 없다.

  표현식의 결과를 바꿀 수 있는 부수 효과가 없기 때문에 아무때나 실행될 수 있다.
  
  = `참조 투명성`을 가진다.
  
  = 흐름의 제어를 지정해야하는 프로그래머의 짐을 덜어준다.
  
  

> 어떻게 side effect가 없을 수 있지??



#### Side effect가 없다?

- 함수형 프로그래밍은 함수들의 조합으로 만들어진다. 각 함수들은 인자를 받고, 그에따른 결과를 반환하고 내부적으로 어떠한 상태도 가지지 않는다.

  - 따라서, 내부에서 어떤 일이 벌어지는지 상관하지않는다.

    함수 호출 시 `입력 값과 반환 값만 중요`하다.

- 변수의 변형이 없다.(변수를 사용하지 않기 때문에)
- 파일, DB, 네트워크 등등 데이터를 쓰지 않는다(writing).
- 예외가 발생하지 않는다.



> DB도 안쓰는데 어플을 어떻게 만들어??

- 그래서 순수 함수로만 작성된 프로그램은 있을 수가 없다.
- side effect는 프로세스의 진행 중간에 발생하는 것 보다 모든 연산이 종료되는 시점에 발생하는 것이 좋다.



#### 동시성 프로그래밍

반도체 성능 향상의 한계에 다다르자 CPU 회사들은 칩 속도를 높이는 방식 대신, 여러 개의 칩이 병렬적으로 동작하도록해 성능을 높이는 방식을 사용하고 있다.

어플리케이션에서는 `멀티쓰레드`를 사용해 CPU 코어를 최대한 활용해야 한다는 것이다.

명령형 프로그래밍에서 멀티스레드를 활용한 동시성 프로그래밍은 개발자들을 아주 힘들게 한다.

`교착상태(deadlock)`에 빠질 위험이 존재하며, 해결하기 위해 고려해야하는 요소들이 많다.

명령형 프로그래밍에서 이러한 문제가 발생하는 주 원인은 쓰레드 간 공유되는 데이터의 상태 값이 `변경 가능(mutable)`하기 때문이다.

하지만, 함수형 프로그래밍에서는 사용하는 모든 데이터가 `변경 불가능(immutable)`하고 함수는 `side effect`를 가지지 않는다.

여러 쓰레드가 동시에 공유 데이터에 접근하더라도 해당 데이터가 변경될 수 없기 때문에 동시성과 관련된 문제를 원천적으로 봉쇄한다.



#### 1 ~ 100까지의 짝수 합 구하기 (1.7 이하 버전)

```
public int evenSum(){
	int sum = 0;
	for(int i=1; i<=100; i++){
		 if ( i % 2 == 0)
             sum += i;
	}
	
	return sum;
}

```



#### 1.8 버전

```
public int evenSum(){
        return IntStream.rangeClosed(1, 100)
                .filter( i -> i % 2 == 0)
                .reduce(0, Integer::sum);
    }
```



#### 요약

 함수형 프로그래밍이란 패러다임이 생긴지는 꽤나 오래되었다. 

그 동안 OOP에 비해 주목을 받지 못하다 처리해야 할 데이터나 트래픽은 기하급수적으로 증가하여 멀티 쓰레드 환경에서의 작업이 중요해진 최근 상황에서야 주목을 받는 것 같다. 





#### 피드백

생활코딩에 글을 공유했더니, 피드백을 받아 아래에 추가하고자한다.



**장준영, 서재원님**

 순수한 함수로만 된 프로그램은 있을 수 없다.

- Monad로 순수함수 프로그램을 짤 수 있다 .



**양욱진님**

 객체지향과 함수지향을 대결 구도로 두는 행위는 절대로 하면 안 된다. 

애초부터 자바는 객체지향이었고, 함수형 지원한다고 해서 함수형 언어가 되는것이 아니기 때문. 

자바가 함수형을 지원한다 해도 영원한 객체지향입니다. 설계가 처음부터 끝까지 그랬으니까. 

개발 언어는 개발 언어대로, 패턴은 패턴대로! 이렇게 접근하는 것을 추천. 

자바는 객체지향이었다가 함수형을 지원하기 시작했는데, 함수형을 사용하면 단순 객체지향에 대한 가독성과 간결함을 통한 멀티쓰레드 접근성 향상을 꾀할 수 있다. (뭐 성능 차이에 대해서는 논란이긴 하지만 결국 하기 나름이죠.) 



#### 참고

 https://jinbroing.tistory.com/229 

 http://ruaa.me/why-functional-matters/ 

