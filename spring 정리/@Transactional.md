# @Transactional

스프링에서 트랜잭션 처리를 지원하는데, 이중 어노테이션 방식으로 @Transactional을 선언해 사용하는 방법이 일반적이고, **선언적 트랜잭션**이라고 부른다.

클래스나 메소드 위에 @Transactional이 추가되면, 이 클래스에 트랜잭션 기능이 적용된 프록시 객체가 생성된다.

이 프록시 객체는 @Transactional이 포함된 메소드가 호출 될 경우, PlatformTransactionManager를 사용해 트랜잭션을 시작하고, 정상 여부에 따라 **commit** 또는 **Rollback**한다.



xml 설정

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource" />
</bean>

<tx:annotation-driven transaction-manager="transactionManager" />
```



##### @Transaction 속성

| 속성          | 설명                                                         | 예시                                             |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------ |
| isolation     | Transaction의 isolation Level. 별도로 정의하지 않으면 DB의 Isolation Level을 따름. | @Transactional(isolation=Isolation.DEFAULT)      |
| propagation   | 트랜잭션 전파규칙을 정의, Default=REQUIRED                   | @Transactional(propagation=Propagation.REQUIRED) |
| readOnly      | 해당 Transaction을 읽기 전용 모드로 처리 (Default = false)   | @Transactional(readOnly = true)                  |
| rollbackFor   | 정의된 Exception에 대해서는 rollback을 수행                  | @Transactional(rollbackFor=Exception.class)      |
| noRollbackFor | 정의된 Exception에 대해서는 rollback을 수행하지 않음.        | @Transactional(noRollbackFor=Exception.class)    |
| timeout       | 지정한 시간 내에 해당 메소드 수행이 완료되지 않은 경우 rollback 수행.          -1일 경우 no timeout(Default = -1) | @Transactional(timeout=10)                       |



##### Isolation Level(격리 레벨)

- 여러 transaction들이 다른 Transaction의 방해로부터 보호되는 정도

| 속성                       | 설명                                                         |
| -------------------------- | ------------------------------------------------------------ |
| ISOLATION_DEFAULT          | 데이터베이스에 의존                                          |
| ISOLATION_READ_UNCOMMITTED | 가장 낮은 격리 레벨. 다른 commit 되지 않은 트랜잭션에 의해 변경된 데이터를 볼 수 있다. |
| ISOLATION_READ_COMMITTED   | 데이터베이스에서 디폴트로 지원하는 격리 레벨.                                 다른 트랜잭션에 의해 commit되지 않은 데이터는 다른 트랜잭션에서 볼 수 없다. 일반적으로 가장 많이 사용. |
| ISOLATION_REPEATABLE_READ  | 다른 트랜잭션이 새로운 데이터를 입력했다면 볼 수 있다.                  트랜잭션이 완료 될때까지 SELECT 문장이 사용하는 모든 데이터에 shared lock이 걸려 다른 사용자는 그 영역에 해당하는 데이터 수정 불가능 |
| ISOLATION_SERIALIZABLE     | 하나의 트랜잭션이 완료된 후에 다른 트랜잭션이 실행하는 것 처럼 지원한다.    동일한 데이터에 대해 동시에 두 개 이상의 트랜잭션이 수행 될 수 없다. |



##### Propagation Behavior (전달 행위)

- mvc-dispatcher-servlet.xml에 아래의 component-scan으로 패키지를 명시해야 한다.

| 속성                      | 설명                                                         |
| ------------------------- | ------------------------------------------------------------ |
| PROPAGATION_REQUIRED      | 기존의 트랜잭션이 존재하면 그 트랜잭션을 지원, 없다면 새로운 트랜잭션을 시작 |
| PROPAGATION_SUPPORTS      | 기존의 트랜잭션이 존재하면 그 트랜잭션을 지원, 없다면 비-트랜잭션 상태로 수행 |
| PROPAGATION_MANDATORY     | 반드시 Transaction 내에서 메소드가 실행되어야 한다. 없으면 예외 발생 |
| PROPAGATION_REQUIRES_NEW  | 언제나 새로운 트랜잭션을 수행, 이미 활성화된 트랜잭션이 있다면 일시정지 |
| PROPAGATION_NOT_SUPPORTED | 새로운 Transaction을 필요로 하지는 않지만, 기존의 Transaction이 있는 경우 Transaction 내에서 메소드를 실행 |
| PROPAGATION_NEVER         | Mandatory와 반대로 Transaction없이 실행되어야 하며 Transaction이 있으면 예외 발생 |
| PROPAGATION_NESTED        | 현재의 트랜잭션이 존재하면 중첩된 트랜잭션내에서 실행, 없으면 REQUIRED처럼 동작 |

