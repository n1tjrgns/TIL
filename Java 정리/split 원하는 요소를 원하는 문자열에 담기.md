# split 세부적으로 사용하기

```java
public class mainClass {
         
    public static void main(String[] args) {
        String str = "IT,개발의,세상";
 
        String val1 = str.split(",")[0];
        String val2 = str.split(",")[1];
        String val3 = str.split(",")[2];
         
        System.out.println("1 :" + val1);
        System.out.println("2 :" + val2);
        System.out.println("3 :" + val3);
 
 
    }
}

1 = IT
2 = 개발의
3 = 세상


```

