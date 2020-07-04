## Equals와 HashCode

----

Equals와 HashCode에 대해 좀 더 자세히 공부 할 필요성을 느껴서 정리한다.



#### Equals

- 두 객체의 `동등성(내용)`을 비교한다.



#### HashCode

- 두 객체의 `동일성(같은 객체)`을 비교한다.



```

public class Human {

    private int age;
    private String name;

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public static void main(String[] args) {
        Human h1 = new Human();
        h1.setAge(28);
        h1.setName("Seokun");

        Human h2 = new Human();
        h2.setAge(28);
        h2.setName("Seokun");

        System.out.println(h1.equals(h2));
    }
}

```

- 결과값은 뭐가 나올까??

  false



> 왜??

- Equals 메소드를 확인해보자

  ```
  public boolean equals(Object obj){
  	return (this == obj);
  }
  ```

  단순 `==` 연산으로 비교하는 것을 볼 수 있다.

  또한 heap 영역에서의 new 객체의 저장 방법도 같이 알아두면 이해하기 쉬울 것이다.



> 동일한지 확인하기 위해서는 어떻게 해야할까?

- equals() 메소드를 @Override 하면 문제를 해결 할 수 있다.

```
    @Override
    public boolean equals(Object obj){

        if(obj == null){
            return false;
        }

        if(this.getClass() != obj.getClass()){
            System.out.println("this :" + this.getClass());
            System.out.println("obj :" + obj.getClass());
            return false;
        }

        if(this == obj){
            System.out.println("Same Object");
            return true;
        }

        Human that = (Human) obj;

        if(this.name == null && that.name != null){
            return false;
        }

        if(this.age == that.age && this.name.equals(that.name)){
            System.out.println("Same value Object");
            return true;
        }
        return false;
    }
```

- 결과는 ?

  Same value Object

  true



----

- equals()가 true인 두 Object를 HashMap에 put 할 때 동인한 Key로 인식하고 싶은 경우

```
				Map<Human, Integer> map = new HashMap<>();
        map.put(h1, 1);
        map.put(h2, 1);
        System.out.println(map.size()); //2
```

> h1과 h2는 같은 객체(equals())가 true 나왔는데 왜 map.size가 2일까?

- Collection(HashMap, HashTable, HashSet, LinkedHashSet 등등) 에서는 key를 결정할 때 `hashcode()` 를 사용하기 때문이다. 



> 그럼 어떻게 key값이 동일하게 인식하게 만들지??

- equals() 를 Override 했듯이 hashcode() 또한 Override 해주자

```
@Override
    public int hashCode(){
        final int prime = 31;
        int hashCode = 1;
        
        hashCode = prime * hashCode + ((name == null) ? 0 : name.hashCode());
        hashCode = prime * hashCode + age;

        return hashCode;
    }
```

- 결과

  Same value Object

  1



----

#### Object 의 hashCode()

- hashCode()로 native call을 하여 Memory에서 가진 해쉬 주소값을 출력한다.
- 특별한 설정을 하지 않을 경우 `System.identityHashcode() 와 동일한 값을 나타낸다.

```
public native int hashCode();
```



#### String의 hashCode()

```
public int hashCode(){
	int h = hash;
	if(h == 0 && value.length > 0){
		char val[] = value;
		
		for(int i=0; i<value.length; i++){
			h = 31 * h + val[i];
		}
		
		hash = h;
	}
	return h;
}
```



#### 왜 31을 곱하지??

- 31은 소수이면서 홀수이다.

- 짝수가 사용되었다면 2로 곱하는 것은 비트를 왼쪽으로 shift하는 것과 같기 때문이다.

- 따라서 곱셈을 shift와 뺄셈의 조합으로 바꾸면 더 좋은 성능을 낼 수 있다

  `(31 * i) -> (i << 5) - i` VM에서 최적화를 자동 실행한다.

- 소수는 1과 자기 자신을 제외한 숫자이기 때문에 Hash 하였을 경우 충돌이 가장 적은 숫자이다.



#### 결론

- equals()를 재정의 한다면 side effect를 줄이기 위해서 hashCode() 또한 재정의 해주자!





**참고**

https://nesoy.github.io/articles/2018-06/Java-equals-hashcode

