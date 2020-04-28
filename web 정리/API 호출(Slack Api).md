## API 호출(Slack Api) 

처음에는 api를 호출 할 때에는 ajax를 사용하면 CORS라는 정책에 막혀 이 방식으로 사용하면 안되는 줄 알았다.

하지만 CORS에 대해 알아봤더니, `내가 못하는 거였다 ㅎㅎ.` 맞는 요청헤더를 지정해 줬으면 됐을텐데,,

다음번에 또 개발할 일이 생긴다면 Ajax로 도전해봐야겠다.



아래 코드는 홈페이지에서 Q&A 문의가 들어왔을 때, 해당 문의의 정보를 받아 회사 업무 메신저인 slack에 알람을 주는 용도로 사용하기 위해 slack api를 사용하게 되었다.



##### 작성 코드

```java
private static String sendPost(ArrayList<Properties> qnaUserList) {
   //0 - content , 1 - reg_dt, 3 - reg_id, 4 - title

   String slackContent = qnaUserList.get(0).getProperty("CONTENT","");
   String slackSeq = qnaUserList.get(0).getProperty("MAX(TO_NUMBER(SEQ))","");
   String slackRegId = qnaUserList.get(0).getProperty("REG_ID","");

       String input = null;
   StringBuffer outResult = new StringBuffer();
   try {
      
      //URL 클래스 생성
      URL url = new URL("발급받은 api url");
      //url클래스의 openConnection()메소드로 HttpURLConnection 접속객체를 생성해 
      // 주어진 URL에 접속 
      HttpURLConnection con = (HttpURLConnection) url.openConnection();
      con.setDoOutput(true); //쓰기모드 지정, DoInput은 읽기모드
      con.setRequestMethod("POST"); //POST 방식으로 요청한다. default=GET
      
      //요청 헤더 정의 
      con.setRequestProperty("Content-Type", "application/json");
      con.setRequestProperty("Accept-Charset", "UTF-8");
      
      //타임 아웃 정의
      con.setConnectTimeout(10000);
      con.setReadTimeout(10000);
	  
      //slack에 요청 할 데이터
      String param ="{\n" +
            "    \"attachments\": [\n" +
            "        {\n" +
            "            \"fallback\": \"문의가 접수되었습니다.\",\n" +
            "            \"pretext\": \""+slackRegId+"님의 문의가 접수되었습니다.\",\n" +
            "            \"title\": \""+slackContent+"\",\n" +
            "            \"title_link\": \"https:해당사이트주소?															seq="+slackSeq+"\",\n" +
            "            \"text\": \"답변을 작성해 주세요.\",\n" +
            "            \"color\": \"#7CD197\"\n" +
            "        }\n" +
            "    ]\n" +
            "}";
	  
      //새로운 OuputStream에 요청할 OutputStream을 넣는다. 
      OutputStream os = con.getOutputStream();
 	  
      //write 메소드로 작성된 파라미터 정보를 byte 단위로 인코딩해 요청
      os.write(param.getBytes("UTF-8"));
       //스트림 버퍼 비우기
      os.flush();
		
       
      if(con.getResponseCode() == HttpURLConnection.HTTP_OK) {
		  //응답받은 메시지를 버퍼를 생성해 읽어들이고, "UTF-8"로 디코딩해서 읽음.       
          BufferedReader in = new BufferedReader(new 										InputStreamReader(con.getInputStream(), "UTF-8"));
                
				//한 라인씩 출력          
          		while ((input = in.readLine()) != null) {
                    outResult.append(input);
                }
      }else{
        System.out.println(con.getResponseMessage());
      } 
      
      //연결종료
      con.disconnect();

   }catch(Exception e){
      e.printStackTrace();
   }
   return outResult.toString();
}
```



#### 소스코드 분석

메소드의 qnaUserList는 Q&A 문의를 한 고객들에 대한 정보를 select해서 넣어두었다.



##### [HttpURLConnection 클래스](https://docs.oracle.com/javase/8/docs/api/java/net/HttpURLConnection.html)

- URLConnection을 구현한 클래스로 http URL을 처리할 때 도움이 되는 몇 가지 추가적인 메소드를 가지고 있다.
- URL 클래스와 URLConnection 클래스를 사용할 때는 각각 MalformedURLException과 IOException을 처리해줘야 한다.



##### 참고

<https://digitalbourgeois.tistory.com/57>