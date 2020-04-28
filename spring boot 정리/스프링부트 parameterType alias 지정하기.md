## 스프링부트 parameterType alias 지정하기

##### 소스코드

```java
SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSource);

		//아래 두줄 추가
        sqlSessionFactoryBean.setVfs(SpringBootVFS.class);
        sqlSessionFactoryBean.setTypeAliasesPackage
        								("DTO/VO 클래스가 있는 패키지 경로");
```



기존의 Config 클래스에 위의 두줄을 추가해주기만하면 끝이다.

여기서 나오는 VFS는 Virtual File System의 약자로 가상의 파일 시스템을 의미하는 용어인데, 파일 등 시스템 리소스에 접근할 때 이용하는 클래스다.

setVfs안에 아무런 지정을 하지 않으면 DefaultVFS라는 구현체가 사용되는데 IDE 위에서 구동될 때 target 디렉토리 이하 classes 파일들에 문제없이 접근할 수 있으나 Spring Boot 프로젝트로 배포한 jar에서는 형태가 달라 classes에 접근이 되지않아 alias들이 등록되지 않는다.

