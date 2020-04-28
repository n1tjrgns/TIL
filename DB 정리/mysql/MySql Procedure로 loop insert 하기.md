# MySql Procedure로 loop insert 하기

```mysql
delimiter $$
create procedure dataInsert() //함수 생성
begin //시작
	declare i int default 1; //i값 생성
    
    while(i<10000) do //while문 
		insert into teacher (name) value (concat('name',i));
        set i=i+1;
        end while; //while문 끝
        
 end $$
 /////
 call dataInsert(); //함수 호출
 
```

