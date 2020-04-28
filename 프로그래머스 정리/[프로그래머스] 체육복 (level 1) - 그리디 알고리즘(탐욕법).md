## [프로그래머스] 체육복 (level 1) - 그리디 알고리즘(탐욕법)



[문제](https://programmers.co.kr/learn/courses/30/lessons/42862)



그리디 알고리즘에 대한 문제를 처음 풀어보았다.

> 이게 1단계라고..?



#### 소스코드

```
public int solution(int n, int[] lost, int[] reserve) {
        int answer = 0;
        int lostLen = lost.length;
        ArrayList<Integer> losts = new ArrayList<>();
        ArrayList<Integer> res = new ArrayList<>();
        for(int a : lost){
            losts.add(a);
        }
        for(int a : reserve){
            res.add(a);
        }
        
        //잃어버린 사람 제외
        answer = n - lostLen;
        
        //도둑질 맞은 경우
        for(int i=0; i<losts.size(); i++){
            for(int j=0; j<res.size(); j++){
                if(res.get(j) == losts.get(i)){
                    res.remove(j);
                    losts.remove(i);
                    i--;
                    answer = answer + 1;
                    break;
                }
            }
        }
        
        //빌려주는 경우
        for(int i=0; i<losts.size(); i++){
            for(int j=0; j<res.size(); j++){
                if(res.get(j) +1 == losts.get(i) || res.get(j) - 1 == losts.get(i)){
                    res.remove(j);
                    answer = answer + 1;
                    break;
                }
            }
        }
        return answer;
    }
```



#### 첫 시도

전체 n 명 중에서 lost와 reserve의 길이 만큼 제외

- lost와 reserve에 대해서만 따로 계산할 생각



빌려줄 수 있는 경우는 +,- 1의 크기 만큼

- 리스트의 중간에 있는 값들을 사용하기 위해 ArrayList 선언

- lost를 순회하면서 reserve의 값 +1 또는 -1 한 값이 lost의 원소 값과 같으면 빌려줄 수 있으므로 +1을 한 후 break



lost에 대한 인원 수를 구했으므로 나머지 reserve 카운트 만큼 ++



예제는 통과하였으나, 실제 채점 결과 처참했다..



#### 두 번째 시도

문제를 다시 읽어보니 `여벌 체육복을 가져온 학생이 체육복을 도난당했을 수 있습니다.`

여벌의 체육복이이 있다고 전부 빌려줄 수 있는게 아니었다!!



나는 처음에 도난을 당하는 경우에는 (최대 1개) reserve의 인덱스 값이 -1로 된다고 생각했다.

그래서 결국 lost의 인덱스 값과 같아지므로  `if(res.get(j) == losts.get(i))`이와 같이 코드를 작성했다.

- 지금 생각하면 왜 이렇게 생각했는지 모르겠다.



하지만 반대로 reserve에서 도둑질을 당하게 된다면 , lost에 목록에 추가가 될테니 list의 목록에서 각각의 원소를 제외시켜주면 된다.



`채점 결과 : 75점`

여기서 더 해줄게 있나..?



#### 세 번째 시도

생각없이 처음 작성 했던 부분에 이어서 코딩을 했던게 원인이었다.

도둑질 맞은 경우 빌려줄 수 있는 경우 자체가 달라지게되는데, 처음 빌려줄 수 있는 경우 부터 계산 후 도둑질을 계산하니 당연히 값이 틀려지는 것이었다.

두 for문의 순서를 바꿔주었다.



`채점 결과 : 95점`

...?? 이제 진짜 모르겠다.



#### 네 번째 시도

lost의 인덱스가 for문에서 i++를 반복하면서 항상 첫 번째 인덱스를 가리키지 못한다.

그래서 도둑질을 맞은 경우 해당 for문에서 `i--` 를 해줘야 했다.



`통과`



1단계여서 가벼운 마음으로 들어왔다가 헤매게되었다.

2단계 문제 부터는 더욱 난관이 예상된다.









