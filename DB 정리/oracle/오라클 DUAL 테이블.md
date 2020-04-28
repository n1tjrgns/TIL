## 오라클 DUAL 테이블

##### Dual 테이블

- 오라클 자체에서 제공되는 테이블
- 간단하게 함수를 이용해서 계산 결과값을 확인 할 때 사용하는 테이블



##### 사용용도

- 사용자가 함수(계산)를 실행할 때 임시로 사용하는데 적합하다.
- 함수에 대한 쓰임을 알고 싶을 때 특정 테이블을 생성할 필요없이 dual 테이블을 이용해 함수의 값을 리턴 받을 수 있다.

ex) 

```oracle
SELECT 시퀀스.NEXTVAL FROM DUAL;
SELECT SYSDATE FROM DUAL;
SELECT CURRENT_DATE FROM DUAL;
MERGE into 내부에서 사용
```

