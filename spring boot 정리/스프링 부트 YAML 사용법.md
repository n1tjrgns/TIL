## 스프링 부트 YAML 사용법

설정파일인 application의 형식에는 .Properties와 .yml 두 가지 방식이 있다.

하지만 .yml 방식을 더 권장한다.



이유는 단순하다. 들여쓰기로 구분해줘서 가독성이 좋다.

```yml
  profiles:
    active: local
    
  server:
 	 port: 8080
 		 servlet:
  		  session:
     		 tracking-modes: cookie  
```

또한 리스트로 표현할 때는 " - " (대쉬)로 사용한다.

```yml
mybatis:
  mapper-locations:
  - classpath:mybatis/mapper/**/*.xml
```



뿐만 아니라 profile을 지정하면 환경에 따라 설정값을 다르게 사용할 수 있고, 파일을 여러개 만들지 않더라도 한 페이지에 여러 설정을 적용할 수 있다.

```yml
server:
    address: 192.168.1.100
---
spring:
    profiles: development
server:
    address: 127.0.0.1
---
spring:
    profiles: production & eu-central
server:
    address: 192.168.1.120

```



#### 프로젝트 구조

스프링 부트 프로젝트를 생성하면 기존의 application.properties는 지우고 classpath인 src/main/resources에 application.yml 파일을 생성한다.

스프링부트에서는 자동으로 classpath안에 있는 설정파일을 찾게 되어있다.



spring-boot-starter-parent의 pom.xml을 보면 이와 같은 내용이 있다.

```xml
<build>
        <resources>
            <resource>
                <filtering>true</filtering>
                <directory>${basedir}/src/main/resources</directory>
                <includes>
                    <include>**/application*.yml</include>
                    <include>**/application*.yaml</include>
                    <include>**/application*.properties</include>
                </includes>
            </resource>
            <resource>
                <directory>${basedir}/src/main/resources</directory>
                <excludes>
                    <exclude>**/application*.yml</exclude>
                    <exclude>**/application*.yaml</exclude>
                    <exclude>**/application*.properties</exclude>
                </excludes>
            </resource>
        </resources>
        <pluginManagement>
            <plugins>
```

보다시피 application이 접두사로 붙은 설정파일을 classpath 이하의 어떤 디렉토리에 있든 .yml, .yaml, .properties 순서로 찾아서 읽는다.



#### profile로 환경에 따른 설정파일 세팅하기

```yml
spring :
	profiles :
		active : local # 기본 환경 선택
		
# local 환경
---
spring :
	profiles : local
		datasource :
			data : classPath:data-h2.sql # 시작할 때 실행시킬 script
		jpa:
			show-sql : true
        	hibernate:
        		ddl-auto: create-drop
        h2:
        	console:
        		enabled: true
        		
# 운영 환경
---
spring:
	profiels: set1
server:
	port: 8081
```

위와 같이 spring.profiles.active:local로 하면 profiles가 :local인 설정이 적용되고 set1으로 하면 set1인 설정이 적용된다.

default 환경을 설정할 수 있고, active되는 것마다 바꾸면 되어 개발하고 배포할 때 편하다.



#### 여러 옵션

##### spring.devtools

의존성 추가

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

restart : classpath 파일이 변경되면, 자동으로 다시 시작되도록 설정한다.

​				true는 사용, false는 사용안함

livereload.enabled: 리소스가 변경될 때, 브라우저를 재시작하지 않아도 리소스가 변경되도록 설정한다.

​									true는 사용, false는 사용안함

 

##### spring.jpa.hibernate.naming

```yml
physical-strategy: #카멜 케이스를 underscore 형태로 변경 org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy
implicit-strategy: org.springframework.boot.orm.jpa.hibernate.SpringImplicitNamingStrategy
```

JPA와 MySql을 연동해서 사용할 경우, 테이블 이름이나 칼럼명을 _ 가 아닌 대문자 형식으로 ex) MyTable 작성하는 경우가 있는데 이 경우 Entity의 테이블 이름을 MyTable로 작성하면, Spring JPA에서 자동으로 my_table로 변경해버려서 에러가 발생한다.

그래서 위 두가지 네이밍 전략을 통해 자동 변경을 방지해 줘야 한다.

##### spring.profiles.jpa.hibernate.ddl-auto 

create : 기존 테이블 삭제 후 다시 생성

create-drop : create 와 같으나 종료시점에 테이블 DROP

update : 변경분만 반영

validate : 엔티티와 테이블이 정상 매핑되었는지만 확인

none : 사용하지 않음



##### spring.server.session.tracking-modes

= # Session tracking modes = cookie (one or more of the following: "cookie", "url", "ssl").

(세션 tracking mode)

- 이 설정을 하면 redirect해도 session id가 쿠키로 남아있는 것 같음. 더 찾아 봐야함.



##### mybatis.mapper-locations:

\- classpath:mybatis/mapper/\**/\*.xml

mybatis 설정 파일 경로 잡아주기.



출처: 

https://nuninaya.tistory.com/category

 [1.너니나야 이야기]



참고

<https://jeong-pro.tistory.com/159>







##### 참고

<https://jeong-pro.tistory.com/159>