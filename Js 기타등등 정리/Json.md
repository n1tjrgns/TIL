# json 파싱

##### json

- 데이터를 더 쉽게 교환하고 저장하기 위해 만들어진 텍스트 기반 데이터 교환 표준.



##### json 데이터 값

1. 숫자
2. 문자열
3. boolean
4. 객체
5. 배열
6. null

이 올 수 있다.



##### 이스케이프 시퀀스(escape sequence)

문자열이 화면에 출력될 때 사용하게 될 특수한 문자를 위해 만들어졌다.

| 이스케이프 시퀀스 | 설명                                  |
| ----------------- | ------------------------------------- |
| \b                | 백스페이스                            |
| \f                | 폼 피드(form feed)                    |
| \n                | 개행                                  |
| \r                | 캐리지 리턴(carriage return)          |
| \t                | 탭(tab)                               |
| \"                | 큰따옴표                              |
| \/                | 슬래시                                |
| \\                | 역슬래시                              |
| \uHHHH            | 16진수 네 자리로 표현된 유니코드 문자 |

```json
{
    "comment":"안녕하세요. \"식빵\" 입니다."
}
```



Json 파서(parser)는 Json 데이터를 모두 읽어들인다.

그렇기 때문에 이스케이프 시퀀스로 표현하지 않는다면, 식빵 앞에서 문자열이 끝났다고 인식하게되어 오류를 발생시킨다.



##### 객체(Object)

Json에서 객체는 데이터 이름과 값(key, value) 한 쌍으로 구성된 프로퍼티의 정렬되지 않은 집합을 의미한다.

Json 객체는 중괄호({})로 둘러싸여 있다.

```

{
    "name" : "식빵",
    "age" : 1,
    "birth" : 2.1
}
```

데이터의 값이 객체라면 객체 안에 객체가 포함되는 계층 구조가 형성된다.

```
{
    "dog":{
        "name":"식빵"
        "owner":{
            "ownerName":"홍길동"
        }
    }
    
}
```



##### JSON 배열

대괄호로 둘러쌓아 표현한다.

배열은 데이터의 값만을 나열한다.

```
{
"dog":[
    "웰시코기",
    "포메라니안",
    "푸들
	]
}
```

배열의 인덱스는 0부터 실행하기 때문에 위에서 웰시코기는 0, 포메라니안은 1의 인덱스를 가지게 된다.



##### 배열과 객체의 차이점

JSON 배열과 객체는 여러 데이터를 묶어놓은 집합이라는 점에서 서로 비슷하다.

하지만 객체는 프로퍼티의 집합이고, 배열은 데이터값의 집합이라는 차이가 있다.





```json
String str =
{
    "data" : {
    "name" : "abc",
    "age" : 14,
    "birth" : 2004
}


JsonParser parser = new JsonParser(); //파싱을 위한 객체 생성

JsonObject object = (JsonObject) parser.parse(json);

System.out.print("name : " + jsonObject.get("name")); => hello!

JsonObject dataObject = (JsonObject) jsonObject.get("data");
sysout("name : "+ dataObject.get("name")); => abc
sysout("age : " + dataObject.get("age")); => 14
sysout("birth : "+ dataObject.get("birth")); => 2004
```





#### JSON 파싱

```java
String str = "[{'NO':1,'NAME':'APPLE','KOR':'사과','PRICE':'1000'},{'NO':2,'NAME':'BANANA','KOR':'바나나','PRICE':'500'},{'NO':3,'NAME':'MELON','KOR':'메론','PRICE':'2000'}]";

JsonParser jsonParser = new JsonParser(); //파싱을 위한 객체 생성

//JsonArray를 선언해 JsonParser로 문자열 파싱
JsonArray jsonArray = (JsonArray)jsonParser.parse(str);


//루프를 돌면서 jsonArray안의 object(중괄호)를 빼내어 값을 추출한다.
//jsonArray의 크기는 Array(대괄호)안의 object(중괄호)수 이다.
for(int i=0; i<jsonArray.size(); i++){
    JsonObject object = (JsonObejct) jsonArray.get(i);
    
    String no = object.get("NO").getAsString();
    String name = object.get("NAME").getAsString();
    String kor = object.get("KOR").getAsString();
    String price = object.get("PRICE").getAsString();
}
```

##### 출력 결과 

1
APPLE
사과
1000
2
BANANA
바나나
500
3
MELON
메론
2000



object.get("태그")로 object안의 key를 찾아 해당하는 value를 리턴해준다.

-----

#### 

##### object의 key가 있는 경우

```java
str = "{'fruit':
[{'NO':1,'NAME':'APPLE','KOR':'사과','PRICE':'1000'},
 {'NO':2,'NAME':'BANANA','KOR':'바나나','PRICE':'500'},
 {'NO':3,'NAME':'MELON','KOR':'메론','PRICE':'2000'}],
 
     'animal':[{'NO':1,'NAME':'cat','KOR':'고양이','age':'3'},
               {'NO':2,'NAME':'dog','KOR':'개','age':'5'},
             {'NO':3,'NAME':'rabbit','KOR':'토끼','age':'2'}]}";
```



key가 있기 때문에 JsonParser를 Object로 먼저 파싱한다.

그 후 array를 object에서 key(fruit)를 get해 해당하는 object를 array로 담는다.

```java
JsonParser parser = new JsonParser();
JsonObject jsonObj = (JsonObject) parser.parse(str);
JsonArray memberArray = (JsonArray) jsonObj.get("fruit");

```



array에서 object를 해당하는 key로 value를 추출한다.

루프가 끝나면 다음 array의 object key(animal)를 넣어 array를 만든 후 루프를돌린다.

```java
//fruit
for(int i=0; i<memberArray.size(); i++){
	JsonObject object = (JsonObject) memberArray.get(i);
	sysout(object.get("NO"))
}

//animal
memberArray = (JsonArray)jsonObj.get("animal");
for(int i=0; i<memberArray.size(); i++){
    JsonObject object = (JsonObject) memberArray.get(i);
    sysout(object.get("NO"));
}


```



-----

##### 직렬화 

- 자바 시스템 내부에서 사용되는 객체 또는 데이터를 외부의 자바 시스템에서도 사용할 수 있도록 바이트 형태로 데이터 변환하는 기술과 바이트로 변환된 데이터를 다시 객체로 변환하는 기술(역직렬화)을 아울러서 이야기한다.



##### 직렬화 조건

자바 기본(primitive) 타입과 **java.io.Serializable** 인터페이스를 상속받는 객체는 직렬화 할 수 있는 기본 조건을 가진다.

```java
//직렬화 할 회원 클래스
public class Member implements Serializable{
    private String name;
    private String email;
    private int age;
    
    public Member(String name, String email, int age){
        this.name = name;
        this.email = email;
        this.age = age;
    }
    
    //Getter 생략
    
    @Override
    public String toString(){
        return String.format("Member{name='%s', email='%s', 									age='%s'}", name, email, age)
    }
}
```



##### 직렬화 방법

직렬화 방법은 java.io.ObjectOutputStream 객체를 이용한다.

`ObjectOutputStream`  : out을 마샬링하기 위한 ObjectOutputStream 객체 생성

`마샬링(marshalling)` : 데이터를 바이트 덩어리로 만들어 스트림에 보낼 수 있는 형태로 바꾸는 변환 작업.

```java
Member member = new Member("홍길동","ghdrlfehd@rlfehd.com",25);
byte[] serializedMember;
try(ByteArrayOutputStream baos = new ByteArrayOutputStream()){
    try(ObjectOutputStream oos = new ObjectOutputStream(baos)){
        oos.writeObject(member);
        //serializedMember -> 직렬화된 member 객체
        serializedMember = baos.toByteArray();
    }
}

//바이트 배열로 생성된 직렬화 데이터를 base64로 변환
System.out.println(Base64.getEncoder().encodeToString(serializedMember));
```

객체를 직렬화 하여 바이트 배열(byte[]) 형태로 변환.



##### 역직렬화 조건

- 직렬화 대상이 된 객체의 클래스가 클래스 패스에 존재해야 하며 import되어 있어야 한다.

  - 직렬화와 역직렬화를 진행하는 시스템이  서로 다를 수 있다.

- 자바 직렬화 대상 객체는 동일한 serialVersionUID를 가지고 있어야 한다.

  ```java
  private static final long serialVersionUID = 1L;
  ```

  Member 클래스에 추가해주자.



##### 역직렬화 예제

```java
//String base64Member = Base64.getEncoder().encodeToString(serializedMember);

byte[] serializedMember = Base64.getDecoder().decode(base64Member);

 try (ByteArrayInputStream bais = new ByteArrayInputStream(serializedMember)) {
        try (ObjectInputStream ois = new ObjectInputStream(bais)) {
            // 역직렬화된 Member 객체를 읽어온다.
            Object objectMember = ois.readObject();
            Member member = (Member) objectMember;
            System.out.println(member);
        }
    }
```



#### 그렇다면 직렬화는 왜 사용하는가??



##### 문자열 형태의 직렬화 방법

직접 데이터를 문자열 형태로 확인 가능한 직렬화 방법. API나 데이터를 변환해 추출할 때 많이  사용된다.

CSV와 JSON

- CSV

  - 데이터를 표현하는 가장 많이 사용되는 방법 중 하나.
  - 콤마(,)를 기준으로 데이터를 구분.

  ```
  홍길동,ghdrlfehd@rlfehd.com
  ```

  - 자바에서 사용법

    ```java
    Member member = new Member("홍길동","ghdrlfehd@rlfehd.com",25);
    
    //member객체를 csv로 변환
    String csv = String.format("%s,%s,%d", member.getName(), 			member.getEmail(), member.getAge());
    sysout(csv);
    
    ```





- JSON

  - 자바스크립트에서 쉽게 사용 가능하고, 다른 데이터 포맷 방식에 비해 오버헤드가 적어 많이 사용

  ```java
  {
      name : "홍길동",
      email : "ghdrlfehd@rlfehd.com",
      age : 25
     
  }
  ```

  - 자바에서 사용법

    ```java
    Member member = new Member("홍길동", "ghdrlfehd@rlfehd.com", 25);
    //member객체를 json으로 변환
    Stirng json = String.format(  "{\"name\":\"%s\",\"email\":\"%s\",\"age\":%d}",
              member.getName(), member.getEmail(), member.getAge());
      System.out.println(json);  
    ```

    GSON 같은 라이브러리를 사용해 변환할 수 있다.





#### 직렬화는 언제 어디서 사용되는가??

JVM의 메모리에서만 상주되어있는 개체 데이터를 그대로 영속화할 때 사용.

시스템이 종료되더라도 없어지지 않는 장점, 영속화된 데이터이기 때문에 네트워크 전송도 가능하다.

필요할 때 직렬화된 객체 데이터를 가져와 역직렬화 하여 객체를 바로 사용할 수 있다.





- 서블릿 세션(Servlet Session)

- 캐시




<http://woowabros.github.io/experience/2017/10/17/java-serialize.html>





##### + 다른예제

JsonObject로 파싱하던 위 코드를 Gson으로 파싱하기 위해선 우선 객체를 담을 클래스를 구현한다.

- @SerializedName은 파싱할 때 사용될 key 이름이다.

```java
class DataJson{
    @SerializedName("name")
    public String name;
    
    @SerializedName("data")
    public Data data;
}
```

클래스를 구현했으면 아래와 같이 사용한다.

```java
DataJson dataJson = new Gson().fromJson(json, DataJson.class);
Sysout("name : "+dataJson.name);
```



이렇게 객체로 데이터를 관리하면, 데이터를 사용할 때마다 key, value로 찾지않아도 쉽게 사용할 수 있다.

하지만 json 규격이 일정해 클래스로 지정할 수 있을 경우에만 사용할 수 있다.