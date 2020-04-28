(1)탄 포스팅에서 작성한 소스코드를 살짝 바꿔봤다.



#### 기존 소스 코드

- 파일을 한 줄씩 읽은 리스트에 한 줄씩 저장
- 리스트에서 user_id를 한 개씩 꺼내서 post 방식으로 전송



#### 개선 소스 코드

- 파일을 한 번에 읽어서 리스트에 저장
- 추가, 삭제에 좋은 큐에 담아 100개씩 꺼내면서 5초 휴식



데이터가 엄청 많은 경우 100줄 정도 씩 끊어서 큐에 저장하여

api 호출도 100개씩 5초 간격으로 하려고 했으나, api가 그렇게 구현되어 있지않아서 하지 못했다.



api를 마음대로 수정할 수 있는 권한이 있었다면 수정도 해봤을텐데 아쉽다.



```
package com.fakesinsa.fakesinsaboot;

import jdk.nashorn.internal.parser.JSONParser;
import org.json.JSONObject;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.web.client.RestTemplate;

import java.io.*;
import java.nio.charset.Charset;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.*;

@SpringBootApplication
public class FakesinsaBootApplication {
    
	private static final int HTTP_CLIENT_RETRY_COUNT = 3;

    private static final int MAXIMUM_TOTAL_CONNECTION = 10;
    private static final int MAXIMUM_CONNECTION_PER_ROUTE = 5;
    private static final int CONNECTION_VALIDATE_AFTER_INACTIVITY_MS = 10 * 1000;

    public static void main(String[] args) {

       Path path = Paths.get("C:\\Sample.txt");

       Charset cs = StandardCharsets.UTF_8;

        List<String> list = new ArrayList<>();
        Queue<String> que = new LinkedList<>();
        JSONObject jObj = new JSONObject();

        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);

        RestTemplate restTemplate = new RestTemplate();

        String url = "api url";

        List<String> result = new ArrayList<>();

        Timer timer = new Timer();

       try{
           //파일을 한 번에 읽어서 리스트에 저장
           list = Files.readAllLines(path, cs);

           int listLength = list.size();

           //리스트를 큐에 저장
           for(int i=0; i<listLength; i++){
               que.add(list.get(i));
           }
           
           for(int i=0; i<que.size(); i++){
           		//100개를 꺼내면 5초 sleep
               if(i !=0 && i % 100 == 0){
                   try{
                       Thread.sleep(5000);
                   }catch (Exception e){
                       e.printStackTrace();
                   }
               }else {
                   jObj.put("user_id", que.poll());
                   HttpEntity<String> entity = new HttpEntity<String>																(jObj.toString(), headers);
                   String answer = restTemplate.postForObject(url, entity, 																		String.class);
                   result.add(answer);
               }
           }


       }catch(IOException e){
           e.printStackTrace();
       }
    }
}

```



이제 몰랐던 메소드 정리를 해야겠다.

끝.

  













