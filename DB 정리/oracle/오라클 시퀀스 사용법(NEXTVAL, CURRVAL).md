## 오라클 시퀀스 사용법(NEXTVAL, CURRVAL)

1. 시퀀스 생성

   - CREATE SEQUENCE [시퀀스 이름] 

     [START WITH 시작번호] - default 1

     [INCREMENT BY 증가단위] - default 1

     [MAXVALUE 최대값] - default 오라클 버전 최대값

     [MINVALUE 최소값] - default 1

     [CYCLE | NOCYCLE]

     [CACHE | NOCACHE] - default 20

이름 빼고는 생략가능

cycle - 시퀀스가 최대 값이 되었을 때 다시 시작값으로 돌아갈 것인지

cache - 시퀀스 값을 메모리에 할당할 것인지



2. 시퀀스 수정

   - ALTER SEQUENCE [시퀀스 이름] 

     `START WITH는 수정 불가능`

3. 시퀀스 검색

   - SELECT * FROM USER_SEQUENCES;
   - 이 쿼리를 사용하면 생성된 모든 시퀀스들을 보여준다.

4. 시퀀스 삭제

   - DROP SEQUENCE [시퀀스 이름]





NEXTVAL와 CURRVAL



시퀀스 값 증가 => NEXTVAL

현재 시퀀스 값 => CURRVAL



##### 사용법

SELECT 해당시퀀스.NEXTVAL FROM DUAL; - 해당 시퀀스의 다음 값

SELECT 해당시퀀스.CURRVAL FROM DUAL; - 해당 시퀀스의 현재 값

INSERT INTO example VALUES(해당시퀀스.NEXTVAL) - 해당 시퀀스의 다음 값을 insert



select문을 하지만 실행 마다 시퀀스 값이 증가된다 => 테이블과 시퀀스는 별도이기 때문에



##### 시퀀스 초기화

초기화 방법은 현재 시퀀스의 값을 확인한 후 초기화 값을 하고 싶은 만큼 값을  빼준다

`INCREMENT BY를 사용`해서



현재 값이 4일경우

ALTER SEQUENCE 시퀀스이름 INCREMENT BY -3; 로 INCREMENT를 변경해준 후

SELECT 해당시퀀스.NEXTVAL FROM DUAL;  //-3 만큼 증가하도록 하고

SELECT 해당시퀀스.CURRVAL FROM DUAL; // 현재값을 확인하면 1로 초기화가 된다.

ALTER SEQUENCE 시퀀스이름 INCREMENT BY 1; //처리를 다 했으면 INCREMENT를 원상복구한다.



