```
public String solution(int n) {
    String answer="";
    int mok=n/3+1;
    int nmg=n%3;
    String num124="";
    ArrayList<String> list = new ArrayList<>();
    list.add("1");
    list.add("2");
    list.add("4");

    if(nmg==1){
        num124 = list.get(0);
    }else if(nmg==2){
        num124 = list.get(1);
    }else {
        mok -= 1;
        nmg=3;
        num124 = list.get(2);
    }

    for(int i=0; i<mok; i++){
       
        answer= answer+num124;
        System.out.println("answer : "+ answer);
    }

    return answer;
}
```

```
@Test
public void 숫자123을_결과124로_표현(){
    CountryNumber124 c = new CountryNumber124();
    assertEquals("1",c.solution(1));
    assertEquals("2",c.solution(2));
    assertEquals("4",c.solution(3));
}
```