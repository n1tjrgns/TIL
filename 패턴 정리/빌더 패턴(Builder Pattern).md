# 빌더 패턴(Builder Pattern)

빌더 패턴은 객체를 생성할 때 흔하게 사용하는 패턴이다.



.build()라는 코드를 봤다면, 빌더 패턴을 사용했다고 할 수 있다.



나 역시 이 코드를 보고 이게 뭐지?? 하고 찾아보게 되었다.

-----

##### Effective Java의 빌더 패턴

- 객체 생성을 깔끔하고 유연하게 하기 위한 기법



##### 점층적 생성자 패턴

만드는 방법

1. 필수 인자를 받는 필수 생성자를 하나 만든다.

2. 1개의 선택적 인자를 받는 생성자를 추가한다.

3. 2개의 선택적 인자를 받는 생성자를 추가한다.

4. ...반복

5. 모든 선택적 인자를 다 받는 생성자를 추가한다.



```java
//점층적 생성자 패턴 코드 예제 : 회원가입
public class Memeber {
    private final String name; //필수 인자
    private final String location; //선택적 인자
    private final String hobby; //선택적 인자
    
    //필수 생성자
    public Member(String name){
        this(name, "출신지역 비공개", "취미 비공개");
    }
    
    //1개의 선택적 인자를 받는 생성자
    public Member(String name, String location){
        this(name, location, "취미 비공개");
    }
    
    //모든 선택적 인자를 다 받는 생성자
    public Member(String name, String location, String hobby){
        this.name = name;
        this.location = location;
        this.hobby = hobby;
    }
}
```



**장점**

```java
new Member("홍길동", "출신지역 비공개", "취미 비공개");
```

와 같은 호출이 빈번하게 일어난다면, new Member("홍길동")으로 대체할 수 있다.



**단점**

- 다른 생성자를 호출하는 생성자가 많아서, 인자가 추가되는 일이 발생하면 코드를 수정하기 어렵다.
- 코드 가독성이 떨어진다.
  - 특히 인자 수가 많을 때 호출 코드만 봐서는 의미를 알기 어렵다.

```java
//호출 코드로만 각 인자의 의미를 알기 어렵다.
Drink cocaCola = new Drink(24, 6, 100, 45);
Drink pepsiCola = new Dringk(10, 7);
Drink fanta = new Drink(5);
```





##### 자바빈 패턴

그래서 이에 대한 대안으로 자바빈 패턴(JavaBeans Pattern)이 있다.

이 패턴은 setter 메소드를 이용해 생성 코드를 읽기 좋게 만든다.

```java
Drink cocaCola = new Drink();
cocaCola.setCalories(24);
cocaCola.setSugar(6);
cocaCola.setSalt(100);
cocaCola.setServings(45);
```

##### 장점

- 이로써 각 인자의 의미를 파악하기 쉬워졌다.
- 복잡하게 여러 개의 생성자를 만들지 않아도 된다.



##### 단점

- 객체 일관성(consistency)이 깨진다.
  - 1회의 호출로 객체 생성이 끝나지 않았다.
  - 즉, 한번에 생성하지 않고 생성한 객체에 값을 계속 덧붙이고 있다.
- setter 메소드가 있어서 변경 불가능(immutable) 클래스를 만들 수가 없다.
  - 스레드 안정성을 확보하려면 점층적 생성자 패턴보다 많은 일을 해야 한다.







##### 빌더패턴(Effective Java 스타일)

```java
//Effective Java의 Builder Pattern
public class Drink{
	private final  int servings;
    private final  int calories;
    private final  int sugar;
    private final  int salt;
    
    
	public static class Builder {
        //필수 인자
        private final int servings;
        
        //선택적 인자는 기본값으로 초기화
        private int calories = 0;
        private int sugar = 0;
        private int salt = 0;
        
        public Builder(int servings){
            this.servings = servings
        }
        
        public Builder calories(int val){
            calories = val;
            return this; //이렇게 하면 .으로 체인을 이어갈 수 있다.
        }
        
        public Builder sugar(int val){
            sugar = val;
            return this; 
        }
        
        public Builder salt(int val){
            salt = val;
            return this; 
        }
        
        public Drink build(){
            return new Drink(this);
        }
	}
    
    private Drink(Builder builder){
        servings = builder.servings;
        calories = builder.calories;
        sugar = builder.sugar;
        salt = builder.salt;
    }
}
```

```java
public class PersonInfo{

  private String name;         //필수적으로 받야할 정보

  private int age;                //선택적으로 받아도 되는 정보

  private int phonNumber;   //선택적으로 받아도 되는 정보

  private PersonInfo(){

  }

  public static Builder{

    private String name;

    private int age;

    private int phonNumber;

   

   public Builder(String name){

      this.name = name;

    }

   public Builder setAge(int age){

 this.age = age;

   }

   public Builder setPhonNumber(int phonNumber){

 this.phonNumber = phonNumber;

   }

   public PersonInfo build(){

   PersonInfo personInfo = new PersonInfo( );

   personInfo.name = name;

   personInfo.age = age;

   personInfo.phonNumber = phonNumber;

   return personInfo;

 }

}

//이러한 빌더 패턴은 아래와 같이 사용한다.

PersonInfo personInfo = new Builder("Mommoo").setAge(12).setPhonNumber(119).build( );

```

위와 같이 하면 아래와 같은 방법으로 객체를 생성할 수 있다.

2가지 방법이 있다. 

첫번째

```java
Drink.Builder builder = new Drink(2); //필수 인자에 값 입력
builder.calories(100);
builder.sugar(35);
builder.salt(14);
Drink cocaCola = Builder.build();
```

두번째 방식이 내가 자주 봤던 빌더 패턴이었다.

```java
각 줄마다 builder를 타이핑하지 않아도 돼서 편리하다.
Drink cocaCola = new Drink
	.Builder(2)		//필수 인자 값 입력
	.calories(100)
	.sugar(35)
	.salt(14)
	.build();		//build()가 객체를 생성해서 돌려준다.
```



**장점**

- 각 인자가 어떤 의미인지 알기 쉽다.
- setter 메소드가 없으므로 변경 불가능 객체를 만들 수 있다.
- 한번에 객체를 생성해서 일관성이 깨지지 않는다.
- build() 함수가 잘못된 값이 입력되었는지 검증하게 할 수 있다.







##### Lombok @Builder

롬복의 @Builder 어노테이션을 사용하면 쉽게 빌더패턴을 사용할 수 있다.

```java
@Builder
public class Drink{
	private final  int servings;
    private final  int calories;
    private final  int sugar;
    private final  int salt;
}
```





##### 객체 생성을 유연하게

- 빌더 패턴을 사용함으로써 하나의 빌더 객체로 여러 객체를 만드는 것이 가능하다.

```java
public interface Builder<T> {
    public T build();
}
```

위 같은 인터페이스를 만들어놓고, 빌더 클래스가 implements하게 해도 된다.  













https://johngrib.github.io/wiki/builder-pattern/