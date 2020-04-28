------------------------------------------------------------------------------------------------
PLAN_TABLE_OUTPUT


Plan hash value: 4190282570





------------------------------------------------------------------------------------------------

| Id  | Operation 								| Name   | Rows  | Bytes | Cost (%CPU)| Time     |

------------------------------------------------------------------------------------------------

|   0 | SELECT STATEMENT                |              |     1 |    40 |     6  (34)| 00:00:01 |

|   1 |  HASH GROUP BY                    |              |     1 |    40 |     6  (34)| 00:00:01 |

|   
2 |   
NESTED LOOPS OUTER            
|              
|     
1 |    
40 |     
5  
(20)| 00:00:01 |


|   
3 |    
VIEW                         
|              
|     
1 |    
25 |     
4  
(25)| 00:00:01 |


|   
4 |     
HASH GROUP BY               
|              
|     
1 |    
36 |     
4  
(25)| 00:00:01 |


|*  
5 |      
TABLE ACCESS BY INDEX ROWID| EBAD0020     
|     
1 |    
36 |     
3   
(0)| 00:00:01 |


|*  
6 |       
INDEX RANGE SCAN          
| EBAD0020_R_D |     
1 |       
|     
2   
(0)| 00:00:01 |


|   
7 |    
TABLE ACCESS BY INDEX ROWID  
| USERS        
|     
1 |    
15 |     
1   
(0)| 00:00:01 |


|*  
8 |     
INDEX UNIQUE SCAN           
| PK_USERS     
|     
1 |       
|     
0   
(0)| 00:00:01 |

------------------------------------------------------------------------------------------------



#### 확인방법

1. 실행계획을 보고 싶은 select 문 앞에 `EXPLAIN PLAN FOR`를 붙여준다
2. `DBMS_XPLAN.DISPLAY`로 실행계획을 조회한다.

```
EXPLAIN PLAN FOR
select문 작성;

select * from table(dbms_xplan.display);
```



DBMS_XPLAN.DISPLAY에는 옵션이 있다.





#### 테이블 설명

- Id : 실행계획의 구분자

- Operation : 각 단계에서의 작업 

- Name : 테이블 이름 또는 index 이름

- Rows : 해당 쿼리 계획 단계에서 나올 것으로 예상되는 행의 수

- Byte : 실행 계획의 각 단계가 반환할 것으로 예상 되는 데이터의 크기를 바이트로 나타낸 수
  - 행의 수와 예상 길이에 따라 결정됨
- Cost : CBO가 쿼리 계획의 각 단계에 할당한 비용, 동일한 쿼리에 대해 다양한 실행경로/계획을 생성하며 모든 쿼리에 대해 비용을 할당
- Time : 각 단계별 수행 시간





#### Predicate 설명

Predicate Information (identified by operation id):

-----

5 - filter(("POST_CHANNEL"='K' OR "POST_CHANNEL"='L' OR "POST_CHANNEL"='M' OR 

​              "POST_CHANNEL"='S') AND "JOB_STATUS"='40')

6 - access("REGISTER_D">='201909270000' AND "REGISTER_D"<'201910040000')

8 - access("A"."REGISTER_ID"="B"."USER_ID"(+))

------



- 어떤 칼럼, 값으로 access 또는 filter 되었는지 표시된다.
- access : 인덱스의 읽는 범위를 줄여주는 조건
- filter : 인덱스에서 filter만 하는 조건



`Predicate` 를 봐야하는 이유 ? 

- 실행 계획이 복잡해지면 where 절의 특정 조건이 각 단계에서 어떻게 사용되는지가 중요해진다.
- `Query Transformation`이란 과정 때문에 Predicate의 변형이 발생할 때는 이 정보가 중요하다.



-----















**참고**

https://positivemh.tistory.com/364