## Spring cloud OAuth2 인증서버 구현하기 - (1)





```
Consider defining a bean of type 'org.springframework.http.codec.ServerCodecConfigurer' in your configuration.
```



```
/*configurations {
            runtime.exclude module: 'spring-boot-starter-web'
        }*/
```

이거 하랬는데 이건 안됨



```
@Bean
    public ServerCodecConfigurer serverCodecConfigurer() {
        ServerCodecConfigurer serverCodecConfigurer = ServerCodecConfigurer.create();
        configureHttpMessageCodecs(serverCodecConfigurer);
        return serverCodecConfigurer;
    }

    protected void configureHttpMessageCodecs(ServerCodecConfigurer configurer) {
    }
```

이거 추가했더니 됨

https://www.codota.com/code/java/methods/org.springframework.http.codec.ServerCodecConfigurer/create





```
spring boot java.lang.IllegalStateException: Cannot configure enpdoints
```

오류

```
@EnableAuthorizationServer 
```

어노테이션을 main으로 이동





----

```
Description:

Parameter 0 of constructor in com.msa.init.DataInitializer required a bean of type 'com.msa.user.User' that could not be found.


Action:

Consider defining a bean of type 'com.msa.user.User' in your configuration.

```

user 클래스의 @Entity 가 존재하는데 DataInitializer클래스에서 user 의존성 주입을 못받아서 임의로 component붙여줌

