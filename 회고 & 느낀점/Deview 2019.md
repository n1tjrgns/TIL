1. ## 로그

로그가 너무 많아서 일일이 관리하기가 힘들어 `로그 저장소`를 만듦



- 유실없는 로그저장 (`하이브가 핵심`)

  - 최소 한 번은 데이터를 전달하도록 At-leaset-once-Delivery 데이터를 보낸 후 일정시간 응답이 안오면 다시 데이터를 보냄 - `데이터 중복이 발생 할 수 있음(중복처리)` - 고유 식별자를 sender에 같이 담아서 식별자를 기준으로 판단.
  - 고유식별자 발급 방법
    - 해쉬값 (X)
    - 시퀀스 값 (X)
    - UUID (O)
  - RowKey 디자인 
    - Region Group MessageType(R,G,M)
    - ex) 네이버에 데뷰라고 검색하면
    - R = korea, G = naver_search, M = query_log
    - 하지만 같은 시간에 많은 데이터 발생시 핫스팟이 발생해서, 시간,ip 등을 묶어 salt키로 만듦

- 보안 정보에 대한 처리

  



네이버 검색 로그(연관 검색어, 실시간 급상승, 추천뉴스) - 로그가 엄청 중요함



ORC 포맷을 사용해 특정 컬럼만 읽는 방식으로 디비 저장 공간도 확보하고 속도도 빠름

배치를 돌려서 미리 데이터를 가공



-----



## 03. SRE ( 사이트 신뢰성 엔지니어링)

##### 네이버의 sre 도입 배경

1. 365/24 무중단 서비스
2. 서비스 규모가 커져도 엔지니어는 한정
3. 재난 상황과 같은 트래픽 급증



#### fail fast, learn faster

이상 징후 탐지, 의사 결정, 복구, 작업영향도 파악, 원인 분석



> 장애를 줄일 수 있는 시간과, 줄일 수 없는 시간으로 분류



복구시간과, 원인을 차단하는 일은 많은 시간이 소요되어 줄일 수 없음



- 이상 징후 탐지의 시간을 줄이자

```
임계 상황 판단 공식 = 부하 증가 배수 > 최대 가용 배수

부하 증가 배수 : 한 서버가 죽으면 나머지 서버들은 몇 배의 부하가 생기는지
최대 가용 배수 : 한 서버가 현재 몇 배까지 받을 수 있는지
```





대량의 경보 > 담당자의 피로 증가 > 거짓 경보를 줄이기

- Pending

  즉각적으로 대응해야 하는 경보가 아닌 경우 우선순위 변경





------

4. #### 네이버 컨텐츠 검색, 몽고DB

몽고DB를 사용하는 이유?

- 빠름
- Scalable
- 높은 가용성



#### 몽고DB 속도올리기 - INDEX

- 컬렉션당 최대 64개의 인덱스 생성가능
- 너무 많은 인덱스 추가시 side effects 발생
  - 메모리에 인덱스를 올려놓기 때문에.
  - 실행계획에서 잘못된 인덱스를 선택 할 수 있음.
- Frequent Swap
- Write performance 감소



Index Prefix 사용

- 집합으로 인덱스를 만들어 놓으면 나머지 각각의 인덱스이 부분집합에 있는 경우 커버 할 수 있음

- `컴파운드 인덱스`

멀티 소팅

- Sort key들은 반드시 인데긋와 같은 순서로 나열 되어야만 한다.



```
하나의 컬렉션이 너무 많은 문서를 가지는 경우, 인덱스 사이즈가 증가하여 인덱스 필드의 cardinality가 낮아질 가능성이 높다.
```



쓰레드를 사용해 대량의 Document를 upset

- 스프링 배치로 여러개의 thread에서 Bulk Operation으로 많은 document를 한번에 write



몽고DB 4.0부터 

write가 primay에 반영되고 secondary에 다 전달 될떄까지 seondary는 read를 block해서 데이터가 잘못된 순서로 read되는 것을 방지함.



key examined 값을 8천 아래로. 잡고 쿼리 튜닝을하여 1초안에 결과값을 1초안에 보여줄 수 있도록



인덱스 힌트를 주면 제대로 속도가 나오는데, 쿼리 플랜은 그 인덱스를 제대로 찾지 못함. 왜?

- 쿼리 플래너의 문서를 찾아봄
- limit 101 row를 가장 빠르게 검색하는 쿼리를 최적의 쿼리라고 판단해서 가져오고있었음



>  실행계획이 역시 중요하다.



#### Hint 이용 vs 엄한 인덱스를 지우기

| Hint 이용                                   | 엄한 인덱스 지우기                                  |
| ------------------------------------------- | --------------------------------------------------- |
| 확실환 쿼리플랜 보장                        | 데이터에 따라 더 효율적인 인덱스가 생기면 자동 보장 |
| 더 효율적인 인덱스가 생겨도 강제 고정       | 또다른 엄한 케이스가 생길 수 있음                   |
| 데이터 분포에 대한 정보를 계속 follow       | ㅇ삭제로 인해 영향 받는 다른 쿼리가 생길 수 있음    |
| 32MB넘는 응답 결과를 sort 해야 할 경우 에러 |                                                     |



-----

5. #### 눈발자국 (Tracing)

- 개발자가 Printf를 활용하는 방식 = instrument
- 단, 자기가 만든 부분만 printf를 찍을 수 있음 > 고도화된 Printf가 필요







-----

#### 스케일아웃없이 급증하는 트래픽 처리

1. 마이크로서비스로 전환



long 트랜잭션을 분리, 비동기로 처리



1.고객의 주문을 rest-api로 카프카에 저장

2. 카프카에 있는데이터를 가공 후 다시 저장
3. 오라클에 최종 데이터를 저장



카프카 기반 서비스 구현

1. consumer group

- 하나의 컨슈머 그룹은 한번만 처리된다.

2. rebalance



 rest - 결제 요청을 보내면 처리결과를 받는다.

event drivern - 결제 요청을 보내면 카프카에 저장이 됐는지 안다. (중간에 메시지가 유실됐는지는 알 수 없다.)

주문번호를 key로 하여 5분간 timewindow를 생성해 응답이 없을시 데이터 유실로 간주



backpressure

- 자신이 처리 할 수 있는 만큼만의 데이터를 가져옴으로써 장애가 발생하지 않게함









카프카를 쓴 이유

oracl



chaos engineering