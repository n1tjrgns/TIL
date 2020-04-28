## [Effective Java] 28. 배열보다는 리스트를 사용해라



#### 배열과 제네릭 타입의 차이

배열은 `공변(covariant)`이다. 

예를들어 sub가 super의 하위 타입이라면 배열 sub[]는 배열 super[]의 하위 타입이 된다고 할 수 있다. 즉, 공변이란 함께 변한다는 뜻이다.



반대로 제네릭은 `불공변(invariant)`이다.

List<sub>는 List<Super> 의 하위 타입도 아니고 상위 타입도 아니다.



예제를 확인해보자.

```
Object[] objectArray = new Long[1];
//ArrayStoreException 발생 => Long 타입에 String을 넣을 수 없음.
objectArray[0] = "루피";


//컴파일시점 에러
List<Object> objectList = new ArrayList<Long>();
objectList.add("루피");
```



제네릭과 다르게 배열은 실체화(reify)된다.

런타임에 자신이 담기로 한 원소의 타입을 인지하도록 확인한다. 그 때문에 `ArrayStoreException`이 발생했다.



하지만 제네릭은 타입 정보가 런타임 시점에 소거(erasure) 된다. 

원소 타입을 컴파일 시점에만 검사하기 때문에 런타임 시점에는 알 수 없다. 타입 정보가 소거된 `Raw type`의 경우 제네릭이 지원되기 전의 레거시 코드와 제네릭 타입을 함께 사용할 수 있도록 해준다.



그래서 배열은 제네릭 타입, 매개변수화 타입, 타입 매개변수로 사용할 수 없다.

```
//사용불가한 배열의 형태
new List<E>[]; //제네릭 타입
new List<String>[]; //매개변수화 타입
new E[]; //타입 매개변수
```



> 왜 제네릭 배열을 생성 못하게 했을까?

- 타입 안정성이 보장되지 않는다.



제네릭 배열의 생성을 허용한다면, 컴파일러가 자동으로 생성한 형변환 코드에서 런타임 시점의 `ClassCastException` 이 발생할 수 있다.

런타임 시점의 형변환 예외 발생을 막기위한 제네릭의 취지에 맞지 않는다.



예제)

```
List<String>[] stringLists = new List<String>[1]; //1
List<Integer> intList = List.of(42);			//2
Object[] objects = stringLists;					//3
object[0] = intList;							//4
Strings s = stringLists[0].get(0);				//5
```

1번처럼 제네릭 배열이 생성 되어있다고 가정해보자.



2번은 원소가 하나인 List를 생성한다.(List.of 메소드는 JDK 9부터 사용 가능하다)



3번은 1번에서 생성된다고 가정한 제네릭 배열을 object[]에 할당한다.

이 때 배열은 공변(Covariant)이므로 아무 문제가 없다.



4번은 2번에서 생성한 List<Integer>의 인스턴스를 Object 배열의 첫 원소로 저장한다.

제네릭은 런타임 시점에서 타입이 사라지므로, List<Integer>는 List가 되고 List<Integer>[]는 List[]가 된다.

따라서 ArrayStoreException이 발생하지 않는다.



하지만, 5번은 다르다. List<String> 의 인스턴스만 담겠다고 선언한 stringLists 배열에 다른 타입의 인스턴스가 담겨있는데 첫 원소를 꺼내려고 한다. 그리고 이를 String 변수에 담으려고 한다.

하지만 이 원소는 Integer 타입이므로 런타임에 `ClassCastException`이 발생한다.



`따라서 이러한 일을 사전에 예방하고자 제네릭에서는 배열이 생성할 수 없도록 1번 과정에서 컴파일 오류가 발생되는 것이다.`



#### 실체화 불가 타입

`E, List<E>, List<String>` 같은 타입을 실체화 불가 타입(non-reifiable type)이라고 한다.

제네릭 소거로 인해 실체화되지 않기 때문에 런타임 시점에 컴파일타임보다 타입 정보를 적게 가지는 타입을 말한다.

소개로 인해 매개변수화 타입 가운데 실체화 할 수 있는 타입은 `List<?>와 Map<?,?>` 같은 비한정적 와일드카드 타입뿐이다.



#### 배열로 형변환시 오류가 발생할 때

배열로 형변환할 때 제네릭 배열 생성 오류가 발생해 경고가 뜨는 경우 대부분은 배열인 E[] 대신에 컬렉션인 List<E>를 사용하면 해결된다.

예제)

```
public class Chooser {
	private final Object[] choiceArray;
	
	public Chooser(Collection choices){
		this.choiceArray = choices.toArray();
	}
	
	
	//이 메소드를 사용하는 곳에서는 매번 형변환이 필요하다
	//형변환 오류의 가능성이 있다.
	public Object choose() {
		Random rnd = ThreadLocalRandom.current();
		return choiceArray[rnd.nextInt(choiceArray.length)];
	}
}
```



위 코드를 제네릭을 사용해 변경해보자.

```
public class Chooser<T> {
    private final T[] choiceArray;

    public Chooser(Collection<T> choices) {
        // 오류 발생 incompatible types: java.lang.Object[] cannot be converted to T[]
        this.choiceArray = choices.toArray();
    }

    // choose 메소드는 동일
}
```



위 코드는 생성자에서 incompatible types 에러가 발생한다.

```
//Object 배열을 T 배열로 형변환하면 된다.
this.choiceArray = (T[]) choices.toArray();
```



컴파일 오류는 사라졌지만 Unchecked Cast 경고가 발생한다.

타입 매개변수인 T가 어떤 타입인지 알 수 없으니 형변환이 런타임에도 안전한지 보장할 수 없다는 메세지이다.

위에서 말했듯이 제네리은 런타임에는 타입 정보가 소거되므로 무슨 타입인지 알 수 없다.

Unchecked Cast와 같은 비검사 형변환 경고를 제거하려면 배열 대신 리스트를 사용하면 된다.



```
class Chooser<T> {
	private final List<T> choiceList;
	
	public Chooser(Collection<T> choices){
		this.choiceList = new ArrayList<>(choices);
	}
	
	public T choose() {
		Random rnd = ThreadLocalRandom.current();
		return choiceList.get(rnd.nextInt(choiceList.size()));
	}
}
```



##### 요약

배열은 공변이고 실체화되지만, 제네릭은 불공변이고 타입 정보가 소거된다.

따라서 배열은 런타임에는 안전하지만, 컴파일타임에는 안전하지 않다.

반면에 제네릭은 컴파일타임에 안전하다.



##### 참고

https://madplay.github.io/post/prefer-lists-to-arrays