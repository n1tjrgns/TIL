# Iterator

- 자바의 컬렉션 프레임워크에 저장되어 있는 요소들을 읽어오는 방법의 표준화 방법 중 하나



##### 구성

```java
public interface Iterator{
	boolean hasNext();
    Object next();
    void remove();
}
```



- ##### hasNext()

Iterator가 순방향으로 이동하는데, 가리키는 데이터저장소의 현재 위치에서 이동할 항목이 있는지 체크.

이동할 항목이 있다면 true, 없다면 false 리턴





- ##### next()

Iterator가 자신이 가리키는 데이터저장소에서 현재위치를 순차적으로 하나 증가해서 이동함





hasNext()가 true를 리턴하는 동안 next() 메소드로 실제 이동.



##### 각 반복문 별 사용법

```java

// using iterators for a clloection of String objects:
// using in a for loop
for (Iterator it = options.iterator(); it.hasNext(); ) {
   String name = (String)it.next();
   System.out.println(name);
}

// using in while loop
Iterator name = options.iterator();
    while (name.hasNext() ){
      System.out.println(name.next() );
    }

// using in a for-each loop (syntax available from java 1.5 and above)
    for (Object item : options)
        System.out.println(((String)item));
```





##### Iterator 예제

```java
list에 이미 값이 있다고 가정할 때

Iterator<String> it = list.iterator();

while(it.hasNext()){
    String str = it.next();
    if(str.compareTo("같냐") == 0){
        it.remove();
    }
    
    it = list.iterator();
    while(it.hasNext()){
        sout(it.next());
    }
}
```



##### CompareTo 메소드

A.compareTo(B)

- A와 B가 같으면 0을 반환
- A가 B보다 크면 양수를 반환
- A가 B보다 작으면 음수를 반환

equals(Object) 메소드는 true, false를 돌려주고, compareTo는 정수값을 돌려준다.





<https://promobile.tistory.com/142>