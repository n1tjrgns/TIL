## [MSA] API Gateway - (4)

간단한 API Gateway를 구현해봤으니, 이제 인증 서버를 구현할 차례이다.

마찬가지로 멀티모듈 구성으로 진행하려고 한다.



#### build.gradle

```groovy
project(':authorizationserver'){
    dependencies {
        compile group: 'org.springframework.boot', name: 'spring-boot-starter-web', version: '2.2.5.RELEASE'
        compile group: 'org.springframework.cloud', name: 'spring-cloud-starter-security', version: '2.2.4.RELEASE'
        compile group: 'org.springframework.cloud', name: 'spring-cloud-starter-oauth2', version: '2.2.4.RELEASE'
        implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
        compile 'com.h2database:h2'
        compile group: 'org.springframework.boot', name: 'spring-boot-devtools'
        testImplementation 'org.springframework.boot:spring-boot-starter-test'
    }
```



#### application.yml

```yaml
# 인증 서버
server:
  port: 8090

###########
security:
  oauth2:
    client: 
      client-id: n1tjrgns # client를 식별하는 고유 id
      client-secret: tistory # access token을 교환하기 위한 암호

    resource:
      jwt:
        key-value: jwt_secret_key

spring:
  h2:
    console:
      enabled: true
  datasource:
    url: jdbc:h2:tcp://localhost/~/apigateway
    username: sa
    password:
    driver-class-name: org.h2.Driver

  jpa:
    hibernate:
      ddl-auto:  create-drop #create 어플 실행시점에 자동 테이블 리셋모드
    properties:
      hibernate:
        dialect: org.hibernate.dialect.H2Dialect #hibernate.dialect' not set h2 에러 해결
        # show_sql: true #sysout 용
        format_sql: true
```

yml에서 security 관련 부분은 보통 OAuth2 인증을 지원하는 웹 사이트에서 클라이언트 계정을 발급받을 수 있다.

소셜 로그인 연동을 해봤다면 뭔지 알 수 있을것이다. 지금은 단순 예제 이기 때문에 임의로 값을 집어 넣는다.

나머지 설정 부분은 h2 DB와 jpa 관련 설정이다.





```
plugins {
    id 'org.springframework.boot' version '2.2.5.RELEASE' //spring cloud gateway에서 사용하는 ga(Generally Available)
    id 'io.spring.dependency-management' version '1.0.9.RELEASE'
    id 'java'
}



configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}



//전체에 적용하고 싶다면 allprojects를 사용하면 된다.
//내부 모듈에만 적용하고 싶다면 subprojects
//allprojects와 subprojects에는 plugins를 쓸 수 없어 apply plugin을 사용해야 한다.
allprojects {
    apply plugin : 'java'
    apply plugin : 'org.springframework.boot'
    apply plugin : 'io.spring.dependency-management'

    group = 'com.msa'
    version = '0.0.1-SNAPSHOT'
    sourceCompatibility = '1.8'

    repositories {
        mavenCentral()
    }

    dependencies {
        //compile group: 'org.springframework.boot', name: 'spring-boot-starter-web', version: '2.2.5.RELEASE'
        //spring cloud gateway를 위한 라이브러리
        compile group: 'org.springframework.cloud', name: 'spring-cloud-starter-gateway', version: '2.2.5.RELEASE'

        // netflix에서 Circuit Breaker Pattern을 구현한 라이브러리리, 하지만 더 이상 추가기능 제공되지 않고 유지보수만 한다고함
        // 최근에는 resilience4j를 사용
        compile group: 'org.springframework.cloud', name: 'spring-cloud-starter-netflix-hystrix', version: '2.2.5.RELEASE'
        //compile group: 'io.github.resilience4j', name: 'resilience4j-circuitbreaker', version: '1.6.1'
        /*compile("org.springframework.cloud:spring-cloud-starter-contract-stub-runner"){
              exclude group: "org.springframework.boot", module: "spring-boot-starter-web"
          }*/
        // Contract Test를 하기 위한 라이브러리, 서비스 제공장
        //compile group: 'org.springframework.cloud', name: 'spring-cloud-starter-contract-stub-runner', version: '2.2.5.RELEASE'
        compileOnly 'org.projectlombok:lombok'
        annotationProcessor 'org.projectlombok:lombok'
    }

}


project(':client1'){
    dependencies {
        compileOnly 'org.projectlombok:lombok'
        compile group: 'io.projectreactor', name: 'reactor-core', version: '3.3.12.RELEASE'
    }
}

project(':authorizationserver'){
    dependencies {
        //compile group: 'org.springframework.boot', name: 'spring-boot-starter-webflux', version: '2.2.5.RELEASE'
        compile group: 'javax.servlet', name: 'javax.servlet-api', version: '3.1.0'
        compile group: 'org.springframework.cloud', name: 'spring-cloud-starter-oauth2', version: '2.2.4.RELEASE'
        compile group: 'org.springframework.cloud', name: 'spring-cloud-starter-security', version: '2.2.4.RELEASE'
        implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
        compile 'org.projectlombok:lombok:1.18.6'
        compile 'com.h2database:h2'
        compile group: 'org.springframework.boot', name: 'spring-boot-devtools'
        testImplementation 'org.springframework.boot:spring-boot-starter-test'
    }

    /*configurations {
        compile.exclude group: 'spring-boot-starter-web'
        compile.exclude group: 'spring-cloud-starter-gateway'
    }*/
}


test {
    useJUnitPlatform()
}

```

초기 apigateway 를 root플젝으로 해서 의존성을 관리했는데

gateway와 starter boot랑 의존성 충돌나서

gateway도 아래로 분리





```
curl -d "first_name=Bruce&last_name=Wayne&press=%20OK%20" http://posttestserver.com/post.php
```

