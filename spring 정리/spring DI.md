# Spring 의 DI



업무를 시작하기 위해 소스코드 분석을 하기 위해 소스를 받았지만 어디서부터 봐야하는지를 모르겠다. 스프링을 잘 모르기 때문에 그런 것 같아 스프링 공부에 대한 필요성을 느꼈다.

또한 스프링을 사용하지만 정작 스프링을 왜 사용하는지, 어떤 점이 좋은지에 대해 제대로 알고자 정리를 시작한다.



##### DI 란? (Dependency Injection) - 의존성 주입

- 객체간의 의존성을 자신이 아닌 외부에서 주입



```
public class HelloApp{
    public static void main(String[] args){
        MessageBeanEng bean = new MessageBeanEng();
        bean.sayHello("spring");
    }
}
```

```
public class MessageBeanEng{
    public void sayHello(String name){
        sysout("Hello, "+name);
    }
}
```



기능이 추가될 때마다 매번 코드를 변경하고, 다시 컴파일하는 것은 비용이 많이 들기 때문에  가급적이면 코드의 변화가 적어지도록 프로그램을 작성하는게 유지보수 측면에서 유용하다.



위의 코드를 한글로 출력하는 코드로 바꿔보겠다.

```
public class HelloApp{
    public static void main(String[] args){
        MessageBeanKor bean = new MessageBeanEng(); 
        	//MessageBeanEng => MessageBeanKor
        bean.sayHello("spring");
    }
}
```

```
public class MessageBeanKor{
    public void sayHello(String name){
        sysout("안녕, "+name);
    }
}
```



main의 MessageBeanEng를 MessageBeanKor로 바꿔줬다.

이 경우를 가리켜 HelloApp이 MessageBeanEng(Kor)에 의존성을 가지고 있다고 한다.

HelloApp의 기능을 바꾸려면 MessageBean도 함께 바꿔줘야 하기 때문이다.



이런 의존성을 해결하기 위해 객체의 인스턴스를 외부에서 생성받을 필요가 있다.



의존성 주입 방법에는 3가지 방법이 있다.

1. 생성자를 이용한 의존성 주입
2. setter메소드를 이용한 의존성 주입
3. 초기화 인터페이스를 이용한 의존성 주입



위에서 작성한 코드를 조금 더 개선해보자.

인터페이스를 사용해 소스 변경을 줄였다.



```
public interface MessageBean{
    public void sayHello(String name)
}
```

```
public class MessageBeanKor implements MessageBean{
    public void sayHello(String name){
        sysout("하이, "+name)
    }
}
```

```
public class MessageBeanEng implements MessageBean{
    public void sayHello(String name){
        sysout("hi, "+name)
    }
}
```

```
public class HelloApp{
    public static void main(String[] args){
        MessageBean bean = new MessageBeanKor();
        bean.sayHello("spring");
    }
}
```



이렇게 작성한 경우 HelloApp class에서 new 이하를 kor, eng로 바꿈에 따라 결과가 다르게 나타나게 된다.

하지만 역시 소스변경을 약간은 해야한다.



마지막으로 외부에서 MessageBean에 주입한다.



**beans.xml**

```
<?xml version="1.0" encoding="UTF-8"?>
<beans
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd">
		
		
		<bean id="messageBean" class="패키지명.MessageBeanKor"/>
</beans>
```



HelloApp

```
public class HelloApp{
    public static void main(String[] args){
        BeanFactory factory = new XmlBeanFactory(new 							 								FileSystemResource("resource/beans.xml"))
    
    	MessageBean bean = factory.getBean("messageBean", MessageBean.class);
    	bean.sayHello("spring")
    }
}
```

결과는 하이, spring 과 같이 출력된다.



영문으로 출력하고 싶다면 

```
//<bean id="messageBean" class="패키지명.MessageBeanKor"/>
<bean id="messageBean" class="패키지명.MessageBeanEng"/>
```

와 같이 바꿔주면 hi, spring이 출력될 것이다.



소스코드의 변경 없이 환경설정으로 프로그램을 제어할 수 있다.

이것이 가능한 이유는 객체의 생명주기를 스프링 컨테이너가 관리하는 IoC개념이 있기 때문이다.



- 스프링에서 의존성 주입은 객체의 실체를 외부 환경설정(spring config xml, Bean 객체 등)에서 컨트롤 할 수 있도록 하는 것이다.
- 이로인해 모듈 간 결합도를 낮춰 유연한 변경을 가능하도록 한다.



https://jhleed.tistory.com/61



-----



