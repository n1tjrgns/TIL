## Apache JMeter - HTTP, WebSocket 설치 및 성능 테스트



JMeter로 성능 테스트를 시킬 수도 있다는 얘기가 나와서 한번 정리해 놓으려고 한다.





1. ##### JMeter 설치 및 실행

Apache JMeter 사이트에서 [다운로드](http://jmeter.apache.org/download_jmeter.cgi) 받는다. Binaries 아래 zip 파일을 받아주면 된다.

Java는 8버전 이상이 설치되어 있어야 한다.



그 다음 JMeter 여러 플러그인을 설치할 수 있도록 도와주는 JMeter Plugin Manager(jar 파일)를 [다운](https://jmeter-plugins.org/install/Install/) 받는다.

다운 받은 jar 파일은 압축 해제 후 JMeter 폴더인 `apahce-jmeter-5.1.1/lib/ext` 폴더로 옮긴다.

그리고 `apahce-jmeter-5.1.1/bin/jmeter.bat`파일을 클릭해 JMeter를 실행한다.



##### 테스트를 해보자



2. ##### Thread Group 생성

JMeter를 실행하면 기본적으로 Test Plan이 있다. 말 그대로 테스트를 진행할 계획을 작성하면 된다.

추가해야 할 것은 Thread Group이다.

JMeter 테스트는 쓰레드를 생성하여 요청을 하게 되는데, Thread Group은 쓰레드 생성 규칙에 대한 정의를 하는 공간이며, 여러 테스트에 대한 그룹 단위다.

예를들어 HTTP 요청과 WebSocket 요청은 다른 Thread Group을 갖는 것이 좋을 것이다.

그렇지 않으면 HTTP와 WebSocket 요청이 번갈아 실행되어 올바른 테스트 결과를 얻을 수 없다.



왼쪽 카테고리 물약모양의 `TestPlan 우클릭 -> add -> Thread(Users) -> Thread Group` 를 하면 Group이 생성된다.



아래 Thread Properties를 설정해야 한다.

##### 1) Number of Threads (users)

쓰레드 개수, 즉 `가상 유저의 수`



##### 2) Ramp-Up Period (in seconds)

`쓰레드 당 생성시간`을 의미한다.

Number of Threads = 1000이고, Ramp-up = 10 이면, 1000명의 유저를 생성하는데 10초가 걸린다는 것이다.

1초 동안 100명의 유저가 요청을 한다는 뜻이다.

Ramp-up = 0 으로 설정하면, 동시 접속 자 수는 1000명이 될 것이다.



##### 3) Loop Count

`하나의 Thread가 수행할 작업 수` 를 의미한다.

Number of Threads = 1000 이고, Loop Count = 10이면, 1000명의 유저는 동일한 작업을 10번 수행하게 된다.

따라서 총 1000 * 10 = 10000 번 수행되고, `총 요청 횟수`로 생각할 수 있다.



##### 주의할점

속성 값을 정의할 때 쓰레드 수가 지나치게 많을 경우 CPU가 과부하 되어, 올바른 테스트 결과를 얻을 수 없기 때문에 적절하게 값을 조정해야 한다.

테스트를 하면서 작업 과리자를 통해 CPU와 메모리 상태를 확인하는 것이 좋다.



3. ##### HTTP Request 생성

요청에 대한 쓰레드 생성 규칙을 정의 했으니, 실제로 어떤 경로에 요청을 보낼 것인지 정의해야 한다.

이를 작성하는 곳이 HTTP Request이다.



위에서 생성한 `ThreadGroup -> add -> Sampler -> Http Request` 를 클릭해 Http Request를 생성한다.

프로토콜, 도메인 이름 or IP 주소, 포트 번호를 작성하고 HTTP 요청 메소드와 경로를 작성하면 된다.



4. ##### 응답 결과 데이터 조회

요청에 대한 응답  결과를 분석하기 위해 여러 가지 결과 데이터가 필요하다.

무엇을 테스트 할 것이냐에 따라 추가하는 Listner가 다르다.

`ThreadGroup -> add -> Listner -> 원하는 Listner`로 추가해준다.



##### 1) View results Tree

Request, Response data 등 요청에 대한 정보를 볼 수 있다.



##### 2) Summary Report

요청 결과를 파일로 저장할 수 있다.

하단의 Save Table Data 버튼을 클릭하면, 데이터를 바탕으로 엑셀에서 그래프를 그릴 수 있다.



##### 3) Response Time Graph

응답 시간 결과를 그래프로 볼 수 있다. (settings에서 x축 간격을 좁힐 수 있다.)



5. ##### 테스트

이제 상단의 Start 버튼을 누르면 Thread가 생성되면서 요청을 하게 된다.

주의할 점은 start하기 전에, 톱니바퀴에 빗자루 2개가 그려진 버튼을 눌러 이전 테스트 내용을 모두 지운 후에 테스트를 해야 한다.



6. ##### WebSocket Sampler 플러그인 설치

WebSocket 테스트를 위한 WebSocket Sampler 플러그인을 설치하자.

`JMeter 상단 options -> Plugins Manager 클릭 후, 상단의 Available Plugins 탭 클릭 -> WebSocket Sampler by Peter Doornbosch 체크 -> 우측 하단 Apply Changes and Restart JMeter 버튼 클릭`



하지만 나는 해당 JMeter 폴더가 있는 C 드라이브의 관리자 권한 문제로 인해 다운받아지지 않아 직접 jar 파일을 [다운](https://bitbucket.org/pjtr/jmeter-websocket-samplers/downloads/)받아서 위에서 했던 방식과 같이 lib/ext경로에 직접 추가해줬다.



7. ##### WebSocket 테스트

Sampler를 작성해보자.

새로운 Thread Group을 생성한다

`Thread Group -> add -> Sampler -> WebSocket Single write Sampler` 를 클릭해 WebSocket Sampler를 생성한다.



##### 1) Connection

요청할 때 마다 새로운 Connection을 생성하도록 setup new connections을 클릭하고,

프로토콜, 도메인 이름 or IP, 포트번호와 경로를 작성한다.

그리고 Request data에 메세지를 작성한다.



테스트 방식은 마찬가지로 상단의 start 버튼을 누르고, Response Graph를 통해 응답 결과를 확인할 수 있다.



추가적으로

HTTP 성능을 높이기 위해서는 웹 서버의 Worker 조정, 캐시 등의 방법이 있고

WebSocket 성능을 높이기 위해서는 WebSocket 요청을 처리하는 서버를 늘려서 load balancing 하는 방법이 있다.



load balancing 성능을 측정 시 주의할 점은 JMeter에서 제공하는 Response Graph를 확인하는 것이 아니라, 서버의 CPU와 메모리를 확인해야 한다.





##### 참고

<https://victorydntmd.tistory.com/267>