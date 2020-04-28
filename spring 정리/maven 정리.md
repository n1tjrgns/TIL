##### maven

- 자바 프로젝트의 빌드(build)를 자동화 해주는 빌드 툴
- 자바 소스를 compile하고 package 해서 deploy하는 일을 자동화 해준다.



##### Maven이 참조하는 설정 파일

##### 1) settings.xml

- maven tool 자체에 관련된 설정을 담당
- MAVEN_HOME/conf/ 아래에 있다. MAVEN_HOME은 환경변수에 설정한 경로



2) pom.xml

- 자바 프로젝트 빌드 툴로 maven을 설정했다면, 프로젝트 최상위 디렉토리에 "pom.xml" 파일이 생성된다.
- POM(Project Object Model)을 설정하는 부분으로 프로젝트 내 빌드 옵션을 설정하는 부분이다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
 
    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>
 
    <name>demo</name>
    <description>Demo project for Spring Boot</description>
 
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.5.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
 
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>
 
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
 
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
 
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>



```

프로젝트 생성시 나오는 pom.xml

-----

##### 1) 프로젝트 정보

<project>...</project> 로 둘러쌓여 있고 section 별로 여러 정보를 나타내며 설정할 수 있다.

<modelVersion> : maven의 pom.xml의 모델 버전

<groupId> : 프로젝트를 생성한 조직, 그룹명

<artifactId> : 프로젝트에서 생성되는 기본 아티팩트의 고유 이름

<version> : 어플리케이션의 버전. 접미사로 SNAPSHOT이 붙으면 아직 개발단계라는 의미

<packaging> : jar, war, ear, pom 등의 패키지 유형



<name> : 프로젝트 명

<url> : 프로젝트를 찾을 수 있는 URL



<properties> : pom.xml에서 중복해서 사용되는 설정(상수) 값을 지정하는 부분.

지정시 다른 위치에서 ${...}로 표기해서 사용할 수 있다.

```xml
ex) <java.version>1.8</java.version> => ${java.version}
```



<profiels> : dev, prod 이런식으로 개발할 때, 릴리즈할 때를 나눠야할 필요가 있는 설정 값은 profiles로 설정 가능.

```xml
<profiles>
  <profile>
   <id>dev</id>
   <properties>
    <java.version>1.8</java.version>
   </properties>
  </profile>
  <profile>
   <id>prod</id>
   <properties>
    <java.version>1.9</java.version>
   </properties>
  </profile>
</profiles>


```

maven goal 부분에 -p 옵션으로 프로파일을 선택할 수 있다.

```xml
ex) mvn compile -P prod
```

------

##### 2) 의존성 라이브러리 정보

의존성 라이브러리 정보를 적을 수 있다.

최소한 groupId, artifactId, version 정보가 필요하다.



스프링부트의 spring-boot-starter-* 가타은 경우 부모 pom.xml에서 이미 버전정보가 있어서 version은 따로 지정할 필요가 없다.

오버라이드해서 문제가 생기는 부분은 개발자 탓이다.



또한 A라는 라이브러리를 사용하는데 B,C,D가 의존성을 가진다면 A를 dependency에 추가하면 자동으로 필요한 B,C,D도 가져오는 기능이 있다.

dependency에 <scope>의 경우 compile, runtime, provided, test 등이 올 수 있는데 해당 라이브러리가 언제 필요한지, 언제 제외되는지를 나타낸다.

-----

##### 3) build 정보

build tool : maven의 핵심인 빌드와 관련된 정보를 설정하는 곳이다.

<build> 부분에서 설정할 수 있는 값들엘 대해 알기 전에 "라이프 사이클"에 대해 알아야 한다.

maven에도 라이프 사이클이 존재한다.

크게 default, clean, site 라이프 사이클로 나누고 세부적으로 페이즈(phase)가 있다.

![1555894576947](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1555894576947.png)

메이븐의 모든 기능은 플러그인을 기반으로 동작한다.



플러그인에서 실행할 수 있는 각각의 작업을 골(goal)이라고 하고 하나의 페이즈는 하나의 골과 연결되며, 하나의 플러그인에는 여러 개의 골이 있을 수 있다.



**라이프 사이클**

mvn process-resources : resources   :  resources의 실행으로 resource 디렉토리에 있는 내용을 target/classes로 복사한다.

mvn compile : compiler    :  compile의 실행으로 src/java 밑의 모든 자바소스를 컴파일해서 target/classes로 복사



mvn install : 로컬 저장소로 배포

mvn deploy : 원격 저장소로 배포

mvn clean : 빌드 과정에서 생긴 target 디렉토리 내용 삭제

mvn site : target/site에 문서 사이트 생성

mvn site-deploy : 문서 사이트를 서버로 배포



위와같은 순서로 라이프 사이클이 진행된다.



**<build>의 설정 값**

<finalName> : 빌드 결과물(ex.jar) 이름 설정

<resources> : 리소스(각종 설정 파일)의 위치 지정

 - <resouce> : 없으면 기본으로 "src/main/resources"

<testResources> : 테스트 리소스의 위치를 지정할 수 있다.

- <testResource> : 없으면 기본으로 "src/test/resources"

<Repositories> : 빌드할 때 접근할 저장소의 위치를 지정할 수 있다. 기본적으로 메이븐 중앙 저장소로 지정

<outputDirectory> : 컴파일한 결과물 위치 값 지정, 기본 "target/classes"

<testOutputDirectory> : 테스트 소스를 컴파일한 결과물 위치 값 지정, 기본 "target/test-classes"

<plugin> : 어떠한 액션 하나를 담당하는 것





##### <parent> pom.xml 상속

<Parent> : pom.xml은 상속을 받을 수 있다. 스프링 부트의 경우 붐 pom.xml에 자주 사용하는 라이브러리들의 버전 정보나 dependency들을 이미 가지고 있어 참조하기 편하다.