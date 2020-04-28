## Oracle SQL Trace 확인하는 방법



Sql 문에 인덱스를 추가하고, 전 후 속도차이를 비교하기 위해 SQL Trace를 확인해보려고 했다.



- sqlplus에서 sysdba 계정으로 해야한다





```
SELECT P.SPID SERVER FROM V$PROCESS P, V$SESSION S WHERE P.ADDR = S.PADDR AND S.AUDSID = USERENV('SESSIONID');
```

우선 위의 쿼리를 사용해서 SPID를 확인해야한다.



나중에 trc 파일을 떨구면 SPID 번호로 생성이된다.

어차피 최신 파일을 보면 되기 때문에 위 과정은 꼭 필요하진 않는 것 같다.



```
alter session set events '10046 trace name context forever, level 1';
```

트레이스의 레벨을 설정할 수 있다. 레벨이 높을 수록 좀 더 심화된 트레이스를 할 수 있다.



```
ALTER SESSION SET SQL_TRACE=TRUE;
```

Sql_Trace 옵션을 false로 해야하지만, 처음 하고나서 다음에 실행 할 때 위 설정을 미쳐 생각하지 못할 수 있어 `TRUE`로 한 번 바꿔주고 하는게 좋은 것 같다.



이제 확인하고 싶은 쿼리를 실행한다.



`select ~~~~`



```
ALTER SESSION SET SQL_TRACE=FALSE;
```

쿼리 실행 결과가 나왔다면 다시 옵션을 `false`로 바꿔준다.



여기까지 수행했다면

```
$Oracle_home/{스키마}/admin/udump
```

경로에 SPID로 .trc라는 파일이 생성되어 있을 것이다.



이 파일은 열어보면 상당히 복잡하게 되어있는데 이를 시각적으로 바꿔주는 Oracle 명령어가 있다.

- tkprof



이 파일은 $Oracle_home의 bin 안에 들어가있다.

```
./tkprof (생성된 trc 파일의 경로와 이름) (kprof로 가공할 파일 이름(test.txt))
```

이렇게 작성하면 해당 bin 파일에 test.txt 파일이 생성되어 있을 것이다.



해당 파일을 열어 쿼리 실행 결과를 확인하자.

tkprof가 제공하는 옵션들을 사용하면 여러가지 정보를 확인 할 수 있다.

- 구글링 ㄱ,ㄱ





oracle rebuild

alter index 'name' rebuild

