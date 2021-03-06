## 옵저버 패턴 (Observer Pattern)

객체에 이벤트가 발생했을 때, 객체와 관련된 객체들(옵저버들)에게 이벤트를 인식하도록 하는 디자인 패턴.

객체의 상태가 변경되었을 때, 특정 객체에 의존하지 않으면서 상태의 변경을 관련된 객체들에게 통지하는 것.

**`Pub/Sub`** 모델로 불리기도 한다.



구독하면 제일 먼저 떠오르는 유튜브를 예로 들어보자.

Pub/Sub 모델에서 유튜브의 크리에이터들은 Pub가 되고 유저들은 Sub(옵저버)가 된다.

구독을 한 크리에이터가 영상을 올리면 구독한 옵저버들은 알림을 받을 수 있다.

구독한 유저들을 `옵저버`라고 할 수 있다.



## 스테이트 패턴(State Pattern)

객체가 특정 상태에 따라 행위를 달리하는 상황에서, 

자신이 직접 상태를 체크하여 상태에 따른 행위를 호출하지 않고,

상태를 개체화 하여 상태가 행동을 할 수 있도록 위임하는 패턴



객체의 특정 상태를 클래스로 선언하고, 클래스에서는 해당 상태에서 할 수 있는 행위들을 메소드로 정의한다.

각 상태 클래스들을 인터페이스로 캡슐화 하여, 클라이언트에서 인터페이스를 호출하는 방식이다.



## **전략 패턴 ( Strategy Pattern )**

객체들이 할 수 있는 행위 각각에 대해 전략 클래스를 생성하고, 유사한 행위들을 캡슐화 하는 인터페이스를 정의하여,

객체의 행위를 동적으로 바꾸고 싶은 경우 직접 행위를 수정하지 않고 전략을 바꿔주기만 함으로써 행위를 유연하게 확장하는 방법을 말합니다.



간단히 말해서 객체가 할 수 있는 행위들 각각을 전략으로 만들어 놓고, 동적으로 행위의 수정이 필요한 경우 전략을 바꾸는 것만으로 행위의 수정이 가능하도록 만든 패턴입니다.