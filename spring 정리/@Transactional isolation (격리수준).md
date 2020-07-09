## [Spring] @Transactional - 2 isolation (격리수준)

[이전 글](https://n1tjrgns.tistory.com/266)에서 트랜잭션 전파레벨에 대해서 알아보았다.

트랜잭션을 사용할 때, 전파레벨과 함께 따라오는 것이 `격리수준`이다.

트랜잭션에서 `일관성이 없는 데이터를 허용하도록 하는 수준`을 말한다.



격리수준에는 `4가지`가 있다.

- READ_UNCOMMITED (level 0)
- READ_COMMITED (level 1)
- REPEATABLE_READ (level 2)
- SERIALIZABLE (level 3)



격리 수준이 높아질수록 동시성(Concurrency)은 높아지고 속도는 느려진다.

이 둘의 `균형`을 잘 맞추는것이 중요하다.

----

#### READ_UNCOMMITED (커밋되지 않는 읽기, level 0)

- 트랜잭션 처리중 or 아직 commit되지 않은 데이터를 다른 트랜잭션이 읽는 것을 허용

- 어떤 사용자가 A라는 데이터를 B로 변경하는 동안 다른 사용자는 B라는 아직 완료되지않은(Uncommitted 혹은 Dirty) 데이터 B를 읽을 수 있다.



> Dirty read 발생
>
> 다른 트랜잭션에서 처리하는 작업이 완료되지 않았는데도 다른 트랜잭션에서 읽을 수 있는 현상
>
> READ_UNCOMMITED 격리수준에서만 발생



#### READ_COMMITED (커밋된 읽기, level 1)

- `dirty read 방지` : 트랜잭션이 commit 되어 확정된 데이터만을 읽도록 허용
- A라는 데이터가 B로 변경되는 동안 다른 사용자는 해당 데이터에 접근할 수 없다.



#### REPEATABLE_READ (반복 가능한 읽기, level 2)

- 트랜잭션이 완료될 때까지 SELECT 문장이 사용하는 모든 데이터에 `shared lock`이 걸린다.

  따라서, 다른 사용자는 해당 영역에 대한 데이터 수정이 불가능하다.

- 선행 트랜잭션이 읽은 데이터는 트랜잭션이 종료될 때까지 후행 트랜잭션이 갱신하거나 삭제하는 것을 불허함으로써 같은 데이터를 두 번 쿼리했을 때 일관성 있는 결과를 리턴한다.

> Phantom READ 발생
>
> 트랜잭션이 데이터를 두 번 읽는다고 가정 할 때 where 절의 조건에 맞는 레코드가 추가로 생성될 때, 
>
> 새로운 "유령(phantom)"행이 나오지만 지원하지 phantom read를 지원하지 않으면 새로 생긴 행을 볼 수 없다.



#### SERIALIZABLE (직렬화 가능, level 3)

- 완벽한 읽기 일관성 모드 제공(가장 엄격함)
- 성능 측면에서 동시 처리성능이 가장 낮다.
- 거의 사용되지 않는다.
- 데이터의 일관성 및 동시성을 위해 `MVCC`(Multi Version Concurrency Control)를 사용하지 않음
- 트랜잭션이 완료될 때까지 SELECT 문장이 사용하는 모든 데이터에 `shared lock`이 걸리므로 다른 사용자는 그 영역에 해당하는 데이터에 대한 수정 및 입력이 불가능하다.

> MVCC
>
> 다중 사용자 DB 성능을 위한 기술
>
> 데이터 조회시 LOCK을 사용하지 않고, 데이터의 버전을 관리해 데이터의 일관성 및 동시성을 높이는 기술



----

#### 각 DB 별 isolation

```
MYSQL : REPEATABLE READ
ORACLE : READ COMMITTED
H2 : READ COMMITTED
```





**참고**

 https://taetaetae.github.io/2016/10/08/20161008/ 

 https://lng1982.tistory.com/287 

 [https://effectivesquid.tistory.com/entry/%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4-Isolation-Level](https://effectivesquid.tistory.com/entry/데이터베이스-Isolation-Level) 

 https://nesoy.github.io/articles/2019-05/Database-Transaction-isolation 

