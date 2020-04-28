## 자바 메모리 관리 Stack & Heap

너무 코딩을 할 때, 코드만 보고 따라친다는 느낌이 많은 요즘이다.

원리는 제대로 알고 하고있는걸까,,



#### Stack

- Heap 영역에 생성된 Object 타입의 데이터의 참조값이 할당된다.
- 원시타입의 데이터가 값과 함께 할당된다.
- 지역변수들은 scope에 따른 visibility를 가진다.
- 각 Thread는 자신만의 stack을 가진다.



Stack 영역에는 Heap 영역에서 생성된 Object 타입의 데이터들에 대한 참조를 위한 값들이 할당된다. 또한, 원시타입(primitive - types) - byte, short, int, long, double, float, boolean, char 타입의 데이터들이 할당된다.

이때 원시타입의 데이터들에 대해서는 참조값이 아닌, 실제 값을 stack에 직접 저장하게 된다.



![1563694400142](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1563694400142.png)

Stack 영역에 있는 변수들은 visibility를 가진다. 변수에 대한 개념이다.

전역변수가 아닌 지역변수가 foo()라는 함수 내에서 Stack에 할당 된 경우, 해당 지역변수는 다른 함수에서 접근할 수 없다. 

예를들어, foo() 라는 함수에서 bar()를 호출하고 bar() 함수가 종료된 경우, bar() 함수 내부에서 선언한 모든 지역변수들은 stack에서 pop 되어 사라진다.



Stack 메모리는 Thread 하나당 하나씩 할당된다. 즉, 스레드 하나가 새롭게 생성되는 순간 해당 스레드를 위한 stack 도 함께 생성되며, 각 스레드에서 다른 스레드의 stack 영역에는 접근할 수 없다.



##### 예제)

```java
public class Main {
	public static void main(String[] args){
		int argument = 4;
		argument = someOperation(argument);
	}
	
	private static int someOperation(int param){
		int tmp = param * 3;
		int result = tmp / 2;
		return result;
	}
}
```

argument에 4를 할당한 후, 이 변수를 함수에 넘겨주어 연산 후 결과 값을 return 받아 argument에 저장된다.



`int argument = 4 ;` 에 의해 스택에 argument라는 변수명으로 공간이 할당된다.

또한 argument의 변수 타입은 원시타입이기 때문에 실제 4 라는 값이 할당된다.

현재 스택의 상태는 아래와 같다.

![1563694634513](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1563694634513.png)



다음은 `argument = someOperation(argument);`

someOperation 함수가 호촐될 때 인자로 argument를 넘겨주며, scope가 someOperation() 함수로 이동한다. scope가 바뀌면서 기존의 argument 값은 scope 에서 벗어나 사용할 수 없다.

이때 인자로 넘겨받은 값은 파라미터인 param에 복사되어 전달되는데, param 또한 원시타입이므로 stack에 할당된 공간에 값이 할당된다. 

![1563694781510](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1563694781510.png)

그 다음,

```java
int tmp = param * 3; //12
int result = tmp / 2; //6
```

![1563694845719](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1563694845719.png)

그 후 someOperation()의 메소드가 끝나는 지점 `}` 에 다다라  함수가 종료되면 호출함수 scope에서 사용되었던 모든 지역변수들은 stack에서 pop 된다.

함수가 종료되어 지역변수들이 모두 pop 되고, 함수를 호출했던 시점으로 돌아가면 스택의 상태는 아래와 같이 변한다.

![1563694967194](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1563694967194.png)

argument 변수는 4로 초기화 되었지만, 함수의 실행결과인 6이 기존 argument 변수에 재할당된다.

물론 함수호출에서 사용되었던 지역변수들이 모두 pop 되기 전에 재할당 작업이 일어난다.



그리고 main() 함수도 종료되는 순간 stack 에 있는 모든 데이터들은 pop 되면서 프로그램이 종료된다.



#### Heap

- Heap 영역에는 주로 긴 생명주기를 가지는 데이터들이 저장된다.
  - 대부분의 오브젝트는 크기가 크고, 서로 다른 코드블럭에서 공유되는 경우가 많다.
- 어플리케이션의 모든 메모리 중 stack에 있는 데이터를 제외한 부분이라고 보면 된다.
- 모든 Object 타입(Integer, String, ArrayList ...)은 heap 영역에 생성된다.
- 몇개의 스레드가 존재하건 `Heap 영역은 단 하나`다.
- Heap 영역에 있는 오브젝트들을 가리키는 레퍼런스 변수가 stack에 올라간다.



##### 예제)

```java
public class Main{
	public static void main(String[] args){
		int port = 4000;
		String host = "localhost";
	}
}
```

이번에는 스택과 힙 영역을 동시에 확인해보자.

`int port = 4000;` 은 Stack에 할당 될 것이다.

![1563695359521](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1563695359521.png)



String은 Object를 상속받아 구현되었으므로, (Object 타입은 최상위 부모클래스다.) 다형성에 의해 Object 타입으로 레퍼런스가 가능하다. String은 Heap 영역에 할당되고 stack에 host라는 이름으로 생성된 변수는 heap에 있는 "localhost"라는 스트링을 레퍼런스 하게 된다. 

![1563695474783](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1563695474783.png)





##### 예제)

```
public class Main {
	public static void main(String[] args){
		List<String> listArgument = new ArrayList<>();
		listArgument.add("yaboong");
		listArgument.add("github");
		
		print(listArgument);
	}

	private static void print(List<String> listParam){
		String value = listParam.get(0);
		listParam.add("io");
		System.out.println(value);
	}
}
```



`List<String> listArgument = new ArrayList<>();`

`new` 키워드가 실행이되면, 생성하려는 오브젝트를 저장할 수 있는 충분한 공간이 Heap 영역에 있는지를 먼저 찾는다.

그 다음, 비어있는 List 를 참조하는 listArgument 라는 로컬변수를 스택에 할당한다.

![1563695771003](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1563695771003.png)



다음 list에 add로 값을 넣어준다. 

위 코드를 풀어서 쓰면, `listArgument.add(new String("yaboong"));`과 같다.

new 키워드에 의해 heap 영역에 충분한 공간이 있는지 확인한 후 "yaboong" 이라는 문자열을 할당한다. 이때 새롭게 생성된 문자열은 stack에 할당되지 않는다.

List 내부의 인덱스에 의해 하나씩 add()된 데이터에 대한 레퍼런스 값을 갖게 된다.

![1563695919777](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1563695919777.png)



하나더 add를 하게되면,

![1563695941038](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1563695941038.png)



다음, 함수 호출이 일어난다.

`print(listArgument)`

이때 lsitArgument 참조변수를 인자로 넘겨준다. 함수 호출시 원시타입의 경우 같이 넘겨주는 인자가 가지고 있는 값이 그대로 파라미터에 복사된다.



`Print(List<String> listParam)`메소드에서는 listParam 이라는 참조변수로 인자를 받게 되어있다. 그래서 print 함수가 호출될 때 메모리의 변화는 아래와 같다.

![1563696071840](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1563696071840.png)

listParam 이라는 참조변수가 새롭게 stack에 할당되어 기존 List를 참조하게 된다.

기존 listArgument는 scope 밖에 있게되므로 접근할 수 없는 영역이된다.



다음 print()함수 내부에서 List에 있는 데이터에 접근하여 값을 value에 저장한다.

value 변수는 stack영역에 추가되고, value는 다시 listParam을 통해 List의 0번째 요소에 접근해 참조값을 가지게 된다. 그리고 또 데이터를 추가하고, 출력해 print()함수의 역할이 마무리 된다.



함수 종료되기 직전의 stack & heap 영역 상태는 아래와 같다.

![1563696305441](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1563696305441.png)



함수가 종료되면 다시 listParam은 pop되고 listArgument에 재할당이 일어난다.



![1563696407934](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1563696407934.png)

`Object 타입의 데이터(= heap 영역의 데이터)`는 함수 내부에서 파라미터로 copied value를 받아서 변경하더도 `함수 호출이 종료된 시점에 변경내역이 반영`된다.



##### 예제)

```java
public class Main {
	public static void main(String[] args){
		Integer a = 10;
		System.out.println("Before : " + a);
		changeInteger(a);
		System.out.println("After : " + a);
	}
	
	public static void changeInteger(Integer param){
		param += 10;
	}
}
```

위 코드를 보면 Before 는 10 After 는 20이 나올것 같다면, 오산이다.



이해를 돕기위해 위의 Integer를 String으로 바꾸고 두 문자를 '+' 한다고 생각해보자.

변수 a의 문자를 change 함수에 전달하면서 함수가 호출된다.

change 함수의 parma은 `a + param` 이 된 새로운 String 오브젝트가 할당되는 작업이다.

기존에 a 가 "hello" 문자를 가지고 있었지만 + param을 함으로써 새롭게 생성된 String 오브젝트를 레퍼런스 하도록 만든다.

하지만 change 함수가 종료되면서 새롭게 생성된 오브젝트를 레퍼런스하는 param이라는 변수는 스택에서 pop 된다. 

따라서 실행 결과는 Befor, After 모두 10이다.



자바에서 Wrapper class에 해당하는 Integer, Character, Byte, Boolean, Long, Double, Float, Short는 모두 Immutable(불변)이다. 

그래서 heap 에 있는 같은 오브젝트를 레퍼런스 하고 있는 경우라도, 새로운 연산이 적용되는 순간 `새로운 오브젝트가 Heap에 할당`된다.



Object인 Integer 클래스를 들어가보면

`private final int value;` 를 볼 수 있는데, 생성되는 순간에만 초기화되고 변경 불가능한 값이 된다. 나머지 Wrapper class도 마찬가지다.





##### 참고

https://yaboong.github.io/java/2018/05/26/java-memory-management/

