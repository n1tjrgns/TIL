```
package programmers;

/*
*  a = 97
*  A = 65
*  0 = 48
*  null = 0
* */
public class Solution3 {
    public Object getStrToInt(String str) {
        boolean Sign = true;
        int result = 0;
        if(str.contains(".")){
            return null;
        }
        for (int i = 0; i < str.length(); i++) {
            char ch = str.charAt(i);
            if (ch == '-') {
                Sign = false;
            } else{
                result = (result * 10) + (ch - '0');
            }
        }
        return Sign ? result :-1 * result;
    }
    //아래는 테스트로 출력해 보기 위한 코드입니다.
    public static void main(String args[]) {
        Solution3 strToInt = new Solution3();
        //String step[] = {"123","123.456","-123"};

        System.out.println(strToInt.getStrToInt("123"));
        System.out.println(strToInt.getStrToInt("123.456"));
        System.out.println(strToInt.getStrToInt("-123"));
    }
}
```



흔히 String을 Int형으로 변환할 때 `Integer.parseInt()` 를 많이 사용한다.

하지만 Integer.parseInt를 사용할 수 없다면?



문자를 아스키코드로 계산을 해야한다.



숫자 : `해당 숫자의 문자 -'0'`

소문자 : `해당 소문자 - 'a'`

대문자 : `해당 대문자 - 'A'`



방식을 사용해야하며, 각각 1의자리, 10자리, 100의 자리 연산을 위해 10씩 곱하여 더해준다.

값이 -일 경우 flag를 사용해 똑같이 연산 후 마지막에 -1을 곱하여 - 값으로 만들어준다.

