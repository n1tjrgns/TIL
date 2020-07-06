## @Transactional 전파 레벨(propagation)



#### @Transactional 사용시 주의사항

- @Transactional을 클래스 또는 메소드 레벨에 명시하면 해당 메소드 호출시 지정된 트랜잭션이 작동하게 된다.

  단, 해당 클래스의 Bean을 다른 클래스의 Bean에서 호출할 때만 `@Transactional`을 인지하고 작동하게된다.



#### Propagation.REQUIRED

```
@Transactional(propagation = Propagation.REQUIRED)
public void doSomething() {
	...
}
```

- `default 값`이기 때문에 생략가능

  부모 트랜잭션 내에서 실행하며, 부모 트랜잭션이 없을 경우 새로운 트랜잭션 생성

  해당 메소드를 호출한 곳에서 별도의 트랜잭션이 설정되어 있지 않다면 트랜잭션을 새로 시작한다.(새로운 연결을 생성하고 실행한다.)

  만약, 호출한 곳에서 이미 트랜잭션이 설정되어 있다면 기존의 트랜잭션 내에서 로직을 실행한다.(동일한 연결 안에서 실행됨)

  예외가 발생하면 롤백이 되고 호출한 곳에도 롤백이 전파된다.

  

#### Propagation.REQUIRES_NEW

```
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void doSomeThing() {...}
```

- Propagation.REQUIRES_NEW로 설정되어있을 때에는 매번 새로운 트랜잭션을 시작한다. (새로운 연결을 생성하고 실행)

  만약, 호출한 곳에서 이미 트랜잭션이 설정되어 있다면(기존 연결이 있다면) 기존의 트랜잭션은 메소드가 종료할 때까지 잠시 대기 상태로 두고 자신의 트랜잭션을 실행한다.(부모 트랜잭션 상관X)

  새로운 트랜잭션 안에서 예외가 발생해도 호출한 곳에는 롤백이 전파되지 않는다.

  즉, 2개의 트랜잭션이 완전 독립적으로 동작한다.



#### Propagation.NESTED

```
@Transactional(propagation = Propagation.NESTED)
public void doSomeThing() {...}
```

- 해당 메소드가 부모 트랜잭션에서 진행될 경우 별개로 커밋되거나 롤백될 수 있다.

  둘러싼 부모 트랜잭션이 없을 경우 Propagation.REQUIRED와 동일하게 작동한다.

  `차이점`은 `SAVEPOINT`를 지정한 시점까지 부분 롤백이 가능하다.

  `유의해야 할 점`은 DB가 SAVEPOINT 기능을 지원해야 사용이 가능(Oracle)하다.

  또한 이미 진행중인 트랜잭션이 있다면 `중첩 트랜잭션`을 시작한다. 



#### Propagation.MANDATORY

```
@Transactional(propagation = Propagation.MANDATORY)
public void doSomeThing() {...}
```

- 부모 트랜잭션 내에서 실행되며, 부모 트랜잭션이 없을 경우 Exception이 발생한다.



#### Propagation.SUPPORT

```
@Transactional(propagation = Propagation.SUPPORT)
public void doSomeThing() {...}
```

- 부모 트랜잭션이 존재하면 부모 트랜잭션으로 동작하고, 없을경우 non-transactional 하게 동작한다.



#### Propagation.NOT_SUPPORT

```
@Transactional(propagation = Propagation.SUPPORT)
public void doSomeThing() {...}
```

- non-transactional 로 실행되며 부모 트랜잭션이 존재하면 일시 정지한다.



#### Propagation.NEVER

```
@Transactional(propagation = Propagation.NEVER)
public void doSomeThing() {...}
```

- non-transactional 로 실행되며 부모 트랜잭션이 존재하면 Exception이 발생한다.







##### 참고

 https://jsonobject.tistory.com/467 

 https://goddaehee.tistory.com/167 

 https://taetaetae.github.io/2016/10/08/20161008/ 

 [https://happyer16.tistory.com/entry/%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98-%EC%A0%84%ED%8C%8C-%EC%86%8D%EC%84%B1-propagation-%EB%A1%A4%EB%B0%B1-%EC%98%88%EC%99%B8](https://happyer16.tistory.com/entry/트랜잭션-전파-속성-propagation-롤백-예외) 

 https://goddaehee.tistory.com/167 

 https://suhwan.dev/2020/01/16/spring-transaction-common-mistakes/ 