## [Spring boot] AutoConfiguration

Spring boot의 AutoConfiguration은 `@Conditional`에 기반한다.



스프링 어플리케이션을 개발 할 때, 조건적으로 beans 을 등록 할 필요가 있다.

예를 들면 local, dev, prod에 따라 db 설정이 다른 경우 yml을 분리하여 profiles에 해당하는 설정 파일을 읽도록 수정하면된다.

그런데 각각의 환경에 따라 변경점이 필요하다면 `configuration`을 변경 할 수 있어야 한다.

spring 3.1에서 `Profiles` 개념이 도입되어 application을 실행시킬 때 원하는 profiles만 활성화 시킬 수 있고, 활성화된 profiles에 연관된 beans만 등록된다.

spring 4에서는 `@Conditional`이 도입되어 Spring beans를 조건에 따라 등록할 수 있게 되었다.

----



#### AutoConfiguration

`@EnableAutoCounfiguration` 어노테이션을 통해서 스프링 부트는 `AutoConfiguration`이 설정된다.

`@SpringBootApplication` 어노테이션의 내부를 보면 `@EnableAutoConfiguration`이 포함되어 있는 것을 볼 수 있다.

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
	...
}	
```



또한 `@ComponentScan`이 있어 classpath에서 components들을 스캔하고 Condition에 매칭되는 beans을 등록해 ApplicationContext의 자동 구성이 가능하다.



**참고**

 https://velog.io/@bk_log/Spring-boot-AutoConfiguration 

 http://dveamer.github.io/backend/SpringBootAutoConfiguration.html 

 https://engkimbs.tistory.com/753 

