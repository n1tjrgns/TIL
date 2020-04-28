## JPA CRUD 완전정복 (with @Test)



우선 CRUD를 작성하기 전에 Entity 클래스와 , Repository 인터페이스는 작성이 되어있다는 가정하에 진행한다.



#### Repository 인터페이스

```
public interface 이름 extends JpaRepository <엔티티 클래스, PK 유형>
```

- 스프링 부트에서는 Repositoy 인터페이스에서 JpaRepository를 상속 받게되면 Entity 클래스의 기본적인 `CRUD`가 가능하다.



>  Test 코드로 CRUD를 확인해보자



#### Create

```
	@Autowired
    private UserRepository userRepository;

    @Test
    public void create(){
        
        User user = new User();
        user.setAccount("TestUser01");
        user.setEmail("TestUser01@gmail.com");
        user.setPhoneNumber("010-1111-1111");
        user.setCreatedAt(LocalDateTime.now());
        user.setCreatedBy("admin");

        User newUser = userRepository.save(user);
        System.out.println("newUser : "+ newUser );
    }
```

- MVC 패턴과 큰 차이점은 없어보인다.
- 다만, 한 가지 체감하는 것은 MVC 패턴이었다면 주로 마이바티스를 많이 사용하기 때문에, userRepository에서 insert 구문을 또 작성했었어야 했을것이다. 
  - 그런데 save 안에 Entity를 담아주는 것으로 끝났다.  -> `시간절약`





#### Read

```
	@Test
    public void read(){
        //제네릭 타입으로 받게되어있다.
       Optional<User> user = userRepository.findById(1L);

       //Optional 객체가 감싸고 있는 값이 존재할 경우에만 실행 로직을 함수형 인자로 넘김
       //user.ifPresent(System.out::println);
        user.ifPresent(selectUser->{
            System.out.println("user : " + selectUser);
            System.out.println("user : " + selectUser.getEmail());
        });
    }
```



Read메소드를 보면 처음보는 메소드가 등장한다. `ifPresent()`



#### ifPresent() 메소드

- 이 메소드는 특정 결과를 반환하는 대신 Optional 객체가 감싸고 있는 값이 존재할 경우에만, 실행될 로직을 함수형 인자로 넘길 수 있다.
- findById 함수를 통해 select 하고자 하는 키값이 반드시 존재해야만 아래 로직이 실행되기 때문에 테스트 코드로 유효성검사를 할 수 있다.

```
Assert.assertTrue(user.isPresent());
```

user의 Id값이 존재하지 않으면 테스트를 통과하지 못한다.



#### Update

```
	@Test
    public void update(){
        Optional<User> user = userRepository.findById(1L);

        user.ifPresent(selectUser->{
           selectUser.setAccount("pppppp");
           selectUser.setUpdatedAt(LocalDateTime.now());
           selectUser.setUpdatedBy("update method()");
        });
    }
```



#### Delete

```
	@Test
    public void delete(){
        //마찬가지로 select를 먼저하고
        Optional<User> user = userRepository.findById(3L);

        //데이터가 조회가 되어야 삭제를 할 수 있기 때문에 반드시 true여야 한다.
        Assert.assertTrue(user.isPresent());

        user.ifPresent(selectUser->{
            userRepository.delete(selectUser);
        });

        //삭제가 제대로 이뤄졌는지 확인
        Optional<User> deleteUser = userRepository.findById(3L);

        //데이터가 삭제됐으면 isPresent는 false가 나와야 한다.
        Assert.assertFalse(deleteUser.isPresent());

        /*if(deleteUser.isPresent()){
            System.out.println("데이터 존재 : "+deleteUser.get());
        }else{
            System.out.println("데이터 없음");
        }*/
    }
```

소스 마지막 부분에 데이터의 존재를 확인하려면, if문을 사용해 현재 값이 존재할 때와 그렇지 않을 때를 나눠야한다.

하지만 테스트코드 1줄이면 if문을 대체할 수 있다.



#### 전체 소스코드 첨부

Git 주소 :  [Jpa를 사용한 CRUD](https://github.com/n1tjrgns/JpaCRUD)



