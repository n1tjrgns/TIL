## Hateoas 구현

#### @JsonCreator 

- json을 역직렬화를 하려면 Jackson이 필요한데 생성자를 통해 생성하거나 setter를 통해 생성한다.

- setter 사용시 가변객체가 되기 때문에 불변객체로 만들기 위해 생성자 위에 `@JsonCreator`를 사용해 setter 메소드를 없앤다.



#### @JsonProperty

- Jackson이 어떤 필드에 인자를 넣어야할지 확실하게 명시해준다.



#### RestController 만들기

```
@RestController
public class GreetingHateoasController {

    private static final String template = "Hello Hateoas, %s!";

    @GetMapping("/greetoas")
    public HttpEntity<GreetingHateoas> greetingHateoasHttpEntity(@RequestParam(value = "name", defaultValue = "World") String name){

        GreetingHateoas greetingHateoas = new GreetingHateoas(String.format(template, name));
        greetingHateoas.add(linkTo(methodOn(GreetingHateoasController.class).greetingHateoasHttpEntity(name)).withSelfRel());

        return new ResponseEntity<>(greetingHateoas, HttpStatus.OK);
    }
}
```



컨트롤러를 가리키는 link를 만들고 표현 모델에 붙이는 방법이다.

linkTo()와 methodOn()은 ControllerBuilder의 static 메소드인데 컨트롤러에 가짜 메소드 콜을 만든다.

리턴된 LinkBuilder는 컨트롤러 메소드





> 근데 hateoas를 사용했는데 swagger랑 충돌이 일어난다. 왜지?







**참고**

 https://pjh3749.tistory.com/260 

