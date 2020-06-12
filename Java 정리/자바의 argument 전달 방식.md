## 자바의 argument 전달 방식



####  call-by-value

- 함수로 인자를 전달할 때 전달될 `변수의 값을 복사`하여 함수의 인자로 전달한다.



#### call-by-reference

- 인자로 사용될 `변수의 레퍼런스`를 함수로 전달한다.



> 자바는 call-by-value일까? call-by-reference일까?

대표적인 `Person`예제를 통해 알아보자.

##### Person Class

```
public class Person {
	private String name;
	public Person(String name){
		this.name = name;
	}
	
	public void setName(String name){
		this.name = name;
	}
	
	@Override
	public String toString(){
		return "name is" + this.name;
	}
}
```



##### Person 객체를 넘겨받아 새로운 Person 객체로 변경하는 부분

```
public class CallByValueTest{
	public static void assignNewPerson(Person p){
		p = new Person("루피");
	}
	
	public static void main(String[] args){
		Person korin2 = new Person("코린이");
		assignNewPerson(korin2);
		System.out.println(korin2);
	}
}
```

- 결과 : name is 코린이



```
public class CallByValuesTest2{
	public static void changePersonName(Person p){
		p.setName("루피");
	}
	
	public static void main(String[] args){
		Person korin2 = new Person("코린이");
		changePersonName(korin2);
		System.out.println(korin2);
	}
}
```

- 결과 : name is 루피



> 매개변수로 넘어오는 변수가 가리키는 인스턴스 메모리 주소를 넘겨주기 때문에 call by value라고 할 수 있다.



assignNewPerson 함수에서 "korin2"라는 변수가 가리키는 "코린이" 인스턴스 주소를 넘기긴 했지만 korin2 변수 자체의 레퍼런스를 넘긴 것은 아니기 때문에 p 변수에 새로운 person을 할당하더라도 korin2 변수에 영향은 없다.

`변수의 레퍼런스`와 `인스턴스의 레퍼런스`를 구별해야한다.



JVM의 String의 new 생성 방식과 "" 생성 방식의 차이점을 같이 알아두면 이해하는데 도움이 될 것 이다.



##### 참고

 https://brunch.co.kr/@kd4/2 