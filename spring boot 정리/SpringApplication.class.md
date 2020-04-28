## Inflearn

- 기존의 어플리케이션 클래스

```
public static void main(String[] args) {

        SpringApplication.run(InflearnApplication.class, args);
    }
```

스프링의 다양한 커스텀 기능을 쓰는데 제한적이다.



그래서 아래와 같이 변경!

```
public static void main(String[] args) {

        SpringApplication app = new SpringApplication(InflearnApplication.class);
        app.run(args);
    }

```



빌더 패턴으로도 작성 가능

```
public static void main(String[] args) {

        new SpringApplicationBuilder()
        				.sources(SpringinitApplication.class)
        				.run(args);
    }
```





#### Application Event와 Listener

이벤트가 ApplicationContext가 만들어 졌느냐 아니냐를 기점으로 이벤트가 호출되냐 안되냐가 나뉜다.

만들어진 다음에 발생하는 이벤트들의 Listener가 Bean이면 알아서 호출된다.

하지만 그렇지 않은 경우 수동으로 등록시켜줘야한다.

```
public class SampleListener implements ApplicationListener<ApplicationStartingEvent> {

    @Override
    public void onApplicationEvent(ApplicationStartingEvent applicationStartingEvent) {
        System.out.println("========================");
        System.out.println("Application ist starting");
        System.out.println("========================");
    }
}
```

하지만 그 전에 발생한 이벤트`ApplicationStartingEvent` 같은 경우에는 Bean으로 등록해도 동작하지 않는다.

- 직접 등록해야한다.



```
public static void main(String[] args) {
        SpringApplication app = new SpringApplication(InflearnApplication.class);
        app.addListeners(new SampleListener());
        app.run(args);
    }
```



어떤 빈의 새엇자가 1개고 그 생성자의 파라미터가 빈일경우 스프링이 생성자를 자동 주입해줌



Application이 실행된 후 추가 작업이 필요한 경우

- implements ApplicationRunner 사용





- Vm Option에 -Ddebug로 작성시 어플리케이션 debug 레벨까지 출력

