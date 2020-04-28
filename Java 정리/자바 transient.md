## 자바 transient

#### transient 란?

- transient는 Serialize 하는 과정에서 제외하고 싶은 경우 선언하는 키워드이다.



#### 사용하는 이유

- 패스워드와 같은 보안정보를 직렬화(Serialize)  과정에서 제외하고 싶은 경우에 적용
- 다양한 이유로든 데이터를 전송하고 싶지 않을 때 선언



**예제**

```java
class Member implements Serializable {
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

public static void main(String[] args throws IOException, ClassNotFoundException){
	Member member = new Member("코린이", "corin2@tistory.com",27); //Model 객체
    String serialData = serializeTest(member); //직렬화
    deSerializeTest(serialData); //역직렬화
}

```

위와 같은 간단한 직렬화 예제가 있다고 했을 때, 결과값은 직렬화가 잘 된것을 확인할 수 있습니다.

이 때

```java
class Member implements Serializable {
    private transient String name;
    private String email;
    private int age;

```

name에 transient keyword를 추가한다면??



역직렬화 결과값을 보면 `name='null'` field는 유지되지만 null값이 대입되는 것을 확인할 수 있다.



#### 주의해야할 점

- 적용할 대상의 Data에 대한 이해 필요.
  - 실제로 필요가 없는 Data인지 고려.
  - Data를 제외시켰을 때 서비스 장애에 이상이 없는지 고려.





**참고**

<https://nesoy.github.io/articles/2018-06/Java-transient>

