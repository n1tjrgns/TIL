> ## PreparedStatement 와 Statement

PreparedStatement와 Statement의 가장 큰 차이점은 캐시(cache)의 사용여부이다.



Statement를 사용하면 매번 쿼리를 수행할 때마다 `쿼리 문장 분석, 컴파일, 실행` 3단계를 거치게 되지만,

PreparedStatement는 처음 한 번만 3단계를 거친 후 캐시에 담아 재사용을 한다. 만약 동일한 쿼리를 반복적으로 수행한다면, PreparedStatement가 DB에 훨씬 적은 부하를 주며, 성능도 좋다.



**Statement**

소스를 보면 좀 더 알기 쉽다.

```java
String sqlStr = "select name from user where num = 10 "
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery(sqlStr);
```

쿼리를 담은 변수인 sqlStr을 executeQuery 또는 executeUpdate를 실행하는 시점에 파라미터로 전달한다.

그렇기 때문에 SQL문을 수행하는 과정에서 매번 컴파일을 하기 때문에 성능상 이슈가 발생한다.



**PreparedStatement**

```java
String sqlStr = "select name from user where num = ? "
PreparedStatement pstmt = conn.prepareStatement(sqlStr);
pstmt.setInt(1,num);
ResultSet rs = pstmt.executeQuery();
```

PreparedStatment 준비된 Statement에서 준비가 되었다는 것은 컴파일(Parsing)을 이야기한다.

컴파일이 미리 되어있기 때문에 한눈에 Statement에 비해 성능상 이점이 있다. 





또한 각각 where절 이하를 보면, Statement는 정확한 값, PreparedStatement는 ? 로 작성되어 있는 것을 볼 수 있다.

Statement는 정적인 쿼리문을, PreparedStatement는 동적인 쿼리문을 처리한다.

















<https://devbox.tistory.com/entry/Comporison>

