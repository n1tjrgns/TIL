# 초기화할때 Throwable이 뛰쳐나왔을때 처리하는 로직을 구현한다. 

##### 문제 

Exception을 절대로 반환하면 안된다. 그러면 ㅠㅠ

     * 에러처리는 구현안에 모두 포함하여 하는 것을 권장한다.


-----

##### 답변

throwable을 함수내부에서 try catch처리를 하라는 의미는, 

디버깅시 코드역추적에 대한 부담이 줄어들어서입니다. 단지 그것뿐. 권장사항일 뿐이에요. 

경우에따라 throwable에 관계없이 동작해야하는 전처리/후처리 함수들도 분명히 존재하니 권장일뿐.



-----

##### 풀이

뭔가 처리가 잘못되서 예외가 발생했는데도
catch로 처리를 해버리면
Http Status 는 200 OK가 떨어질텐데
이러면 클라이언트 측에서는 내 요청이 잘 됬는지 잘못 됬는지 알 수가 없지



그래서 API 서버 같은 경우는 예외를 보통은 처리 안하고 (경우에 따라 처리하기도 하고)



걍 예외를 발생시켜셔 메세지정도만 커스터마이징 한담에
클라이언트로 던짐
그럼 클라이언트가 예외처리 하는거지
사용자에게는 예외가 노출되면 안됨



로그의 목적도 있지
다 catch 해버리면
명시적으로 로깅 안하는이상
로그가 안잡힐테니까
이게 골때림 ㅋㅋ 큰 서비스는 보통 로그 모니터링 시스템이 있는데
예외처리 다해버리면
모니터링 시스템에 안잡혀서



뭐가 잘못되도 소스 까보기 전엔 알수가 없음
결국 책임분배하는거임
이 예외는
어디서 핸들링해야한다~ 이런거