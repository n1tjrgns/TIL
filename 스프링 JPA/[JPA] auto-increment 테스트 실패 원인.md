## [JPA] auto-increment 테스트 실패 원인 및 해결

기존에 각각 메소드 단위로 단위 테스트를 마치고 PR을 하여 Merge 되었던 소스가 있었다.



> 근데 이거 전체로 돌리니까 ID 값이 삭제가 안되었는데 ??
>
> - 응???



다음날 바로 전체 테스트 케이스를 돌려보았더니 테스트가 깨졌다.

> 뭐지 merge..? 





#### 원인 분석

![delete1](C:\Users\Lenovo\Desktop\delete1.JPG)

![delete2](C:\Users\Lenovo\Desktop\delete2.JPG)

![delete3](C:\Users\Lenovo\Desktop\delete3.JPG)



@After 어노테이션을 사용해 각 메소드를 테스트 할 때 마다 `usersRepository.deleteAll()` 하도록 설정을 해놓은 상태였다.

delete 쿼리는 매 테스트 마다 잘 수행이 되었지만, 우리의 auto-increment Id님 께서는 1,2,3 순차적으로 증가를 하고계셨다.

> ㅎㅎ



[이동욱님 블로그](https://jojoldu.tistory.com/295)에서도 비슷한 사례로 Entity의 Id 칼럼의 GeneratedValue 전략을 `GenerationType.IDENTITY` 로 지정해 해결 한 사례가 있지만, 나는 이미 이렇게 설정 한 상태였다.



구글링을 통해 알게되었는데

@After에서 `usersRepository.deleteAll()`를 통해 이전 테스트에서 생성된 레코드를 지우더라도, auto-increment에 의해 두 번째 실행되는 메소드에서 생성되는 레코드의 id는 2가 된다.



그래서 내가 단독으로 테스트 했을 때는 다 통과가 되었던것이다!

단 하나의 메소드만 실행하므로 auto-increment에 영향을 받지않는다!



#### 해결 방법

매번 auto-increment값을 초기화 해주고 싶다면 H2 DB에서는 아래와 같이 설정해줘야한다.

```
@After
    public void teardown() {
        usersRepository.deleteAll();
        this.entityManagr
            .createNativeQuery("ALTER TABLE users ALTER COLUMN `id` RESTART 																	WITH 1")
            .executeUpdate();
    }
```



그런데 또 문제가 있다.



이 방법은 `@DataJpaTest` `@Transactional`이 적용되는 서비스 클래스를 테스트 할 때는 적용 가능하지만, 

`@SpringBootTest( webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT )`를 사용하는 환경에서는 사용 할 수 없다.



@SpringBootTest 에서 tearDown 메소드를 실행하면

>  javax.persistence.TransactionRequiredException: Executing an update/delete query 

에러가 발생한다.



Transaction을 적용하기 위해 @Transactional을 붙여주더라도, deleteAll()을 해도 제대로 사라지지 않는 현상을 볼 수 있다..!!



#### 다른 방법

테스트 클래스에 아래 어노테이션을 사용한다.

```
@DirtiesContext(classMode = DirtiesContext.ClassMode.AFTER_EACH_TEST_METHOD)
```

auto-increment 문제는 해결되지만, `테스트 메소드 하나하나 실행 될 때 마다 applicationContext를 새로 만들기 때문에 테스트 실행 속도가 매우 느려지므로` 권장되는 방법은 아니다.



그래서 특정 id의 조회가 아니고서야 id에 대한 테스트는 존재 유무만 확인 하기로 했다.



끝.