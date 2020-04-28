oracle 프로시저로 값 넣기

```
create procedure dataInsert
IS 
i number := 1; 
begin
loop
   
Insert into POSTMAN.TEMPLATE_ALIM (SEQ,USER_ID,TEMP_NAME,TEM_KEY,COMPANY_NAME,TEM_ALIM_CONTENT,TEM_POST_CONTENT,CUSTOM_GROUP,BTN_TYPE,BTN_TITLE,URL_TITLE,REGISTER_D,APPROVAL_CONDITION,APPROVAL_REGISTER,TEM_TYPE,PRO_KEY,YELLOW_ID,PRO_NAME,NOTE,R_RETURN,URL_TITLE2,BTN_KIND,MODIFY_D) values (TEMPLATE_AILM_SEQ.nextval,'dlrudtn108','휴머스온가입축하',concat('14688',i),'이진우','#{고객이름}님 휴머스온입니다.
가입축하드립니다.','${EMS_M_NAME}님 휴머스온입니다.
가입축하드립니다.','그룹','F','휴머스온홈','http://www.humuson.com',to_date('16/10/04','RR/MM/DD'),'Y',to_date('18/09/27','RR/MM/DD'),'I','146','@포스트맨','67688','67599','44',null,null,null);
       i:=i+1;
       exit when i > 100;
end loop;
       
end;

call dataInsert(); 

drop procedure dataInsert;

```

시퀀스 자동 증가를 위해

TEMPLATE_AILM_SEQ.nextval 사용