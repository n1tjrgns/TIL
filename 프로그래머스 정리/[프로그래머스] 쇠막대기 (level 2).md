## [프로그래머스] 쇠막대기 (level 2)

[프로그래머스 쇠막대기](https://programmers.co.kr/learn/courses/30/lessons/42585)



이 문제는 해결 방법을 생각해내기가 정말 쉽지 않았다.



내가 계속 전체 막대기를 한 번에 계산하려해서 더더욱 해결 방법을 찾지 못한 것 같다.



중요 포인트는 `()`를 만나기 전까지 `(`를 스택에 담고, `()`를 만나면 여태까지의 스택 사이즈를 더해준다.

만약 레이저를 만난 후 `())`  `)` 가 하나 더 나왔다면 막대기 한 개가 전부 잘렸기 때문에 끝부분 +1을 해준다. 



```
public int solution(String arrangement) {

        int answer = 0;
        String bar[] = arrangement.split("");

        Stack<String> stack = new Stack<>();

        for(int i=0; i<bar.length; i++){
            if("(".equals(bar[i])){
                stack.add(bar[i]);
            }else{
                stack.pop();
                if("(".equals(bar[i-1])){
                    answer = answer + stack.size();
                }else{
                    answer++;
                }
            }
        }

        return answer;
    }
```

