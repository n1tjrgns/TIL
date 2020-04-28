## ENUM 클래스 작성방법

예를들어 상품 주문에 대한 상태값을 관리할 때, 여태의 나였으면

```
if(해당경우){
	status = "상태값";
}
```

으로 해당 상태를 하드코딩의 형태로 바꿔주었다.



하지만 이제 ENUM 클래스를 활용하는 방법을 배웠으므로 앞으로는 이렇게 작성하도록 하자.



```
private String status; 
```

위처럼 하드코딩의 형태였다면 해당 변수 타입은 String 형태이다.



```
//등록됨, 해제됨, 대기중의 상태값을 가지는 enum 클래스를 작성해보자.
@Enumerated(EnumType.STRING)
private UserStatus status;  // REGISTERED / UNREGISTERED
```

Enum 값을 가지는 필드에 @Enumerated 어노테이션을 붙여준다.

이 어노테이션은 인자로 EnumType.STRING || EnumType.ORDINAL 가질 수 있다.

- ORDINAL은 enum 순서 값을 DB에 저장
- STRING은 enum 이름을 DB에 저장



status 상태값을 관리할 ENUM 클래스를 생성한다.

- UserStatus
- ENUM 클래스에 값을 받아오기만 하면 되므로 @Getter 메소드만 사용하면된다.

```
@Getter
@AllArgsConstructor
public enum UserStatus {

    REGISTERED(0,"등록상태","사용자 등록상태"),
    UNREGISTERED(1,"해지","사용자 해지상태");

    private Integer Id;

    private String title;

    private String description;
}

```



이렇게만 작성하면 ENUM 클래스 생성이 끝난다.

> 쉽쥬?



이제 실제 insert를 어떻게 해야하는지 확인해보자.

아래 코드에서 status를 관련된 부분을 제외하고는 일부러 삭제를 했다.

```
	@Test
    public void create(){
     
        UserStatus status = UserStatus.REGISTERED;
        

        User user = new User();
        user.setStatus(status);
        
        User newUser = userRepository.save(user);

        Assert.assertNotNull(newUser);

        //빌더 패턴을 사용해 일일이 각각의 상황에 발성하는 생성자를 생성 할 필요가 없다.
        User u = User.builder()
                .status(status)
                .build();

    }
```

UserStatus에서 내가 등록한 Enum 클래스의 상태값을 받아와 user클래스에 저장해준다.

ENUM 클래스를 사용함으로써 오타도 방지할 수 있고, 코드도 간결해진다.

![image-20191207161941108](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20191207161941108.png)