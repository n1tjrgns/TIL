## @Profile



config 디렉토리 안에 설정 파일이 2개 존재 한다고 가정해보자.



```
@Profile("prod")
@Configuration
public class ProdConfiguration {
	
	@Bean
	public String hello(){
		return "hello prod";
	}
}


###############################################
@Profile("local")
@Configuration
public class LocalConfiguration {
	
	@Bean
	public String hello(){
		return "hello local";
	}
}
```

 

그 후 hello 메소드를 의존성 주입을 통해 호출을 해보자.

```
public class SampleListener implements ApplicationListener<ApplicationStartingEvent> {

    @Autowired
    private String hello;
    
    @Override
    public void onApplicationEvent(ApplicationStartingEvent applicationStartingEvent) {
        System.out.println("========================");
        System.out.println(hello);
        System.out.println("========================");
    }
}
```



그럼 에러가 발생 할 것이다.

hello 메소드를 가진 클래스가 2개가 존재해 어느 메소드의 hello를 불러야 하는지 알 수 없다.



properties 파일에 설정을 추가해주자.

```
spring.profiles.active=prod
```

위와 같이 설정을 하면 @Profile 어노테이션 안의 값을 확인 하고 해당 클래스의 메소드를 읽어오게 된다.



위처럼 운영, 로컬 테스트 환경을 바꿔줄 수 있다.



-----



또는 properties 파일을 prod, local과 같이 2개로 만들어줘도 된다.

