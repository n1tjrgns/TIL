## Truncate, Delete, Drop



#### 테이블에서 행을 삭제하는 방법

1. DELETE (데이터만 삭제)

   ```
   DELETE FROM example;
   ```



- DELETE 문을 사용할 때 TABLE이나 CLUSTER에 행이 많으면 삭제 될 때마다 많은 SYSTEM 자원이 소모된다.

- 트리거에 걸려있으면 각 행이 삭제될 때 실행된다.
- 이전에 할당되었던 영역은 삭제되어 `빈 테이블이나 클러스터에 그대로 남아` 있게 된다.
- 커밋을 하지 않았다면 롤백이 가능하다.
- 전체 or 일부 삭제가 가능하다.(WHERE 절 사용)
- 디스크에 용량은 그대로 남아있다. 
  - 데이터만 없어질 뿐.



2. DROP (테이블 전체 삭제)

   ```
   DROP TABLE example;
   ```

   

- 테이블이나 클러스터를 삭제하면 모든 관련 INDEX, CONSTRANINT, TRIGGER, 권한도 삭제된다.



3. TRUNCATE 

   ```
   TRUNCATE TABLE example;
   ```



- TRUNCATE는 TABLE이나 CLUSTER에서 모든 행을 삭제하는 빠르고 효율적인 방법이다.
  - 테이블을 최초 생성된 상태로 만듦.
- 롤백 정보를 만들지 않고 즉시 COMMIT => TRUNCATE를 사용하면 `롤백이 불가능`하다.
- TRUNCATE를 사용해 삭제할 테이블과 관련된 구조(제약조건, 트리거), 권한에 영향을 주지 않는다.
- TRUNCATE 명령문으로 행을 삭제하면 테이블에 걸려있는 트리거는 실행되지 않는다.
- 삭제 행수를 반환 하지 않는다.
- 무조건 전체 삭제만 가능하다. (WHERE 절 사용 불가능)
  - 디스크의  용량, 인덱스도 모두 삭제됨.

