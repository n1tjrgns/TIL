## SecurityConfig 설정하기

#### UserDetailsService 인터페이스

- DB에서 유저 정보를 가져오는 역할
- 해당 인터페이스 메소드에서 DB의 유저 정보를 가져와서 AuthenticationProvider 인터페이스로 유저 정보를 리턴하면 , 그 곳에서 사용자가 입력한 정보와 DB에 있는 유저 정보를 비교.
- loadUserByUsername() 메소드 - DB에서 유저 정보를 불러오는 중요한 메소드 





```java
@Autowired
	UserDetailsService userDetailsService;
	
	@Autowired
	public void configure(AuthenticationManagerBuilder auth)throws Exception{
		auth.userDetailsService(userDetailsService);
	}
```

`AuthenticationManagerBuilder`



-----



[https://postitforhooney.tistory.com/entry/SpringSecurity-%EC%B4%88%EB%B3%B4%EC%9E%90%EA%B0%80-%EC%9D%B4%ED%95%B4%ED%95%98%EB%8A%94-Spring-Security-%ED%8D%BC%EC%98%B4](https://postitforhooney.tistory.com/entry/SpringSecurity-초보자가-이해하는-Spring-Security-퍼옴)

```java
 @Override
    protected void configure(HttpSecurity http) throws Exception{
    	...
    }
```

위 메소드는 각각의 URL Path에 대해 선택적으로 보안을 적용하기 위한 configure(HttpSecurity) 오버라이딩이다.



`http.csrf().disable()` :

- 웹 취약점인 csrf 공격을 막기 위한 설정
- disable을 하지 않으면 해당 작업을 수행하기 위한 파라미터가 없다고 에러가 발생
  `HTTP Status 403 - Expected CSRF token not found. Has your session expired?`



`http.formLogin().loginPage("/login");`

- 로그인 페이지 지정



`http.authorizeRequest()`

- authorizeRequest를 호출하고, 다음에 반환되는 객체로 호출되는 메소드들은 요청 보안 수준의 세부적인 설정을 나타낸다.



`http.authorizeRequest().antMathcers("경로").permitAll()`

- 해당 경로에 무조건 접근 허용
- (ex) 회원가입처럼 인증이 없어도 사용할 수 있어야 하는 부분



`http.anyRequest().authenticated();`

- 어떠한 요청이 오던 인증이 필요하다.



##### 로그아웃 설정

```
http
                .logout()
                //logout요청 url
                .logoutRequestMatcher(new AntPathRequestMatcher("/logout")) 
                //로그아웃 성공시 이동할 url
                .logoutSuccessUrl("/login");
```





##### 인터페이스 UserDetailsService 

- 스프링 시큐리티에서 제공

- 사용자 특정 데이터를 로드하는 핵심 인터페이스

  

userDetailsService 인터페이스를 상속받은 클래스는 사용자 이름을 기반으로 사용자를 찾는다.

```
@Override
public UserDetails loadUserByUsername(String username) UsernameNotFoundException {

}
```

