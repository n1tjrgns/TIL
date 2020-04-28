## [MySql] 테이블 조인

MySql은 다른 DBMS보다 조인을 처리하는 방식이 단순하다. 현재 릴리즈된 MySql의 모든 버전에서 조인 방식은 `Nested-loop` 로 알려진 중첩된 루프와 같은 형태만 지원한다. 그리고 `조인되는 각 테이블 간의 레코드를 어떻게 연결할지`에 따라 여러 가지 종류의 조인으로 나뉜다.



#### 테이블 조인의 종류

조인은 크게 INNER, OUTER JOIN으로 구분할 수 있고, OUTER JOIN은 다시 LEFT OUTER, RIGHT OUTER, FULL OUTER JOIN으로 구분할 수 있다. 그리고 조인의 조건을 어떻게 명시하느냐에 따라 NATURAL, CROSS JOIN(ULL, CARTESIAN)으로 구분 할 수 있다.

`조인을 할 때 어느 테이블을 먼저 읽을지 결정`하는 것은 상당히 중요하며, 그에 따라 `처리할 작업량이 크게 달라진다.`

**INNER JOIN**은 어느 테이블을 먼저 읽어도 결과가 달라지지 않으므로 MySQL 옵티마이저가 조인의 순서를 조절해 다양한 방법으로 최적화를 수행할 수 있다.

하지만 OUTER JOIN은 반드시 OUTER가 되는 테이블을 먼저 읽어야 하기 때문에 조인 순서를 옵티마이저가 선택할 수 없다.



#### INNER JOIN

일반적으로 `조인`은 INNER JOIN을 지칭하는데, OUTER JOIN과 구분할 때 INNER JOIN이라고 한다.

위에서 언급했듯이 MySql에서 조인은 Nested-loop 방식만을 지원한다.

> Nested-loop : 프로그램을 작성할 때 두개의 for, while과 같은 반복 루프 문장을 실행하는 형태로 조인이 처리되는 것을 의미한다.



의사코드

```mysql
for (record1 in table1) { //외부 루프(outer)
	for(record2 in table2) { //내부 루프(inner)
		if(record1.join_column == record2.join_column){
			join_record_fount(record1.*, record2.*);
		}else{
			join_record_notfount();
		}
	}
}
```

위의 의사코드에서 알 수 있는 점은 조인은 2개의 반복 루프로 두 개의 테이블을 조건에 맞게 연결해주는 작업이다.

두 개의 for 문장에서 바깥쪽을 outer 테이블이라고 하고, 안쪽을 inner 테이블이라고 표현한다.

또한 outer 테이블은 inner 테이블보다 먼저 읽어야 하고, 조인에서 주도적인 역할을 한다고 해서 `드라이빙(Driving)테이블`이라고도 한다. 반대로 inner 테이블은 끌려가는 역할을 한다고 해서 `드리븐(Driven) 테이블`이라고 한다.

중첩된 반복 루프에서 최종적으로 선택될 레코드가 안쪽 반복 루프 inner 테이블에 의해 결정되는 경우를 INNER JOIN이라고 한다.

즉, 두 개의 반복 루프를 실행하면서 table2 inner table의 if문 이하의 조건을 만족하는 레코드만 조인의 결과로 가져온다.



#### OUTER JOIN

의사코드

```mysql
for (record1 in table1) { //외부 루프(outer)
	for(record2 in table2) { //내부 루프(inner)
		if(record1.join_column == record2.join_column){
			join_record_fount(record1.*, record2.*);
		}else{
			join_record_notfount(record1.*, NULL);
		}
	}
}
```

위 코드에서 table2가 일치하는 레코드가 있으면 INNER 조인과 같은 결과를 만들어내지만, 일치하는 레코드가 없는 경우에는 table2의 칼럼을 모두 null로 채워서 가져온다.

즉, INNER JOIN은 일치하는 레코드를 찾지 못하면 table1의 결과를 모두 버리지만 OUTER JOIN은 table1의 결과를 버리지 않고 그대로 결과에 table1의 결과를 버리지 않고 그대로 포함한다.

`inner 테이블의 조인이 결과에 전혀 영향을 미치지 않고, outer 테이블의 내용에 따라 조인의 결과가 결정되는 것`이 `OUTER JOIN`의 특징이다.



그리고 다시 OUTER JOIN은 조인의 결과를 결정하는 outer 테이블이 조인의 왼쪽, 오른쪽 위치에 따라 LEFT, RIGHT JOIN 그리고 FULL OUTER JOIN으로 나뉜다.

```mysql
select * from employee e
left outer join salary s on s.emp_no = e.emp_no;

select * from salary s
right outer join employee e on e.emp_no = s.emp_no;
```



LEFT OUTER, RIGHT OUTER 조인은 결국 처리 내용이 같기 때문에 혼동을 막기 위해 LEFT OUTER JOIN으로 통일해서 사용하는 것이 일반적이다.



FULL OUTER JOIN은 JOIN 키워드를 기준으로 왼쪽 테이블도 OUTER JOIN도 하고 싶고, 오른쪽 테이블도 하고 싶은 경우 FULL OUTER JOIN을 하는데, MySql 에서는 Full OUTER JOIN을 지원하지 않는다.



MySql의 실행계획은 어떤 JOIN을 사용했는지 알려주지 않기 때문에 OUTER JOIN을 의도한 쿼리가 INNER JOIN으로 실행되지 않는지 주의해야 한다. 

그래서 `OUTER JOIN에서 레코드가 없을 수도 있는 쪽의 테이블에 대한 조건은 반드시 LEFT JOIN의 ON 절에 명시`하는 것이 좋다. 그렇지 않으면 옵티마이저는 OUTER JOIN을 내부적으로 INNER JOIN으로 변형시켜 처리해 버릴 수 있다.

LEFT OUTER JOIN의 ON 절에 명시되는 조건은 조인되는 레코드가 있을 때만 적용된다.

하지만, where 절에 명시되는 조건은 INNER, OUTER JOIN에 관계없이 조인된 결과에 대해 모두 적용된다. 그래서 OUTER_JOIN으로 연결되는 테이블이 있는 쿼리에서는 가능하면 모든 조건을 ON 절에 명시하는 습관을 들이는 것이 좋다.

```mysql
select * from employee e
left outer join salary s on s.emp_no = e.emp_no;
where s.salaries > 5000; 
```

 이러한 형태의 쿼리는 아래 2가지 중 한 방식으로 수정해야 쿼리 자체의 의도나 결과를 명확히 할 수 있다.



```mysql
//순수하게 OUTER JOIN으로 표현한 쿼리
select * from employee e
left outer join salary s on s.emp_no = e.emp_no AND s.salaries > 5000;

//순수하게 INNER JOIN으로 표현한 쿼리
select * from employee e
inner join salary s on s.emp_no = e.emp_no
where s.salaries > 5000;
```



LEFT OUTER JOIN이 아닌 쿼리에서는 검색 조건이나 조인 조건을 where 절이나 on 절 중에서 어느 곳에 명시해도 성능상의 문제나 결과의 차이가 나지 않는다.



#### 조인 관련 주의 사항

MySql의 조인 처리에서 특별히 주의해야 할 부분은 `실행 결과의 정렬 순서`와 `INNER JOIN과 OUTER JOIN`의 선택 2가지이다.





참고

<https://12bme.tistory.com/165>

<https://jojoldu.tistory.com/173>