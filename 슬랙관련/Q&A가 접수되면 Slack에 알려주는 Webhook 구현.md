## Q&A가 접수되면 Slack에 알려주는 Webhook 구현

회사에서 업무의 효율성을 위해 고객들의 문의를 Slack으로 알려줘 쉽게 확인할 수 있으면 좋겠다는 얘기가 나왔고, 내가 맡게 되었다.



이러한 방법을 webhook이라고 한다.

> 우선 webhook이 뭔지부터 알아야겠다;;



#### Webhook이란?

Webhook(웹훅)이란, 서버에서 어떠한 작업이 수행 되었을 때 해당 작업이 수행되었음을 `HTTP POST`로 알리는 개념을 말한다. Webhook을 구현한 웹 애플리케이션은, 특정 작업이 수행될 때 `URL`에 대해 `POST`방식으로 요청을 생성한다. 이 때, url(콜백 url)은 웹 애플리케이션을 사용하는 유저가 자신의 `URL`을 지정할 수 있다.

유저의 입장에서는 지속적으로 데이터를 폴링(polling)하여 대부분의 경우 불필요한 정보를 받는 대신, webhook을 활용하여 중요한 이벤트가 발생했을 때에만 정보를 수신할 수 있다. 이를 활용하여 유저의 커스텀 기능이나 다른 애플리케이션과 통합하거나 기능을 확장할 수 있다.



#### 슬랙 웹 훅 URL 발급받기

1. 웹훅으로 뿌려줄 채널을 선택한다. 
2. 톱니바퀴 모양의 설정버튼 -> Add an app -> Incoming WebHooks 를 선택해서 추가
3. 추가하고 나면 웹훅 URL이 생성되는데 기억해 놓자.

```
https://hooks.slack.com/services/~~~~ 이런방식으로 생성된다
```



##### 터미널에서 테스트 하는 방법

```
curl -s -d "payload={\"text\":\"hi\"} "https://hooks.slack.com/services/~~~"
```

 내거

```
https://hooks.slack.com/services/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
```



##### 보내는 방식

- payload 파라미터는 JSON string으로 보내야 한다.

- method는 POST로 전송해야 한다.

```
var payload = { 
// bot 이름을 바꿀 수 있다. 
"username" : "", 
// bot 아이콘을 바꿀 수 있다. 
"icon_url" : "",
// bot 아이콘을 이모티콘으로 사용할 수 있다. 위의 icon_url 중 하나만 사용하면 된다. 
"icon_emoji" : "", 
// 본문 내용을 입력한다. (필수) 
"text" : "", 
// 채널을 override 시킬 수 있다. 
"channel" : "" 
}

```





#### Slack message template 

username : 웹훅 앱 이름 변경

icon_emoji : 웹훅 앱 아이콘 변경 -> 이모지에 있는 것들

애초에 웹훅을 만들 때 바로 페이지 하단에서

Customize Name, Customize Icon으로 바꿀 수 있음.



- 구현한 뒤에 바꾸려 했지만 바뀌지 않았다. 처음에 생성할 때 바꾸자.

```
\"username\": \"webhookbot\", 
\"icon_emoji\": \":ghost:\"}" 
```



아래 데이터를 어떻게 넣냐에 따라 슬랙 메세지가 바뀐다.

[공식문서](https://api.slack.com/docs/message-attachments) 를 참고해 마음에 드는 템플릿을 고르면 된다.

```json
{
    "attachments": [
        {
            "fallback": "New ticket from Andrea Lee - Ticket #1943: Can't reset my password - https://groove.hq/path/to/ticket/1943",
            "pretext": "New ticket from Andrea Lee",
            "title": "Ticket #1943: Can't reset my password",
            "title_link": "https://groove.hq/path/to/ticket/1943",
            "text": "Help! I tried to reset my password but nothing happened!",
            "color": "#7CD197"
        }
    ]
};
```



![1558426293848](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1558426293848.png)

포스트맨 테스트 결과 날라온 메세지

> 오오.. 재밌어



메세지가 날라오니깐 위 코드로 개발하면 되겠다고 생각했다.

하지만 나는 api를 호출해 본 적이 없다.



post방식으로 값을 전달하라그래서 '이렇게 해도 되겠지' 하고 단순하게

type, url, data에 위 코드를 넣고, contentType:json으로 해서 ajax 방식으로 값을 보내봤다.

그런데 자꾸 아래와 같은 에러가 발생했다. 

해당 오류에 대해서는 따로 [정리](https://n1tjrgns.tistory.com/196)를 했다.

![1558426866361](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1558426866361.png)



그래서 구글에 `api 호출` 을 검색했더니 전혀 다른 방식으로 호출을 하고 있었다.



검색해서 찾은 api 호출 방식대로 동작 시켰더니 맨 처음 postman으로 값을 보냈을 때 처럼 메시지가 성공적으로 왔다..!



원래 계획은 Q&A가 접수되면 스케줄러가 돌다가 접수된 Q&A를 읽어서 메시지를 보내주는 것이었지만, 해당 레거시 프로젝트가 JSP페이지에 모든 코드가 들어가있는 `초 울트라 레거시 프로젝트`라서 스프링 배치는 사용해보지 못했다(아쉽 ㅠㅠ).

그래서 방식 또한 문의하기 버튼을 눌러 DB에 insert 될 때, slack api를 호출해서 insert한 데이터를 전달하기로 했다.

insert한 데이터를 select해서 api 호출 코드에 필요한 데이터만 넣어준다.

```java
private static String sendPost(ArrayList<Properties> qnaUserList) {
   //qnaUserList{content,reg_dt,reg_id,MAX(TO_NUMBER(SEQ))}

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



##### 결과

![1558427909768](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1558427909768.png)

테스트를 누르면 해당 글의 시퀀스를 태워보냈기 때문에 바로 Q&A글을 확인할 수 있다.

 



추가요건

incoming-webhook APP [5:58 PM]
| 문의접수
| 접수내용

접수내용이 100자(200bytes)를 넘으면 100자 + "..." 로 표현하면 좋을 것 같습니다.

```
if(slackContent.getBytes().length > 200) {
			slackContent = new String(slackContent.getBytes(),0,200)+"...";
		}
		
		한글이 깨지면 
		new String(slackContent.getBytes("utf-8");
```



기왕 해주시는김에 혹시 링크기능이 있으면 문의상세화면으로 바로 연결해주시면 대단히 감사하겠습니다ㅋ
http://www1.postman.co.kr:13080/ems_admin/08_consult/consult_dtl_new.jsp?flag=mod&seq=49606





##### 참고

[슬랙 webhook API 문서](https://api.slack.com/incoming-webhooks)

[슬랙 webhook 메시지 템플릿](https://api.slack.com/docs/message-attachments)






