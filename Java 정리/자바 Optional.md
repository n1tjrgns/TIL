## 자바 Optional



#### Optional.of

- value가 `null`인 경우 NullPointerException 발생, 반드시 값이 존재해야 하는 경우 사용

```
public static <T> Optional<T> of(T value);

ex)
Optional<String> opt = Optional.of("result");
```



#### Optional.ofNullable

- value가 `null`인 경우 비어있는 Optional을 반환, 값이 `null`일 수도 있는 경우 사용

```
public static <T> Optional<T> ofNullable(T value);

ex)
Optional<String> opt = Optional.ofNullable(null);
```



#### Optional.empty

- 비어있는 Optional 객체 생성

```
public static<T> optional<T> empty();

ex)
Optional<String> emptyOptional = Optional.empty();
```



#### .map(mapper)

- 입력값을 다른 값으로 변환

```
ex)
Integer test = Optiponal.of("1").map(Integer::valueOf).orElseThrow(NoSuchElementException::new)

//문자열 "1"을 찾아서 Integer로 변환하고 그렇지 않은 경우 Exception을 발생
```



#### .orElse

- null인 경우, 아닌 경우 항상 호출됨

```
private static String wontRunThis() {
	System.out.println("Won't run this"); 
	return "foo"; 
} 

public void optional1() { 
	String o = Optional.of("Hello World!").orElse(wontRunThis()); 					System.out.println("Result : " + o); 
}

```



`result`

```
Won't run this
Result : Hello World!
```



#### .orElseGet

- null인 경우에만 호출됨

```
public void optional2() { 
	String o = Optional.of("Hello World!").orElseGet(() -> wontRunThis()); 			System.out.println("Result : " + o); 
}

```



`result`

```
Result : Hello World!
```



차이점은 `orElse`는 무조건 객체의 null 여부에 상관없이 무조건 실행되기 때문에 Won't run this를 출력하지만, `orElseGet` 은 내부 객체가 null인 경우에만 실행되기 때문에 `Hello World`만 출력 된 것을 볼 수 있다. 