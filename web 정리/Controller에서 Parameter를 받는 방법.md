## Controller에서 Parameter를 받는 방법

`httpServletRequest.getParameter()` - 가장 흔히 사용하는 일반적인 방법

```java
@RequestMapping("/test")
public String test(HttpServletRequest req){
	String userId = req.getParamenter("userId");
	return "test";
}
```





`httpServletRequest.getAttribute()`

getParameter()와 비슷하다.

차이점은 getParameter는 String 객체로 처리하고, getAttribute는 Object로 처리한다.

```java
User user = (User)req.getAttribute("user");
String userId = user.getUserId();
```



`@RequestParam` - 파라미터와 1:1 대응

name, required, defaultValue 속성값을 가진다

required 값이 true일 경우 해당 parameter는 반드시 request에 담겨져 있어야 한다.(없을 경우 400 에러)

```java
@RequestMapping("/test")
public String test(@RequestParam(value="userId", required = false, 																	defaultValue="test123") String userId){
	return "test";
}
```

파라미터가 필수로 들어 있는 경우가 아닌 경우에는 required = false, default 값을 지정해준다.

반대로 required = true 인 경우, 반드시 값이 들어있기 때문에

```java
public String test(@RequestParam(value="userId")String userId {

}
```

이렇게만 사용할 수 있다.





`@RequestBody` 

@RequestBody로 파라미터를 받으려면 반드시 POST 방식이어야 한다.

JSON이나 XML에서 던져준 데이터를 받을 경우 주로 사용한다.





`@ModelAttribute` 

@RequestParam이 1:1로 파라미터를 받았다면, @ModelAttribute는 DTO/VO로 받을 경우 사용한다.

```java
public String test(@ModelAttribute("User") User user){
	log.info(user.getUserId());
	return "test";
}
```

AJax로 객체배열을 넘길 때도 @ModelAttribute를 사용해 받을 수 있다. (객체 안에 List로 Type의 변수를 가지고 있을 경우)

아니면 @RequestBody도 가능하다. (JSON.stringify()를 이용해 배열형태로 받기)



각각 상황에 맞게 잘 선택해 사용하자.

