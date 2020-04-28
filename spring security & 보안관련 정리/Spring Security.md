## Spring Security

스프링 기반으로 어플리케이션의 보안을 담당하는 프레임워크. 스프링 시큐리티를 사용하지 않으면, 자체적으로 세션을 체크하고 redirec를 해야한다. spring security는 filter 기반으로 동작하기 때문에 spring MVC와 분리되어 관리 및 동작한다.

security 3.2 부터는 XML로 설정하지 않고도 자바 bean 설정으로 간단하게 설정할 수 있도록 지원해준다.



자주쓰는 용어

- 접근 주체(Principal) : 보호된 대상에 접근하는 유저
- 인증(Authenticate) : 현재 유저가 누구인지 확인( ex : 로그인) - 어플리케이션 작업을 수행할 수 있는 주체임을 증명
- 인가(Authorize) : 현재 유저가 어떤 서비스, 페이지에 접근할 수 있는 권한이 있는지 검사
- 권한 : 인증된 주체가 어플리케이션의 동작을  수행할 수 있도록 허락되있는지 결정
  - 권한 승인이 필요한 부분의 접근은 인증 과정을 통해 접근 주체가 증명 되어야 한다.
  - 권한 부여에는 `웹 요청 권한`, `메소드 호출 및 도메인 인스턴스에 대한 접근 권한` 두 가지가 있다.



**spring-security**는 `세션-쿠키` 방식으로 인증한다.

물론, OAuth2를 활용한 방법 등등 다양한 인증방법이 존재한다.



#### 의존성 추가

**Maven**

```xml
<dependencies>
<!-- ... other dependency elements ... -->
<dependency>
	<groupId>org.springframework.security</groupId>
	<artifactId>spring-security-web</artifactId>
	<version>4.2.2.RELEASE</version>
</dependency>
<dependency>
	<groupId>org.springframework.security</groupId>
	<artifactId>spring-security-config</artifactId>
	<version>4.2.2.RELEASE</version>
</dependency>
</dependencies>
```

**Gradle**

```xml
dependencies {
	compile 'org.springframework.security:spring-security-web:4.2.2.RELEASE'
	compile 'org.springframework.security:spring-security-config:4.2.2.RELEASE'
}
```



#### Java Configuration

요즘의 스프링 시큐리티 레퍼런스는 자바 기반의 설정으로 설명하고 있다.

위에서 언급했듯이 스프링 시큐리티 3.2부터 빈(bean)설정이 가능하기 때문이다.



원래 XML 기반의 설정에서는 web.xml에 `org.springframework.web.filter.DelegatingFilterProxy`라는 springSecurityFilterChain을 등록하는 것으로 시작한다.

하지만 자바 기반의 빈 설정에서는 WebSecurityConfigurerAdapter를 상속받은 클래스에 @EnableWebSecurity 어노테이션을 명시하는 것만으로도 springSecurityFilterChain이 자동으로 포함되어진다.

```java
@EnableWebSecurity
public class WebSecurity extends WebSecurityConfigurerAdapter{

}
```



그리고 포함된 springSecurityFilterChain을 등록하기 위해서는 AbstractSecurityWebApplicationInitializer를 상속받은 WebApplicationInitializer를 만들어두면 된다.

```java
public class SecurityWebApplicationInitializer extends 																				AbstractSecurityWebApplicationInitializer{

}
```



스프링 MVC를 이용해 어플리케이션을 구성하기 때문에 ApplicationInitializer에 WebSecurityConfigurerAdapter를 상속받은 클래스를 getRootConfigClasses() 메소드에 추가하는 것으로 시큐리티 기본 적용이 끝난다.



#### HttpSecurity

기본 적용을 마쳤으면, configure(HttpSecurity http)메소드를 통해서 인증 매커니즘을 구성해야한다.  



```
http
	.a
```





<https://okky.kr/article/382738>