## JPA 엔티티의 상태 변화



> 백기선님의 Spring Data JPA 강좌를 참고하여 정리한 내용입니다.



#### 엔티티의 상태

엔티티의 상태에는 크게 4가지가 있다.

- Transient
- Persistent
- Detached
- Removed



![image-20200303082849119](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20200303082849119.png)

> 각각의 상태 변화에 대해 알아보자



- Transient : JPA가 모르는 상태 (단순 객체 생성)

  ```
  Account account = new Account();
  account.setUsername("seokhun");
  account.setTitle("JPA");
  ```



- Persistent : JPA가 관리중인 상태 (1차 캐시, Dirty Checking, Write Behind)

  ```
  Session session = entityManager.unwrap(Session.class);
  session.save(account);
  ```

  save를 하는 시점부터는 JPA가 account 객체를 인지하고 관리를 하기 시작한는 것이다.

  여기서 중요한 점은, `save가 됐다고 해서 바로 insert문이 날라가는 것이 아니라는 점`이다.

  저장 할 시점이 되어야 insert문이 날라간다.



​		Persistent 상태에서는 `1차 캐시, Dirty Checking, Write Behind` 등등을 지원해준다.



####  **1차 캐시**

위의 코드에서 save를 한 후에 아래 코드를 실행하면 어떻게 될까?

  ```
  Account n1tjrgns = session.load(Account.class, account.getId());
  System.out.println(n1tjrgns.getUserName());
  ```

  ​	

해당 코드에 대한 select 쿼리는 발생하지 않고, 객체에 저장되어있는 userName값을 나타내준다.

그리고 나서 insert 쿼리는 Transaction이 끝난 후 1번만 발생한다.



영속성 컨텍스트는 내부에 캐시를 가지고 있는데 이를 `1차 캐시`라고하며, 영속 상태의 엔티티는 모두 이곳에 저장된다.

그래서 1차 캐시에서 먼저 엔티티를 찾고 없으면 그 때 DB에서 조회를 한다.



> hibernate가 관리하는 객체의 장점. 크으...



#### 	**Dirty Checking, Write Behind**

```
  Session session = entityManager.unwrap(Session.class);
  session.save(account);

  Account n1tjrgns = session.load(Account.class, account.getId());
  n1tjrgns.setUsername("n1tjrgns");
  System.out.println(n1tjrgns.getUserName());
```



> 위 코드는 어떻게 동작할까?



신기하게도 insert와 update 쿼리가 발생하게된다.



> 업데이트는 하라고 하지 않았는데?



#### 이게 바로 `Dirty Checking`

엔티티를 영속성 컨텍스트에 보관할 때, 최초 상태를 복사해서 저장하는데 이것을 `스냅샷` 이라고 한다.

그리고 `flush 시점`에 스냅샷과 엔티티를 비교해 `객체의 변경사항을 감지`해서 변경된 상태가 변경 되었으니 update를 해야 함을 스스로 알고 쿼리를 발생 시킨다. 

약간 특이한 점은 update 대상이 아닌 칼럼이 있더라도 엔티티의 `모든 필드`를 업데이트 한다.



만약에, update 할 필드가 많거나 저장되는 내용이 너무 크면 수정된 데이터만 사용해 `동적 Update Sql`을 사용해야한다. (= **DynamicUpdate**)



#### Write Behind(트랜잭션 쓰기 지연)

`객체의 상태 변화를 DB에 최대한 늦게 최적화 된 시점에 저장`



```
Account account = new Account();
account.setUsername("seokhun");

Session session = entityManager.unwrap(Session.class);
session.save(account);
  
Account n1tjrgns = session.load(Account.class, account.getId());
n1tjrgns.setUsername("n1tjrgns1");
n1tjrgns.setUsername("n1tjrgns2");
n1tjrgns.setUsername("n1tjrgns3");
n1tjrgns.setUsername("seokhun");
System.out.println(n1tjrgns.getUserName());
```

맨 처음에 넣었던 seokhun과 맨 마지막에 저장 될 userName이 결국 seokhun으로 같은 것을 확인 하고 결국 update를 할 필요가 없음을 알고 update를 실행시키지 않는다.



트랜잭션이 커밋되기 직전까지 DB에 엔티티를 저장하지 않고 내부 쿼리 저장소에 INSERT SQL을 모아둔다.

그 후 `트랜잭션이 끝나는 시점(= transaction commit())`이 되면 `영속성 컨텍스트들을 flush()` 해준다. 이 때 실제 DB에 insert 쿼리가 날라가는 것이다.



Persistent 상태에 있는 객체들의 성능적인 면에서의 장점이다..!



- Detached : JPA가 더이상 관리하지 않는 상태

  쉽게 말해 Transaction 영역을 벗어나는 순간 JPA가 더 이상 관리하지 않는 상태가 된다.



- Removed : JPA가 관리하긴 하지만 삭제하기로 한 상태

  delete() 메소드 처럼 이제 삭제하기로 한 상태를 나타낸다.

  

  remove 하는 순간 영속성 컨텍스트에서 제거된다. 하지만 엔티티 저장처럼 쿼리 저장소에 모아두었다가 flush를 호출하는 순간 삭제 쿼리가 날라간다.

  `영속성 컨텍스트의 변경 내용을 데이터베이스에 동기화 하는 것!`

  



#### flush() 방법

1. em.flush() 호출
2. 트랜잭션 commit 시 자동 호출
3. JPQL 쿼리 실행 시 자동 호출



나머지 세세한 부분들은 따로 추가 정리를 해야겠다.

JPA 너무 재미있따!!









