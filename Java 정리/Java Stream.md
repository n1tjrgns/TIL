## Java Stream (BufferStream의 그 Stream 아님)



`Stream`은 자바 8부터 추가되었다.

- 컬렉션, 배열등의 저장 요소를 하나씩 참조하며 함수형 인터페이스(람다식)를 반복적으로 처리할 수 있도록 해주는 기능이다.



1. 기존 방식

```
List<String> names = Arrays.asList("test", "test1","test2");

long count = 0;

for(String name : names){
	if(name.contains("es")){
		count++;
	}
}

System.out.println("Count : " + count); //3
```



2. 람다식을 이용한 방식

```
count = names.stream().filter(name -> name.contains("es")).count();
System.out.println("Count : " + count); //0
```



### 스트림의 구조

1. 스트림 생성
2. 중개 연산
3. 최종 연산

`Collections 객체 집합. 스트림 생성().중개연산().최종연산();`



-----



### 중개 연산

Filter는 조건에 맞는 것을 거른다는 내용이다.

위의 코드는 names의 리스트가 "es"를 포함하고있으면 거른다는 뜻이 되겠다.



#### Map

```
List<String> names = Arrays.asList("test", "test1","test2");

names.parallelStream().map(
					(name) -> return name.concat("o");
					})
					.forEach(name -> System.out.println(name));

//testo, test1o, test2o
```

- 각 문자열 요소 뒤에 "o"를 붙였다. 숫자의 경우 값을 * 2 와 같이 조작이 가능하다.



#### Peek

- Map과 유사하게 각 요소에 어떤 연산을 적용할 때 사용한다.



#### Sorted

- 스트림의 요소들을 정렬해준다.



#### Limit

```
List<Integer> ages = Arrays.asList(1,2,3,4,5,6,7,8,9);
ages.stream().filter(age -> age > 5).limit(3);
//6,7,8
```



#### Distinct

- DB의 distict처럼 스트림 요소의 중복을 제거해준다.



#### Skip

- `.skip(3)` 이면 처음 3개 요소는 제외하고 나머지 요소들로 새로운 stream을 만든다.



#### mapToInt, mapToLong, mapToDouble

- mapTo`XXX`에 해당하는 타입의 스트림으로 바꿔준다.
- "1","2","3" 을 가지는 스트림요소에 mapToInt를 적용하면 int형 1,2,3 으로 변환해준다.



-----



### 최종연산

`count(), min(), max(), sum(), average()`

- 최종 연산이기 때문에 앞서 함수를 적용했던 스트림에 있는 요소들에 대한, count, 최소, 최대, 합계, 평균 값을 얻을 수 있다.



#### reduce

```
List<Integer> ages = new ArrayList<Integer>();

ages.add(1);
ages.add(2);
ages.add(3);

ages.stream().reduce((b,c) -> b+c).get()); 
// 1+2 = 3
// 3+3 = 6 의 과정을 거치게된다.

```



#### forEach

```
List<Integer> ages = new ArrayList<Integer>();

ages.add(1);
ages.add(2);
ages.add(3);

Set<Integer> set = ages.stream().collect(Collectors.toSet());
set.forEach(age -> System.out.println(age));
```

- forEach는 map과 peek의 최종버전이다. 
- 각 요소를 돌면서 처리할 수 있도록 되어있다.



#### collect

- 스트림의 값들을 모아주는 기능을 한다. toMap, toSet, toList로 해당 스트림을 다시 컬렉션으로 바꿔준다.



#### Iterator

```
List<String> names = Arrays.asList("test", "test1","test2");

Iterator<String> it = names.stream.iterator();

while(it.hasNext()){
	System.out.println(it.next());
	//test, test1, test2
}
```



#### noneMatch, anyMatch, allMatch - return boolean

```
List<Integer> ages = new ArrayList<Integer>();

ages.add(1);
ages.add(2);
ages.add(3);

ages.stream().filter(age -> age>1).noneMatch(age -> age>2); //false
```

- noneMatch : 얻은 스트림의 `모든` 요소들이 조건을 만족하지 않는지 판단
- anyMatch : 하나라도 조건을 만족하는지 판단
- allMatch : `모든` 요소들이 조건을 만족하는지 판단



------

### 꾸르팁

- Stream은 재사용이 불가능하다.

```
Stream<String> a = names.stream().filter(x -> x.contains("es"));
count = a.count;

List<String> lists = a.collect(Collectors.toList());

//재사용 불가 에러
//stream has already been operated upon or closed.
```



- 병렬 스트림은 여러 쓰레드가 작업한다.

```
names.parallelStream().filter(x -> x.contains("o")).count();
```

위처럼 parallelStream을 사용하면 병렬 스트림을 만들 수 있다.

여러 쓰레드가 스트림에서 요소를 필터링하고 나온 요소 수를 계산하고 쓰레드끼리 다시 한 번 각자 계산한 count 값을 더해서 리턴해준다.



- 만약, 어플리케이션에서 사용하는 쓰레드가 많거나 스트림의 요소 수가 많지 않다면 오히려 쓰레드를 사용하는데 드는 오버헤드가 더 클 수 있다. 



- 중개 연산은 미리 실행되지 않는다.

```
Stream<String> a = names.stream().filter(x -> x.contains("es")).map(x -> x.concat("o"));

a.forEach(x -> System.out.println(x));
```

위 코드에서 filter와 map은 미리 계산하지 않고 forEach같은 최종연산이 적용될 때 중개 연산이 실행된다.

- 미리 계산하면서 두 번 순회하는 낭비를 방지할 수 있다.





**참고**

https://jeong-pro.tistory.com/165 