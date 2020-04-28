# 자바 equals(), hashCode()



자바에서 각각의 객체가 동일한지 확인하는 방법에는

'==', 'equals', 'hashCode'가 있다.



##### == 연산자

==연산자는 피연산자가 primitive type(int, double, boolean ,,,) 일 때는 값이 같은지 비교하고, 그 이외의 객체나, reference type일 때는 가리키는 주소가 같은지 검사한다.



여기까지는 다 알고 있는 사실일 것이다. 



하지만 String의 경우에는 살짝 다르다.

이전 정리 [valueOf](<https://n1tjrgns.tistory.com/181>) 를 참고하면 new 생성자로 생성한 객체는 각각의 메모리에 String을 만들기 때문에, 주소값도 서로 다르게 된다.

그래서 이 경우 String 비교를 '==' 연산자로 하게되면 false를 리턴한다.





##### equals()

equals는 내용이 같은지를 검사하는 메소드다.

default로 primitive type은 내용이 같은지 검사하고, reference type은 객체의 주소가 같은지 검사한다.

(Object 클래스의 메소드이기 때문에 모든 객체는 equals()메소드를 사용할 수 있다.)

그래서 new 생성자로 객체를 생성했을 때, 주소가 다르지만 내용이 같아 true를 리턴한다.



하지만 String 클래스는 역시 다르다.

위의 new String으로 객체 생성시, String 클래스 내부적으로 equals메소드를 오버라이드하기 때문에 true가 나왔지만, 개발자들이 생성한 클래스들에 대해서까지 equals가 판단하기는 어렵다.





##### 예제)

```java
public class Person{
    private String name;
    private int age;
    
    public String getName(){
        return name;
    }
    
    public void setName(String name){
        this.name = name;
    }
    
    public int getAge(){
        return age;
    }
    
    public void setAge(int age){
        this.age= age;
    }
}
```

Person클래스를 만들고

```java
Person p1 = new Person("aa","27");
Person p2 = new Person("aa","27");
p1.equals(p2);
```

위의 equasl 연산을 수행하면 어떻게 될까?

자체적으로 클래스를 만들었기 때문에 자바에서는 정확하게 알지를 못한다.



그래서 스프링에서는 이에대해 어노테이션을 제공해준다.

@EqualsAndHashCode



하지만 자바에서는 불행하게도 어노테이션이 없기 때문에

내가 작성한 의도에 맞게 equals()메소드를 오버라이딩 해줘야 한다.



```java
@Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Person)) return false;
        Person person = (Person) o;
        return age == person.age &&
                name.equals(person.name);
    }
```



IDE에 보면 자체적으로 equals와 hashCode를 생성해주는 기능이 있다.

그렇다는건 equals만 재정의 해줘서는 안된다는 소리다.

equals를 재정의 해주면 hashCode도 재정의 해줘야한다.

hashCode를 재정의하지 않으면 부작용이 있다.

##### hashCode

```java
@Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
```

hashCode()는 메모리에서 가진 hash 주소 값을 반환해준다.

hash 함수를 쓰는 Collection 객체 때문에 equals와 함께 사용해준다.





equals만 재정의 해주면 두 객체가 같다고 했지만, hash를 사용하는 Collection(HashSet,HashMap)  등등에 넣을 때는 문제가 발생한다.

```java
		Set<Person> set = new HashSet<>();
        Person p1 = new Person("bb",27);
        Person p2 = new Person("bb",27);
        System.out.println(p1.hashCode());
        System.out.println(p2.hashCode());
        set.add(p1);
        set.add(p2);
        System.out.println(set.size());
```

위 처럼 Collection 함수에 같은 값으로 생성한 객체를 넣어보자.

하지만 hashCode의 출력결과가 다르게 나오고, set은 위의 생성된 객체를 각각 다른 객체로 인식해 size 또한 2로 출력이 되게된다.

set의 장점은 중복제거를 수행하지 못하게 된 것이다.



즉, eqauls로 같은 객체라면 hashCode 또한 같은 값이어야 한다.

또한 같은 파라미터를 사용해야한다.

위에서 설명했듯이 파라미터 타입을 Object에서 다른 타입으로 바꿀 경우 오버로딩으로 인식해 기존의 equals 메소드가 남아있게 된다.

