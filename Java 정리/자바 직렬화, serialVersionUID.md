## 자바 직렬화, serialVersionUID 

기존 정리 내용에 추가하기



#### serialVersionUID

클래스를 역직렬화 할 때, 멤버 변수가 새로 추가 되었다면 예외가 발생한다.

- `java.io.InvalidClassException`

- ```
  local class incompatible: stream classdesc serialVersionUID = -8896802588094338594, local class serialVersionUID = 7127171927132876450
  ```



serialVersionUID의 정보가 일치하지 않기 때문에 발생한 것이다.



>  serialVersionUID 값을 설정하지도 않았는데 왜 이런 에러가 발생할까?

- SUID 는 필수 값은 아니다.
- 호환 가능한 클래스는 SUID값이 고정되어 있다.
- SUID가 선언되어 있지 않으면 클래스의 기본 해쉬값을 사용한다.



> 그럼 어떨 때 SUID를 설정해줘야할까?

```
private static final long serialVersionUID = 1L;
```

값을 추가해줬을 때, 멤버 변수를 추가하면, 해당 데이터는 역직렬화 대상에서 제외되어

`null`값이 출력된다.



> 반면 멤버 변수의 타입이 바뀐다면?

- `java.lang.ClassCastException: cannot assign instance of java.lang.String to field com.serial.Member.email of type java.lang.StringBuilder in instance of com.serial.Member`

위와 같은 에러가 발생한다.



#### 정리

- 직렬화는 상당히 타입의 변화에 엄격하다는 것을 알 수 있다. 역직렬화 대상의 클래스의 멤버 변수 타입 변경을 지양해야한다.

- 특별한 문제가 없으면 직렬화 버전 `serialVersionUID`의 값은 개발 시 직접 관리해야한다.

- `serialVersionUID`의 값이 동일하면 멤버 변수 및 메소드 추가는 문제되지 않는다.

  추가된 멤버 변수 및 메소드의 데이터는 누락된다.

- 외부(DB, 캐시 서버, NoSQL 서버 등)에 장기간 저장될 정보는 직렬화 사용을 지양해야 한다.

  역직렬화 대상의 클래스가 언제 변경이 일어날지 모르는 환경에서 긴 시간 동안 외부에 존재했던 직렬화된 데이터는 쓰레기(Garbage)가 될 가능성이 높다.

  `언제 예외가 발생할지 모른다`

- 직접 컨트롤 가능한 클래스의 객체가 아닌 클래스의 객체에 대해서는 직렬화를 지양해야 한다.

  ex)

  1. 프레임워크 or 라이브러리가 버전업 되면서 `serialVersionUID`가 변경됨
  2. 테스트시에는 발생 안 하다가 운영에 반영



#### 결론

- 직렬화를 사용할 때에는 될 수 있으면 자주 변경되는 클래스의 객체는 사용을 지양한다.

  변경에 취약하기 때문에 생각지도 못한 예외가 발생할 수 있다.

  `특히 역직렬화가 되지 않을 때와 같은 예외처리는 하는것을 추천한다.`

  

