## SOP와 CORS

> 본 내용은 백기선님의 스프링 부트 개념과 활용 강의를 정리한 내용입니다.



SOP (Single-Origin Policy)

CORS (Cross-Origin Resource Sharing)



Origin

- URI 스키마 (http, https)
- hostname (whiteship.me, localhost)
- 포트(8080, 18080)



스프링 부트에서는 `@CrossOrigin` 어노테이션 하나로 CORS가 지원된다.



#### 실습

간단한 웹 하나를 띄운다.

별도의 포트 설정을 하지 않았으므로 8080포트로 뜨게된다.

```
@SpringBootApplication
public class InflearnApplication {


    @GetMapping("/hello")
    public String getHello() {
        return "seokhun hello";
    }
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(InflearnApplication.class);
        app.run(args);
    }

}
```



새로운 프로젝트를 생성해 18080포트를 사용하는 웹 페이지를 하나 띄운다.

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1>CORS client</h1>
<script src="/webjars/jquery/3.4.1/dist/jquery.min.js"></script>
<script>
    $(function () {
        $.ajax("http://localhost:8080/hello")
            .done(function (msg) {
                console.log(msg);
            })
            .fail(function () {
                console.log("fail");
            });
    })
</script>
</body>
</html>
```

ajax를 사용해 8080포트로 띄워진 컨트롤러를 호출해본다.

![CORS fail](C:\Users\Lenovo\Desktop\CORS fail.JPG)

실패



다시 원래 프로젝트로 돌아와 해당 메소드에 어노테이션을 추가해준다.

`@CrossOrigin(origins = "http://localhost:18080")`



그 후 다시 ajax 호출 페이지를 새로고침 해보자.

![Cors success](C:\Users\Lenovo\Desktop\Cors success.JPG)

성공





하지만 매번 이 어노테이션을 붙여야 한다면 번거로운 작업이 될것이다.

설정파일로 분리하자.

config 패키지를 만들어 CORS 설정을 해줄 클래스를 작성한다.

```
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry){
        registry.addMapping("/**")
                .allowedOrigins("http://localhost:18080");
    }
}
```

이제 아까 작성해준 `@CrossOrigin` 어노테이션을 삭제해도 정상동작한다.