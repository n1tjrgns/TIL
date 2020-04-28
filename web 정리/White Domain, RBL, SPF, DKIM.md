## White Domain, RBL, SPF, DKIM

> White Domain 

- 정상적으로 발송하는 대량 이메일이 RBL 로 간주되어 차단되는 것을 방지하기 위해, 사전에 등록된 개인이나 사업자에 한하여 국내 주요 포털사이트로의 임일 전송을 보장해주는 제도(무료)
  - 단, 이후 모니터링을 통해 RBL 이력발송 사실이 확인되면 즉각 차단조치 또는 white 리스트에서 삭제될 수 있음
    
    

>  RBL (Real-time Blocking List)

- 이메일 수신시 간편하게 스팸여부를 차단할 수 있도록 제공되는 스팸발송에 이용되는 IP 리스트를 말하며, 대체로 DNS Lookup을 통해 확인하는 방식을 이용하므로 DNSBL(DNS-based Blackhole List) 라고도 함.



> 메일서버등록제 SPF(Sender Policy Framework) 

- 메일서버 정보를 사전에 DNS에 공개 등록함으로써 수신자로 하여금 이메일에 표시된 발송자 정보가 실제 메일 서버의 정보와 일치하는지를 확인할 수 있도록 하는 인증기술



**SPF 인증절차**

발신자 : 자신의 메일서버 정보와 정책을 나타내는 SPF 레코드를 해당 DNS에 등록

수신자 : 이메일 수신시 발송자의 DNS에 등록된 SPF 레코드를 확인해 해당 이메일에 표시된 발송 IP와 대조하고 그 결과값에 따라 수신여부를 결정 ( 메일서버나 스팸차단 솔루션에 SPF 확인기능이 설치되어 있어야 함)



> DKIM(Domainkeys Identified Mail)

DKIM은 새 도메인 이름 식별자를 메시지에 첨부하고 암호화 기술을 사용해 메시지 존재 여부에 대한 유효성을 검사한다. 식별자는 작성자의 From : 필드와 같이 메시지의 다른 식별자와 독립적이다.



gmail의 경우 spf와 dkim 이 모두 있어야 메일이 정상적으로 전송된다.



**참고**

[https://spam.kisa.or.kr](https://spam.kisa.or.kr/)

<https://support.google.com/a/answer/174124?hl=ko>

<http://www.dkim.org/>