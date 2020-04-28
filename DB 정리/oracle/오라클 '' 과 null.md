## [Trouble Shooting] 오라클 '' 과 null



```
SELECT * FROM test a LEFT JOIN example b
ON a.ID = b.ID
WHERE a.ID = 000000 
AND b.flag = 'Y'
AND NVL(a.ERROR_CODE, '') NOT in ('',' ','--');
```

이런 쿼리가 있었는데 ERROR_CODE가 있음에도 불구하고 출력이 되지 않았다.



> 왜???



오라클에서 길이가 없는 문자열(aka empty string, white space, blank)은 null 로 취급된다.

즉, 필드에 들어있는 값이나 '' 문자 또는 함수가 반환하는 값이 길이가 없는 문자열이라면 null 로 취급된다.

이러한 특징은 다른 DBMS 와 다르므로 주의해야 한다.

```
SELECT NVL(null, '') FROM DUAL; // null
```



그래서 쿼리를 아래와 같이 수정했더니 해결되었다.

```
SELECT * FROM test a LEFT JOIN example b
ON a.ID = b.ID
WHERE a.ID = 000000 
AND b.flag = 'Y'
AND NVL(a.ERROR_CODE, '') NOT in (' ','--');
```

