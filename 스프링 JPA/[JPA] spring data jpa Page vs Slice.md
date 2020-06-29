## [JPA] spring data jpa Page vs Slice

> 페이지와 슬라이스는 무슨 차이점이 있을까?



```
Page<User> findByLastname(String lastname, Pageable pageable);

Slice<User> findByLastname(String lastname, Pageable pageable);

List<User> findByLastname(String lastname, Sort sort);

List<User> findByLastname(String lastname, Pageable pageable);
```



- Pageable 객체를 쿼리 메소드로 전달해 쿼리에 페이징을 동적으로 추가할 수 있다.

  page는 사용 가능한 데이터의 총 개수 및 전체 페이지 수를 알 수 있다.

  - 총 개수를 알아내기 위해 추가적으로 카운트 쿼리가 실행된다.
  - 기본적으로 카운트 쿼리는 실제로 실행되는 쿼리에서 파생된다.

  

- Slice는 Page에서 카운트 쿼리에 많은 비용이 발생하는 경우에 Slice를 사용하면 된다. 

  Slice는 다음 Slice가 존재하는지 여부만 알기 때문에 전체 데이터의 셋의 크기가 큰 경우에는 Slice를 사용하는 것이 성능상 유리하다.

- Pageable을 통해서 정렬을 할 수 있지만, 정렬만 하는 경우 Sort를 사용하는 것이 좋다.

- 결과를 단순히 List로 받을 수 있다.

  이 경우 Page 인스턴스를 생성하기 위한 메타데이터가 생성되지 않기 때문에 `카운트 쿼리`가 실행되지 않는다. 

  단순히 주어진 범위내의 엔티티를 검색하기 위한 쿼리만 실행된다.

  

**결론**

 사용하고자 하는 용도에 알맞게 쓰자 