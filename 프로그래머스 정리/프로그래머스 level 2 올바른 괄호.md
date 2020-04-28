## 프로그래머스 level 2 올바른 괄호 

```
boolean solution(String s) {
        boolean answer = true;

        Stack<Character> stack = new Stack<>();

        for(int i=0; i<s.length(); i++){
            if(s.charAt(i) == '('){
                stack.add(s.charAt(i));
            }else{
                if(stack.isEmpty()){
                    answer = false;
                }else{
                    stack.pop();
                }
            }
        }

        if(stack.size() != 0){
            answer = false;
        }
        return answer;
    }
```



문제를 푸는 과정에서 계속 테스트 케이스 2,6 번이 틀리다고 나왔었다.

한참을 헤매다 알게 되었는데

마지막 if 문을

```
if(stack.size() == 0){
    answer = true;
}else{
	answer = false;
}
```

위와 같이 해놔서 size가 0이기만 하면 true를 리턴해버리기 때문에 해당 케이스에 대해 통과를 하지 못한 것 같다.

