## [프로그래머스] level2 - 가장 큰 수

```
import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;
class Solution {
    public String solution(int[] numbers){
            String answer ="";
            List<String> list = new ArrayList<>();
            int length = numbers.length;

            for(int i=0; i<length; i++){
                list.add(Integer.toString(numbers[i]));
            }

            int size = list.size();
            
            Collections.sort(list, new Comparator<String>(){
                @Override
                public int compare(String num1,String num2){
                    return (num2+num1).compareTo(num1+num2);
                }
            });
            
            if(list.get(0).equals("0")){
                return "0";
            }

            for(int i=0; i<size; i++){
                answer = answer + list.get(i);
            }
            return answer;
        }
}
```

