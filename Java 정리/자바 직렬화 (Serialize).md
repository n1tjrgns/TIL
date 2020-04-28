## 자바 직렬화 (Serialize)

자바 transient 를 찾아보려 했는데 transient를 알기 위해서는 Serialize에 대한 이해가 필요하다고 한다.

그 이유는 transient는 Serialize 하는 과정에서 제외하고 싶은 경우 선언하는 키워드이기 때문이다.



#### 직렬화(Serialize)

- 자바 시스템 내부에서 사용되는 `Object 또는 Data를 외부의 자바 시스템에서도 사용할 수 있도록 byte 형태로 데이터를 변환하는 기술`.
- JVM의 메모리에 상주(힙 또는 스택) 되어 있는 객체 데이터를 바이트 형태로 변환하는 기술



#### 역직렬화(Deserialize)

- `byte로 변환된 Data를 원래대로 Object나 Data로 변환하는 기술`을 역직렬화(Deserialize)라고 부른다.
- 직렬화된 바이트 형태의 데이터를 객체로 변환해 JVM으로 상주시키는 형태



#### 직렬화 조건

- `java.io.Serializable 인터페이스를 상속받은 객체`는 직렬화 할 수 있는 기본 조건이다.

```java
public class Member implements Serializable {
    private String name;
    private String email;
    private int age;

    public Member(String name, String email, int age) {
        this.name = name;
        this.email = email;
        this.age = age;
    }
    @Override
    public String toString() {
        return String.format("Member{name='%s', email='%s', age='%s'}", name, email, age);
    }
}
```



#### 직렬화 방법

- `java.io.ObjectOutputStream` 을 사용해 직렬화를 진행한다.

```java
public static void main(String[] args){
	Member member = new Member("코린이", "corin2@tistory.com",27);
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
}
```



#### 역직렬화 조건

- 직렬화 대상이 된 객체의 클래스가 클래스 패스에 존재해야 하며, import 되어 있어야 한다.
- `직렬화와 역직렬화를 진행하는 시스템이 서로 다를 수 있다는 점을 고려`해야 한다.
- 자바 직렬화 대상 객체는 `동일한 serialVersionUID`를 가지고 있어야 한다.
  - private static final long serialVersionUID = 1L;



#### 역직렬화 방법

- `java.io.ObjectInputStream`을 사용해 진행한다.

```java
public static void main(String[] args){
	//직렬화 예제에서 생성된 base64 데이터
	String base64Member = "~~~";
	byte[] serializedMember = Base64.getDecoder().decode(base64Member);
	try(ByteArrayInputStream bais = new ByteArrayInputStream(serializedMember)){
		try(ObjectInputStream ois = new ObjectInputStream(bais)){
			//역직렬화된 Member 객체를 읽어온다
			Object objectMember = ois.readObject();
			Member member = (member) objectMember;
			System.out.println(member);
		}
	}
}
```



예외 처리를 왜 저런식으로 했는지 궁금하다면 [[Effective java]try with resources 사용](https://n1tjrgns.tistory.com/183) 링크를 참조하자.





직렬화 방법에는 여러 Format이 존재한다.

- 표 형태의 다량의 데이터는 CSV 형태
- 구조적인 데이터는 XML, JSON 형태



> CSV 파일 (comma-separated values) 
>
> 몇가지 필드를 ' , ' 로 구분한 텍스트 데이터 및 파일이다. 확장자는 .csv 이고 스프레드시트나 데이터베이스 소프트웨어에서 많이 쓰인다.



**CSV**

- 데이터를 표현하는 방법 중 가장 많이 사용하는 방법으로 콤마(,)를 기준으로 데이터를 구분한다.

```java
Member member = new Member("코린이", "corin2@tistory.com",27);
//member 객체를 csv로 변환
String csv = String.format("%s,%s,%d", member.getName(), member.getEmail(), member.getAge());
System.out.println(csv);
```



**JSON**

```java
Member member = new Member("코린이", "corin2@tistory.com",27);
//member 객체를 json로 변환
String json = String.format("{\"name\":\"%s"\",\"email\":\"%s\",\"age\":%d}",member.getName(), member.getEmail(), member.getAge());
System.out.println(json);
```



#### 사용하는 이유

- 복잡한 데이터 구조의 클래스 객체라도 직렬화 기본 조건만 지키면 큰 작업 없이 바로 직렬화가 가능하다.
- 데이터 타입이 자동으로 맞춰지기 때문에 관련 부분을 큰 신경 쓰지 않아도 된다.



#### 사용되는 곳

**서블릿 세션**

- 세션을 서블릿 메모리 위에서 운용하면 직렬화가 필요하지 않지만, 파일로 저장하거나 세션 클러스터링, DB를 저장하는 옵션 등을 선택하면 세션 자체가 직렬화가 되어 저장된다.



**캐시**

- Encache, Redis, Memcached 라이브러리 시스템에 많이 사용된다.



**자바 RMI(Remote Method Invocation)**

- 원격시스템 간의 메시지 교환을 위해서 사용하는 자바에서 지원하는 기술.



#### 자바 직렬화의 단점

**역직렬화시 클래스 구조 변경 문제**

**기존 멤버 클래스**

```java
 public class Member implements Serializable {
      private String name;
      private String email;
      private int age;
    // 생략
  }
```



**직렬화한 Data**

```
rO0ABXNyABp3b293YWhhbi5ibG9nLmV4YW0xLk1lbWJlcgAAAAAAAAABAgAESQADYWdlSQAEYWdlMkwABWVtYWlsdAASTGphdmEvbGFuZy9TdHJpbmc7TAAEbmFtZXEAfgABeHAAAAAZAAAAAHQAFmRlbGl2ZXJ5a2ltQGJhZW1pbi5jb210AAnquYDrsLDrr7w=
```



**멤버 클래스 속성이 추가된 상황**

```java
 public class Member implements Serializable {
      private String name;
      private String email;
      private int age;
     //phone 추가
     private String phone;
    
  }
```



속성이 추가된 상황에서 직렬화한 Data를 역직렬화 한다면??

- java.io.InvalidClassException 발생

- 직렬화하는 시스템과 역직렬화하는 시스템이 다른 경우 발생
- 각 시스템에서 사용하고 있는 모델 버젼 차이 발생시 생기는 문제



> 그렇다면 ??



- 위에서 언급했던 SUID(serialVersionUID)를 정의해야 한다.
- Default는 클래스의 기본 해쉬값을 사용한다.



#### 다른문제

- String -> StringBuilder, int -> long으로 변경해도 역직렬화에서 Exception 발생한다.
- 멤버 변수가 빠지면 Exception 대신 nul값이 들어간다.



#### 직렬화 Data Size 문제

```java
{"name" : "코린이", "email" :"corin2@tistory.com","age":27 }
serializedMember (byte size = 146)
json (byte size = 62)
```

- 위 처럼 간단한 객체의 내용도 크기 차이가 많이 나는것을 볼 수 있다.
- 일반적인 메모리 기반의 Cache에서는 Data를 저장할 수 있는 용량의 한계가 있기 때문에 Json 형태와 같은 경량화된 형태로 직렬화하는 것도 좋은 방법이다.





#### 요약

- 외부 저장소로 저장되는 데이터는 짧은 만료시간의 데이터를 제외하고 자바 직렬화의 사용을 지양한다.
- 역직렬화는 반드시 예외가 생긴다는 점을 고려한다.
- 자주 변경되는 비즈니스적인 데이터는 직렬화를 하지 않는다.
- 긴 만료 시간을 가지는 데이터는 JSON 등 다른 포맷을 사용해 저장한다.



**참고**

<https://nesoy.github.io/articles/2018-04/Java-Serialize>

