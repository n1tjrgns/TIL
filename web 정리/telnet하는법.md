telnet하는법



nslookup

set type=mx

gmail.com

set type=a

https://kebimail.tistory.com/42



우선 cmd에서 telnet 명령어를 사용하기 위해서

제어판 => 프로그램 및 기능 => windows 기능 켜기/끄기 => 텔넷 클라이언트를 체크 => 확인 해준다.

이제 telnet을 사용하기 위한 설정이 끝났다.



nslookup을 통해 MX레코드가 잘 설정되었는지 확인 할 수 있다.

nslookup

set type=mx

도메인 주소 입력 ex) naver.com

확인 결과 네이버는 3개의 메일 서버가 있음을 확인 할 수 있다.



이제 보낼 메일 서버로 접속을 한다.

telnet mx1.naver.com

220을 반환하면 정상적으로 접속이 됐다는 소리이다.

helo naver.com -어디서 접속했는지 알려준다.

mail from:<n1tjrgns@naver.com> - 보낸사람

rcpt to:n1tjrgns@naver.com - 받는사람

data - 메일을 작성한다고 알려준다.

subject:test mailSend  

from:seokhun1

to:seokhun2

메일 제목, 보낸사람, 받는사람을 입력한다.

(빈공간) 

.    마침표로 메일 작성을 완료하고

quit 접속을 종료한다.