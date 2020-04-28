## JPA repository.save is Null (NullPointerException) - feat. @RunWith & @SpringBootTest & @DataJpaTest



JPA로 save 를 작성해보던 도중

```
	@Autowired
    private UserRepository userRepository;

    @Test
    public void create(){
        System.out.println(userRepository);
        User user = new User();
        user.setAccount("TestUser01");
        user.setEmail("TestUser01@gmail.com");
        user.setPhoneNumber("010-1111-1111");
        user.setCreatedAt(LocalDateTime.now());
        user.setCreatedBy("admin");
        System.out.println(user);

        User newUser = userRepository.save(user);
        System.out.println("newUser : "+ newUser );
    }
```

이와 같은 테스트코드를 실행했는데 자꾸 `save` 구문에서 NullPointerException이 발생했다.



> 이상하다.. Repository는 문제가 없는데,,



알고보니 

의존성을 주입한 `userRepository`가 null이었던것,,



>  근데 왜 null 이지?



검색하다 알게되었다

`@RunWith` 어노테이션이 없었던 것.



SpringBootTest를 사용해 테스트를 진행하려 했으니 JUnit이 상속받는 SpringRunner 클래스를 사용하지 않아

Bean이 주입되지 않아 NullPointer 에러가 발생한 것!



#### @RunWith(SpringRunner.class)

- @RunWith 어노테이션은 JUnit 프레임워크가 테스트를 실행할 시 내장된 Runner를 실행한다.
- @SpringBootTest 어노테이션을 사용하려면 JUnit SpringJUnit4ClassRunner 클래스를 상속 받는 `@RunWith(SpringRunner.class)`를 사용해야한다.



#### @SpringBootTest

- @SpringBootTest는 통합 테스트를 제공하는 기본적인 어노테이션이다.
- 여러 단위 테스트를 하나의 통합된 테스트로 수행할 때 사용한다.
  - 단, 어플리케이션의 설정을 모두 로드하기 때문에 어플리케이션의 규모가 클수록 느려진다.



#### @DataJpaTest

- SpringBoot에서 JPA를 테스트 할 수 있도록 제공하는 어노테이션

- 단위 테스트가 끝날 때 마다 자동으로 DB를 롤백시켜준다.
- 인메모리 DB를 사용한다.
- 실제 디비에서 테스트를 하고 싶다면                                                            `@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)` 어노테이션을 추가해줘야한다.



#### 기타 Test 어노테이션

| 어노테이션      | 설명                         | Bean            |
| :-------------- | :--------------------------- | :-------------- |
| @SpringBootTest | 통합 테스트, 전체            | Bean 전체       |
| @WebMvcTest     | 단위 테스트, Mvc 테스트      | MVC 관련된 Bean |
| @DataJpaTest    | 단위 테스트, Jpa 테스트      | JPA 관련 Bean   |
| @RestClientTest | 단위 테스트, Rest API 테스트 | 일부 Bean       |
| @JsonTest       | 단위 테스트, Json 테스트     | 일부 Bean       |

