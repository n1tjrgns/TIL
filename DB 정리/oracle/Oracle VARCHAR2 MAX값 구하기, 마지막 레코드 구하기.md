## Oracle VARCHAR2 MAX값 구하기, 마지막 레코드 구하기

seq 테이블이 VARCHAR2 타입으로 구현되어 있었다.

> 그럼 MAX값을 가져오고 싶은데 어떻게 가져오지?



다행히 이런 경우에도 가져오는 방법이 있기는 했다.

- MAX(to_number(해당 컬럼 명)) from 테이블 명 group by 그룹 아이디.



이렇게 하면 MAX인 VARCHAR2 값을 뽑을 수 있다.



그렇다면 MAX인 레코드 전체를 구하고 싶으면 어떻게 할까?



##### Oracle에서 제일 마지막 레코드를 구하는 쿼리문

```sql
# 사용방법
SELECT * FROM (SELECT * FROM 테이블명 WHERE 조건 AND 조건 ORDER BY 필드명 DESC) WHERE ROWNUM = 1


select * from (select max(to_number(seq)), reg_dt, reg_id,content from consult group by reg_dt, reg_id,content order by reg_dt desc) where rownum = 1;
```

