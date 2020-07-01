## [Java] Primitive Type VS Reference Type

자바에서 타입은 크게 기본형 타입과 참조형 타입이 있다.



#### 기본형 타입(Primitive Type)

- 기본 자료형은 반드시 사용하기 전에 선언되어야 한다.

- `비객체` 타입이기 때문에 `null`값을 가질 수 없다. 

  기본형 타입에 Null을 넣고 싶다면 `래퍼 클래스`를 활용해야한다.

- 실제 값을 저장하는 공간으로 `스택(Stack)` 메모리에 저장된다.

- 컴파일 시점에 데이터의 표현 범위를 벗어나면 컴파일 에러가 발생한다.

- boolean, byte, short, int, long, float, double, char



#### 참조형 타입(Reference Type)

- 기본형 타입을 제외한 타입들이 모두 참조형 타입이다.

  기본적으로 `java.lang.Object`를 상속받으면 참조형이 된다.

- 빈 객체를 의미하는 `Null`이 존재한다.

- 값이 저장되어 있는 곳의 `주소값`을 저장하는 공간으로 `힙(Heap)` 메모리에 저장된다.

- 런타임 에러가 발생한다.

  배열을 Null로 받으면 NPE 발생.

- 배열(Array), 열거(Enumeration), 클래스, 인터페이스

