## [Java] Optional은 왜 사용할까? (Feat. NullPointerException)

----

자바 프로그래밍을 하면서 NPE를 겪어보지 않은 사람은 없을것이다.

null 처리를 하기 위해 우리는 많은 노력을 한다.



- 모든 객체를 null 체크, null 일 경우에 대한 처리

```java
if(person != null){
		//null이 아닐 때만 코드 수행
}

#################
  
if(person == null){
  throw new PersonNotExistException();
}
```

이 경우 객체의 개수 만큼 if절이 늘어나게 되어 가독성에 좋지 않다.



#### Optional (java.util.Optional<T>)

자바8 에서는  Optional이라는 새로운 클래스를 제공한다.

예를 들어, 위의 person 객체가 있다고 가정했을 때 **`Optional<Person>`** 처럼 객체를 감싼다.

값이 있을 경우 객체를 감싸지만, 없는 경우 **`Optional.empty`** 메소드로 Optional을 반환한다.



#### null 참조와 Optional.empty 는 다른가??

null의 경우 NullPointerException을 반환한다. 반면에, Optional.empty의 경우 Optional 객체이기 때문에 이를 다양하게 활용할 수 있다.

또한 객체를 Optional로 감싸게되면, 해당 객체에 null 값이 할당될 수 있음을 명시적으로 나타내준다.

> null 이 있을 수 있음을 명시적으로 보여줄 수 있구나
>
> 필수값이 아님을 알 수 있겠네.



#### Optional과 스트림

Optional도 마찬가지로 스트림과 함께 효율적으로 사용할 수 있다.

예를 들어보자.

```java
// 사람
public class Person{
	private Optional<Car> car; //사람이 차가 있을 수도 있고, 없을 수도 있다.
	public Optional<Car> getCar(){
		return car;
	}
}
```

```java
// 차량
public class Car{
	private Optional<Insurance> insurance; //보험에 가입했을 수도 있고, 아닐 수도 있다.
    public Optional<Insurance> getInsurance(){
      return insurance;
    }
}
```

```java
// 보험
public class Insurance{
		private String name; //	보험 회사는 이름이 반드시 존재한다.
		public String getName(){
			return name;
		}
}
```

위와 같이 객체를 정의할 수 있다.

여기서 눈여겨봐야 할 부분은 각각의 객체를 Optional로 감싸주어 해당 객체가 null을 가질 수 있음을 명시적으로 표시한점이다.



```java
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
Optional<String> name = optInsurance.map(Insurance::getName);
```

마찬가지로 스트림의 .map을 사용해서 보험이름을 조회할 수 있다.



또한, flatMap을 사용하여 객체를 조회할 수도 있다.

```java
public String getCarInsuranceName(Person person){
		return person.getCar().getInsurance().getName();
}
```



위 코드를 flatMap을 사용해 바꿔보자.

```java
public String getCarInsuranceName(Optional<Person> person){
	return person.flatMap(Person::getCar)
							 .flatMap(Car::getInsurance)
							 .map(insurance::getName)
							 .orElse("UNKOWN");
}
```

불필요한 if 문 null체크 분기를 없앨 수 있게된다. 또한, 코드가 직관적이다.



#### Optional을 사용하면서 주의해야할 점

1. Optional은 선택형 반환값을 지원하는 용도로만 사용해야 한다.

클래스 필드 형식으로 사용할 것을 가정하지 않았기 때문에, Serializable 인터페이스를 구현하지 않는다. 따라서 도메인 모델에 Optional을 사용한다면 **`직렬화`**시 문제가 생길 수 있다.

그래서 별도로 Optional 메소드를 도메인에 추가하는 방식을 권장한다.

```java
public class Person{
		private Car car;	//직렬화 용도
		public Optional<Car> getCarAsOptional(){ //optional 용도
			return Optional.ofNullable(car)
		}
}
```



2. 기본형 Optional을 사용하면 안된다.

스트림처럼 기본형인 OptionalInt, OptioanlLong, OptionalDouble을 제공한다.

하지만 Optional의 최대 요소 수는 1개이므로 Optional에서는 기본형 특화 클래스로 성능을 개선할 수 없다.