# Spring Batch - meta table

##### Batch_Job_Instance

![1552792007457](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1552792007457.png)

MySql에서 조회를 해보면 아래와 같이 검색이 된다.

- Job_Instance_ID : 테이블의 PK
- Job_Name : 수행한 Batch Job Name



Batch_Job_Instance 테이블은 Job Parameter에 따라 생성되는 테이블이다.

Job Parameter는 Spring Batch가 실행될 때 외부에서 받을 수 있는 파라미터이다.

예를 들어, 특정 날짜를 Job Parameter로 넘기면 Spring Batch에서는 해당 날짜 데이터로 조회/가공/입력 등의 작업이 가능하다.

같은 Batch Job이라도 Job Parameter가 다르면 Batch_Job_instance에는 기록되고, 같다면 기록되지 않는다.