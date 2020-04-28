# 스프링 BeanFactory 컨테이너

##### 특징

- 스프링 IoC 컨테이너의 일종

- 의존성 주입을 지원하는 컨테이너
- 인스턴스를 생성하고 설정하고  bean들을 관리하는 컨테이너
- 빈의 제어(생성,관계설정 등)를 담당하는 IoC 오브젝트
- org.springframework.beans.factory.BeanFactory 인터페이스로 정의됨

```
Resource res = new FileSystemResource("beans.xml");
XmlBeanFactory factory = new XmlBeanFactory(res);

ClassPathResource res = new ClassPathResource("beans.xml");
XmlBeanFactory factory = new XmlBeanFactory(res);

ClassPathXmlApplicationContext appContext = new ClassPathXmlApplicationContext(
        new String[] {"applicationContext.xml", "applicationContext-part2.xml"});
// of course, an ApplicationContext is just a BeanFactory
BeanFactory factory = (BeanFactory) appContext;
```