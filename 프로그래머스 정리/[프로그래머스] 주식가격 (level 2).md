## [프로그래머스] 주식가격 (level 2)

이 문제는 그렇게 많이 어려웠던 문제는 아니였지만, 내가 문제를 풀었던 과정을 기록해야 할 필요를 느껴 정리를 하려한다.



[문제](https://programmers.co.kr/learn/courses/30/lessons/42584)



- 소스코드

```
public int[] solution(int[] prices) {

        int[] answer = new int[prices.length];

        LinkedList<Integer> stockQue = new LinkedList<>();
        ArrayList<Integer> result = new ArrayList<>();

        for(int p : prices){
            stockQue.add(p);
        }

        int stockSize = stockQue.size();


        while(!stockQue.isEmpty()){
            int current = stockQue.poll();
            //System.out.println("current : "+ current);

            int cnt=0;
            ListIterator it = stockQue.listIterator();
            while(it.hasNext()){
                int next = (int) it.next();
                //System.out.println("next : " + next);

                //그런데 바로 뒤의 인덱스가 조건에 맞지 않을 경우 1이 아닌 0이 나와버리네
                //3 다음에 2가 나온 경우 1초만에 떨어졌으니 이 경우를 1초라고 보는듯
                if(current <= next){
                    cnt++;
                }else{
                    cnt++;
                    break;
                }
            }

            result.add(cnt);
            //System.out.println(result);

        }

        //배열은 항상 초기화를 해놓아야 에러가 안난다.
        for(int i=0; i<result.size(); i++){
            answer[i] = result.get(i);
        }

        return answer;
    }
```

처음 큐에서 하나하나의 인덱스를 비교하려 했지만, 그 방법이 마땅치 않아 리스트로 풀었다.

비교를 하고 난 요소는 필요가 없으므로 remove를 해주는 방식으로 count를 체크했더니

for문의 반복 횟수가 계속 바뀌어 원하는 값이 나오지 않았다.



결국 다시 큐로 복귀

```
while(!nums.isEmpty()){
            int num=nums.poll();
            int cnt=0;
                for(int index=totalSize-nums.size();index<totalSize; index++){
                    if(num>stockQue.get(index)){
                                            }
                    //이하 생략
                }
        }
```

nums 라는 큐에 주식 가격들을 다 넣어놓은 후,

전체 주식의 길이 totalSize에서 큐에서 하나씩 뺀 size를 계산하여 index를 구한 후

가격이 감소했는지의 여부를 구하려했다.



하지만 stockQue.get(index)의 값이 내가 예상했던대로 나오지 않았다.



> ???



구글링을 하다가 생활코딩의 글에서 아래와 같은 글을 보았다.

이는 흔히 ArrayList와 LinkedList의 차이점에 해당하는데 간과하고있었다.



```
linkedlist는 검색에 효율적이지 않다. 
get을 호출할 때마다 내부적으로 반복문이 실행된된다.
이런 경우Iterator를 사용하는 것이 좋다. 
Iterator는 내부적으로 반복 처리된 노드가 무엇인지에 대한 정보를 유지하기 때문이다. 
그래서 linkedlist의 get을 호출할 때마다 내부적으로 반복이 실행되어 원하지 않는 값이 나올 수 있다.

그래서 내가 stockQue.get(index)를 하였을 때 원하지 않는 값이 나왔다.
```



그래서 iterator를 사용해서 풀었더니 통과가 되었다.





다른사람의 풀이

```
class Solution {
    public int[] solution(int[] prices) {
        int[] answer = new int[prices.length];

        for(int i = 0; i < prices.length; i++)
        {
            for(int j=i+1; j < prices.length; j++)
            {
                if(prices[i] > prices[j])
                {
                    answer[i] = j-i;
                    break;
                }
                else
                    answer[i] = j-i;
            }
        }

        //System.out.println(Arrays.toString(answer));

        return answer;
    }
}

```

나도 처음에 list를 사용하여 풀려고 했었는데 잘 안되었다.



다른 사람의 풀이를 보니, j-i를 빼주면 결국 그 차이 값이 시간이 된다.

`와-우`



저 생각으로 인해 for문안의 조건이 상당히 간결해졌다....

