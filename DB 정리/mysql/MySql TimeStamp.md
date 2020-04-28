

MySql에서 timestamp 를 default 값으로 현재 시간을 지정하거나, 임의로 값을 지정할 때

reg_date TimeStamp defalut '0000-00-00 00:00:00'



내가 사용하고 있는 8.0 최신버전에서는 default 아래를  

CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP 라고 사용한다.