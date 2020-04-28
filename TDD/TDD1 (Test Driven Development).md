#### TDD (Test Driven Development)

- 테스트 주도 개발



#### 흐름

![image-20191127161842064](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20191127161842064.png)

1. 무엇을 테스트할 것인가 생각한다.
2. 실패하는 테스트를 작성한다.
3. 테스트를 통과하는 코드를 작성한다.
4. 코드를 리팩토링한다. (테스트코드 또한 리팩토링한다)
5. 구현해야 할 것이 있을 때까지 위의 작업을 반복한다.



#### 핵심

- `테스트를 먼저 만들고 테스트를 통과하기 위한 코드를 짜는 것`

- 이미 완성 된 코드에 테스트 케이스를 작성하는 것은 TDD가 아님



#### TDD를 적용해야 하는 상황

- 처음 해보는 주제 : 불확실성이 높음
- 개발 중 코드를 많이 바꿔야 된다고 생각되는 경우
- 요구조건이 자주 바뀌는 경우



#### TDD의 효과

##### `피드백`

- TDD를 하면 피드백이 증가한다.
  - 테스트 케이스의 통과를 통해 잘 동작하는지 확인할 수 있다.



#### TDD 도입을 꺼려하는 이유

- 개발속도가 느려진다고 생각하는 사람이 많다.
  - 처음부터 2개의 코드 작성(서비스 로직, 테스트 로직)
  - 테스트를 하며 계속 테스트 코드를 고쳐야한다.



- 많은 기업들이 단기적인 성과에 집중해 있다.
  - 전체 개발 시간을 줄이는 것보다 오늘 일을 끝내는 것을 강조
  - 그때가서 고치지 뭐 ...



- TDD 자체의 어려움

  - 여태 개발 방식과는 많이 다르다(테스트 코드부터 작성)

  - **`테스트를 할 수 있는 것` VS `테스트를 할 수 없는 것`**

    



#### TDD의 장단점

- 개발 후 테스트 단계에서 예상치 못한 결함을 줄일 수 있다.
  - 어떻게 테스트할지 고민하는 단계에서 해당 결함을 찾아 낼 수 있다.
  - 연구 결과 TDD를 작성하지 않은 경우 개발 시간이 10~30% 늘어남
  - 결함이 1/2 ~ 1/10 까지 줄어듬
  - 유지보수 비용이 낮아진다.
  - 깨끗한 코드가 나온다.



-----

```
@MockBean //가짜 객체 주입
    private RestaurantService restaurantService;
    
@Test
    public void list() throws Exception {
        //Mock 객체를 가짜로 주입함으로써 Service에 의존하지 않고 테스트가 가능하다.
        List<Restaurant> restaurants = new ArrayList<>();
        restaurants.add(new Restaurant(1004L,"Bob","Seoul"));
        given(restaurantService.getRestaurants()).willReturn(restaurants);

        mvc.perform(get("/restaurants"))
                .andExpect(status().isOk()) //getMapping의 상태값 확인
                .andExpect(content().string(containsString("Bob"))); //콘텐츠 안의 값이 Bob이 포함되어있는지
    }    
```

- given
- willReturn

