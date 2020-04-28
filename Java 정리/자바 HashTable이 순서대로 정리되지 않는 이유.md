## 자바 HashTable이 순서대로 정리되지 않는 이유

HashTable 클래스를 찾아보다가 문득 정리하게되었다.

HashMap/HashTable은 우선 Hashing의 원리로 데이터 검색속도를 높일 수 있는 자료구조다.



#### HashTable과 HashMap의 차이점

- Hashtable은 synchronized로 메소드들이 묶여있어, thread safe 하다.
- HashMap은 null value를 입력할 수 있어서 속도가 좀 더 빠르다.



하지만, hashing을 이용하는 기본적인 구조와 데이터 처리 메커니즘은 동일하다.



new Hashtable(); 로 객체를 생성하게되면, 기본값으로 객체를 저장할 수 있는 `11개의 배열공간`이 생성된다.

객체에 데이터를 넣기 위해 put("banana", "fruit") 을 넣으면 key값에 대한 hashCode()를 얻어온다.



key String에 대한 hashCode() 값이 나오면, 배열공간의 갯수로 나머지 연산을 하여 배열상에 저장할 공간을 확정해 key와 value를 저장한다. (`key 값이 동일하면, 항상 동일한 공간에 저장된다.`)

그래서 조회시 hashCode()를 사용해 데이터가 저장된 공간을 바로 확인할 수 있다.



ArrayList의 경우 add를 하면 0번 배열부터 차례대로 하나씩 채워가지만, Hashtable은 key 값의 hash 값을 이용해 저장될 위치를 지정해 다르게 들어간다.

그렇기 때문에 Hashtable,Map 의 데이터를 iterator 로 출력시 입력한 순서가 보장되지 않는다.

Hashtable은 key 값으로 데이터 저장위치를 바로 확인할 수 있기 때문에, 조회 시 높은 성능을 보여준다.



만약에 key, value를 Hashtable에 저장할 때, 새로 저장하려는 공간에 이미 다른 값이 있는경우

현재 저장된 값에 LinkedList로 새로운 값을 연결한다.



저장위치(index)를 찾아 LinkedList의 길이만큼 조회하여 원하는 값을 찾는 코드

```java
 public synchronized V get(Object key) {
	Entry tab[] = table;
	int hash = key.hashCode();
	int index = (hash & 0x7FFFFFFF) % tab.length;
	for (Entry<K,V> e = tab[index] ; e != null ; e = e.next) {
	    if ((e.hash == hash) && e.key.equals(key)) {
		return e.value;
	    }
	}
	return null;
}
```





좁은 공간에 데이터를 저장하다보면, 계속해서 충돌이 발생하고 연결된 LinkedList가 늘어나면서 조회 성능은 떨어질 수 밖에 없다. 

그래서 어느정도 데이터가 차게되면 자동으로 배열공간(buckets)을 늘리는 작업을 하게된다.



Hashtable의 put() 소스를 보면 rehash는 현재 저장된 데이터를 변경된 크기의 배열공간으로 모두 재배치 하는 것으로 데이터가 많이 들어있다면 rehash 비용이 상당할 것이다.

```java
public synchronized V put(K key, V value) {
	// Make sure the value is not null
	if (value == null) {
	    throw new NullPointerException();
	}

	// Makes sure the key is not already in the hashtable.
	Entry tab[] = table;
	int hash = key.hashCode();
	int index = (hash & 0x7FFFFFFF) % tab.length;
	for (Entry<K,V> e = tab[index] ; e != null ; e = e.next) {
	    if ((e.hash == hash) && e.key.equals(key)) {
		V old = e.value;
		e.value = value;
		return old;
	    }
	}

	modCount++;
	if (count >= threshold) {
	    // Rehash the table if the threshold is exceeded
	    rehash();

            tab = table;
            index = (hash & 0x7FFFFFFF) % tab.length;
	}

	// Creates the new entry.
	Entry<K,V> e = tab[index];
	tab[index] = new Entry<K,V>(hash, key, value, e);
	count++;
	return null;
    }
```

count는 Hashtable에 현재 저장되어 있는 key/value 의 갯수이고, threshold는 그에 대한 한계값인데, 

이 값의 기본값은 전체 배열수(11) * 0.75 로 75%가 차면 rehash()를 호출하여 배열공간을 두배로 늘리는 작업을 하게된다.



#### Hash 자료구조의 default capacity size

HashSet-16 
HashMap-16 
HashTable-11 
HashSet-16







<https://m.blog.naver.com/PostView.nhn?blogId=dg110>