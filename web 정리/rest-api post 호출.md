rest-api post 호출 

3천개 데이터로 1개씩 호출했더니 서버가 부하를 못견디고 connection fail 발생

그래서 연결이 끊어질 때 쉬었다 다시 하는 설정을 했더니 됨



하지만 데이터가 더 많아지면 문제

3천개를 파일에서 읽을 때 부터 끊어서 읽어야하고

읽은 데이터를 100개씩 끊어서 넣어야함



파일 읽기할 때 jvm 모니터링 툴로 메모리 얼마나 치는지 확인



서블릿은 멀티쓰레드 환경인걸 명심





소스

```
package com.fakesinsa.fakesinsaboot;

import org.apache.http.impl.client.DefaultHttpRequestRetryHandler;
import org.apache.http.impl.client.HttpClientBuilder;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.impl.conn.PoolingHttpClientConnectionManager;
import org.apache.http.protocol.HttpContext;
import org.json.JSONObject;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.http.*;
import org.springframework.web.client.RestTemplate;

import javax.imageio.IIOException;
import java.io.*;
import java.util.*;

@SpringBootApplication
public class FakesinsaBootApplication {

    private static final int HTTP_CLIENT_RETRY_COUNT = 3;

    private static final int MAXIMUM_TOTAL_CONNECTION = 10;
    private static final int MAXIMUM_CONNECTION_PER_ROUTE = 5;
    private static final int CONNECTION_VALIDATE_AFTER_INACTIVITY_MS = 10 * 1000;

    public static void main(String[] args) {


        HttpClientBuilder clientBuilder = HttpClients.custom();

        PoolingHttpClientConnectionManager connectionManager = new PoolingHttpClientConnectionManager();

        // Set the maximum number of total open connections.
        connectionManager.setMaxTotal(MAXIMUM_TOTAL_CONNECTION);
        // Set the maximum number of concurrent connections per route, which is 2 by default.
        connectionManager.setDefaultMaxPerRoute(MAXIMUM_CONNECTION_PER_ROUTE);

        connectionManager.setValidateAfterInactivity(CONNECTION_VALIDATE_AFTER_INACTIVITY_MS);

        clientBuilder.setConnectionManager(connectionManager);

        clientBuilder.setRetryHandler(new DefaultHttpRequestRetryHandler(HTTP_CLIENT_RETRY_COUNT, true, new ArrayList<>()) {

            @Override
            public boolean retryRequest(IOException exception, int executionCount, HttpContext context) {
                return super.retryRequest(exception, executionCount, context);
            }

        });
        

        //고객리스트를 담을 배열 선언
        ArrayList<String> userList = new ArrayList<>();
        try{
            File file = new File("C:\\Sample.txt");

            FileReader fileReader = new FileReader(file);

            BufferedReader bufferedReader = new BufferedReader(fileReader);

            String line = "";


            while((line = bufferedReader.readLine()) != null){
                userList.add(line);
            }
            bufferedReader.close();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }catch (IOException ez){
            System.out.println(ez);
        }

        System.out.println(Arrays.toString(new ArrayList[]{userList}));
        System.out.println("사이즈 :"+userList.size());

        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);

        RestTemplate restTemplate = new RestTemplate();

        String url = "https://api-mkt.tason.com/user/userInfoCost";

        JSONObject jsonObject = new JSONObject();

        ArrayList<String> list = new ArrayList<>();
        String answer = "";
        for(int i=0; i<userList.size(); i++) {
            jsonObject.put("user_id", userList.get(i));
            HttpEntity<String> entity = new HttpEntity<String>(jsonObject.toString(),headers);
             answer = restTemplate.postForObject(url, entity, String.class);
            list.add(answer);
        }

        System.out.println(Arrays.toString(new ArrayList[]{list}));


        SpringApplication.run(FakesinsaBootApplication.class, args);
    }
}

```







-----

2탄

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

        String url = "https://api-mkt.tason.com/user/userInfoCost";

        List<String> result = new ArrayList<>();

        Timer timer = new Timer();

        System.out.println(result);
       try{
           //파일을 한 번에 읽어서 리스트에 저장
           list = Files.readAllLines(path, cs);

           int listLength = list.size();

           //리스트를 큐에 저장
           for(int i=0; i<listLength; i++){
               que.add(list.get(i));
           }
           
           for(int i=0; i<que.size(); i++){
               if(i !=0 && i % 100 == 0){
                   try{
                       Thread.sleep(5000);
                   }catch (Exception e){
                       e.printStackTrace();
                   }
               }else {
                   jObj.put("user_id", que.poll());
                   HttpEntity<String> entity = new HttpEntity<String>(jObj.toString(), headers);
                   String answer = restTemplate.postForObject(url, entity, String.class);
                   result.add(answer);
               }
           }


       }catch(IOException e){
           e.printStackTrace();
       }

       
        SpringApplication.run(FakesinsaBootApplication.class, args);
    }
}

```

