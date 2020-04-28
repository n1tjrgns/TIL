# Item 15. 클래스와 멤버의 접근 권한은 최소화 하라

잘 설계된 모듈은 구현 세부사항을 전부 API 뒤쪽에 감춘다.

그리고 해당 API를 통해서만 서로 통신하며, 각자 내부적으로는 무슨 일을 하는지 신경쓰지 않는다.

이 개념을 **정보은닉, 캡슐화**라고 한다.

##### 정보 은닉의 장점

1. 시스템 개발 속도를 높인다.
2. 시스템 관리 비용을 낮춘다.
3. 성능 최적화에 도움을 준다.
4. 소프트웨어 재사용성을 높인다.
5. 큰 시스템을 제작하는 난이도를 낮춰준다.(testcase 작성)



- ##### 각 클래스와 멤버는 가능한 접근 불가능하도록 만들어라

정상적인 동작을 보증하는 한도 내 가장 낮은 접근 권한을 설정해라

최상위 레벨 클래스나 인터페이스는 package-private로 선언하면 API의 일부가 아니라 구현 세부사항에 속하게 되어, 자유롭게 변경, 삭제 할 수 있다.

- package-private : 아무런 접근제어도 주지 않는다.

  같은 패키지 안에서만 사용 가능

```java
public class Access{
    ...
}
```



package-private로 선언된 최상위 레벨 클래스나 인터페이스가 하나뿐이면, priavate static으로 중첩 클래스를 만드는 것을 추천한다.

=> 패키지 전체가 아니라 단 하나의 클래스만이 해당 클래스 접근 권한을 가진다.



public 클래스의 접근 권한을 낮추는게 더 중요하다.

=> public 클래스는 해당 패키지 API의 일부이다.



private < package-private < protected < public



상위 클래스 메소드를 재정의 할 때 원래 메소드 접근 권한보다 낮은 권한을 설정할 수 없다.

=> 이 규칙을 어기면 컴파일 시 오류가 발생한다.

=> 그래서 특정 인터페이스를 제공할 때는 모든 메소드를 public 메소드로 선언한다.



객체 필드는 절대 public 으로 선언하면 안된다.

=> final을  사용해 저장값에 제한을 할 수 없다.

=> 다중 스레드에 안전하지 않다.



길이가 0이 아닌 배열은 언제나 변경 가능하기 때문에,

public static final 배열 필드를 두거나, 배열 필드를 반환하는 접근자를 정의하면 안 된다.

이런 필드들은 관습적으로 대문자로 구성된 이름을 가지며, 밑줄 기호로 구분한다.

```java
public static final Thing[] VALUES = {}; // x
```

이 문제를 해결하기 위해 두 가지 방법이 있다.

1. public으로 선언된 배열을 private로 바꾸고, 변경이 불가능한 public 리스트를 하나 만든다.

   ```java
   private static final Thing[] Private_VALUES={};
   public static final List<Thing> VALUES = Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
   
   ```

2. 배열을 private로 선언하고, 해당 배열을 복사해서 반환하는 public 메소드를 만든다.

   ```
   private static final Thing[] Private_VALUES={};
   public static final Thing[] values(){
       return PRIVATE_VALUES.clone();
   }
   ```





- 접근 권한은 가능한 낮추자.
- public static final 필드를 제외한 어떤 필드도 public 으로 선언하지 말자.









<https://jaehun2841.github.io/2019/01/19/effective-java-item15/#%EC%9D%B4%EB%84%88%ED%81%B4%EB%9E%98%EC%8A%A4-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0>