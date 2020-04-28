# 가장 큰 수 - TDD로 풀이해보기

- #### 첫번째 테스트 코드 작성

1. solution의 반환값을 0으로 설정, 23과 일치하지 않는다.
2. 테스트코드 실패

```
public class NextBigNumberTest {

    @Test
    public void n보다_큰_자연수() {

        NextBigNumber nb = new NextBigNumber();
        assertEquals(23, nb.solution(15));
    }
}
```

```
public class NextBigNumber {
    public int solution(int n) {

        return 0;
    }
}
```



- #### 첫번째 테스트 코드 통과시키기

```
public class NextBigNumber {
    public int solution(int n) {

        return 23;
    }
}
```

1. solution의 반환값을 23으로 설정
2. 반환값이 일치하기 때문에 테스트 코드는 통과한다.



두번째 TDD Cycle

TDD 사이클은 실패하는 테스트 케이스를 도출하는 것임을 명심한다.

현재 상태에서 실패하는 테스트 케이스 중 가장 간단한 것은?

15를 입력했을 때, 15보다는 크지만 23보다 작은 수 일것이다.



실패하는 테스트 케이스 만들기

입력값이 78인 경우를 테스트하는 경우를 추가한다.



```
@Test
public void n보다_큰_자연수() {

    NextBigNumber nb = new NextBigNumber();
    assertEquals(23, nb.solution(15));
    assertEquals(83, nb.solution(78));
}
```

입력값이 78이므로 78보다 큰 수중에서 하나를 임의로 넣었다. 당연히 실패하는 것을 볼 수 있다.



테스트 통과 시키기

추가된 assert 문을 통과시키기 위해서 production code를 수정했다.

```
public class NextBigNumber {
    public int solution(int n) {
        if(n==15){
            return 23;
        }
        return 83;
    }
}
```



여기서 변수 n은 이름만으로는 어떤 값이 들어올지 전혀 예측할 수 없다.

나는 현재 입력된 값보다 조건에 맞는 가장 큰 수를 찾는 것이 가장 주된 목적이다.

그래서 n보다는 입력되는 값 == inputNumber로 바꿔주어 변수의 용도를 좀 더 명확히 전달하려 한다.

이런식으로 의도를 드러내기 위해 비즈니스 언어를 사용하는 것이 좋다고 한다.

변수를 수정할 때는 refactor -> rename 기능을 사용하자



푸는 방식을 따라하려 했으나, 결국 실패.



처음이라 어떻게 해야하는 지 모르겠어서 결국 방법을 바꿨다.



Test코드를 먼저 작성해야하지만 익숙하지 않아 내가 접근하기 쉬운 방법을 선택.

1. 의사코드 작성
2. 풀이
3. 의사코드 요건대로 테스트 작성
4. 테스트에 맞게 리팩토링하면서 TDD작성

이렇게 풀었다. 위의 모든 삽질과정으로 5시간이 걸렸다.



- 의사코드

```
의사코드 작성
1. 숫자를 입력받음 -> 입력받은 수를 2진수로 변환 -> 비교할 수를 위해 +1을 해줌, ->2진수로 변환
2. 두 수들의 1의 갯수를 셈 -> 갯수가 다르면 +1
3. 갯수가 같으면 그 수가 정답
```

- 첫 풀이

```
public int solution(int inputNumber) {
    int answer=0;
    int count1=0;
    int count2=0;
   /* String checkBinaryResult = checkBinary(inputNumber);
    // String binaryInputNumber = Integer.toBinaryString(inputNumber);
    int count1 = countIntOne(checkBinaryResult);*/
    String binaryInputNumber = Integer.toBinaryString(inputNumber);
    while(true){

        int nextInputNumber = inputNumber + 1;
        String nextBinaryInputNumber = Integer.toBinaryString(nextInputNumber);
       
        int length1 = binaryInputNumber.length();
        int length2 = nextBinaryInputNumber.length();
        
        for(int i=0; i<length1; i++){
            if(binaryInputNumber.charAt(i)=='1'){
                count1++;
            }
        }

        for(int i=0; i<length2; i++){
            if(nextBinaryInputNumber.charAt(i)=='1'){
                count2++;
            }
        }
      
        //1의 개수 같으면 그 수가 정답
        if(count1==count2){
            answer =  nextInputNumber;
            break;

            //아니면 다음 수를 비교하기 위해 +1
        }else{
            inputNumber++;
            // System.out.println("inputNumber : "+ inputNumber);
        }
    }

    return answer;
}
```

일단 성공하게 풀었다.

보다시피 엄청난 중복이,,



내가 테스트를 작성할 부분은 3가지다.

1. 10진수를 2진수로 바꿨을 때 그 값이 제대로 나오는지.
2. 바꾼 2진수의 1의 갯수를 카운트 했을 때 카운트 값이 제대로 나오는지.
3. 카운트한 값을 비교해 같으면 해당 카운트한 2진수의 10진수 값을 리턴.

```
@Test
public void 십진수를_이진수로_변환(){
    NextBigNumber nb = new NextBigNumber();
    assertEquals("1111",nb.checkBinary(15)); //15
    assertEquals("1001110",nb.checkBinary(78));
}
@Test
public void 이진수로_변환된_값의_1의개수_카운트(){
    NextBigNumber nb = new NextBigNumber();
    assertEquals(4,nb.countIntOne("1111")); //15
    assertEquals(1,nb.countIntOne("10000")); //16
    assertEquals(2,nb.countIntOne("10001")); //17
    assertEquals(2,nb.countIntOne("10010")); //18
    assertEquals(3,nb.countIntOne("10011")); //19
    assertEquals(2,nb.countIntOne("10100")); //20
    assertEquals(3,nb.countIntOne("10101")); //21
    assertEquals(3,nb.countIntOne("10110")); //22
    assertEquals(4,nb.countIntOne("10111")); //23

}

@Test
public void 카운트결과가_같으면_그_수를_리턴(){
    NextBigNumber nb = new NextBigNumber();
    assertEquals(23,nb.solution(15));
    assertEquals(83,nb.solution(78));
}
```





하나하나의 테스트를 통과하기 위해 리팩토링을 진행했다.

10진수를 2진수로 바꿔주는 Integer.toBinaryString이 겹친다. -> 메소드로 빼준다(중복제거)

```
public String checkBinary(int inputNumber) {
    return Integer.toBinaryString(inputNumber);
}
```



for문 2개가 중복되므로 1의 갯수를 세는 메소드를 하나로 합쳤다.(중복제거)

```
public int countIntOne(String binaryNumbers){
    int count=0;
    int length = binaryNumbers.length();
    for(int i=0; i<length; i++){
        if(binaryNumbers.charAt(i)=='1'){
            count++;
        }
    }
    return count;
}
```



이상으로 내가 생각하기에 할 수 있는 만큼 했다고 생각한다...

더 할 수 있으면 피드백좀 부탁드립니다.



#### 느낀점

그냥 2번의 코드를 풀었을 때는 30분도 채 안걸렸는데,,

삽질하다 다시 리팩토링하는데 엄청난 시간이 걸렸다.

하루아침에 되는게 아닌것을 느꼈다. 꾸준함이 필요하다.



