Gradle

진화된 빌드툴로 빌드, 테스트, 배포, 개발 등을 자동화 할 수 있다.

Ant와 Maven의 장점을 모두 가짐.



gradle을 통해 실행되는 단위를 "task"라고 한다.

task는 의존 관계에 따라 단 한 번만 실행된다.



##### 참고

<https://www.slideshare.net/sup2rior/gradle-guide-v-01-31412469>





#### 앵귤러 (프레임워크)

```html
<html ng-app>
	...
</html>
```

해당 태그안에 모든 것이 앵귤러 어플리케이션으로 처리됨.



```html
<input type="text" ng-model="x">
```

데이터 바인딩 방법. ng-model 지시자는 input 요소와 바인딩 된다.



```html
<p> {{x}} </p>
```

input 태그에 값이 입력되면 p 태그에 자동으로 업데이트 된다.

ng-model 디렉티브로 input이 양방향 바인딩(two-way binding)되었다.



input 태그에 입력한 값들은 $scope에 저장되어 있다.

input 태그에 값을 입력할 때마다 스코프의 해당 객체가 갱신된다.



$scope는 기본적으로 컨트롤러와 템플릿을 연결하고 모델을 보강해서 양방향 바인딩을 할 수 있게하는 객체다.



![1559009334783](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1559009334783.png)



##### 주요 개념

**Model**

- 단순 자바스크립트 객체로 된 데이터 및 데이터 구조
- 변형되지 않은 단순 자바스크립트 객체를 그대로 사용

**View**

- 템플릿과 모델이 합쳐져서 화면에 보여지는 DOM
- 템플릿이 HTML이거서 바로 DOM으로 해석됨
- DOM안에 directive가 템플릿 엔진인 $compile 지시어를 통해 $watch를 설정하고 모델의 변경을 게속 감시하게됨

**Controller**

- 자바스크립트로된 제어 로직
- 모델을 생성하고 메소드를 가지고 View로 퍼블리싱을 담당

**Scope**

- 특정 DOM 영역을 위한 모델
- 모델과 뷰를 감시하고, 반영하고, 컨틀로러에 이벤트를 보내는 역할
- DOM과 가까운 계층구조를 갖고있음

**Template**

- html 자체를 템플릿으로 사용
- 지시어,표현식,필터등으로 템플릿 제어

**Directives**

- HTML 확장하는 앵귤러JS만의 지시어
- 예 : {{}},ng-app,ng-controller 등
- 사용자 정의를 할 수 있음

**Expressions**

- 해당 Scope로부터 함수나 변수에 접근하는 표현식으로 템플릿에서 사용
- 반복 및 조건 관련 표현식은 존재하지않고, 지시어로 존재함

**Filter**

- 표현식이 화면에 표기되는 포맷
- {{표현식 | 필터 }}
- 사용자 정의를 할 수 있음

**Data Binding**

- 모델과 뷰의 데이터를 실시간 연동
- 양방향 데이터 바인딩

**Module**

- 컨트롤러,서비스,필터,지시어등을 포함하여 응용프로그램의 서로 다른 기능을 구성하는 컨테이너
- 모든 자바스크립트 기능들이 ng-app"첫 모듈명"을 시작으로 모듈 단위로 관리됨

**Service**

- 특정 기능을 담당하는 객체들
- 공통된 특정 작업을 수행하는 Singleton객체
- Singleton은 인스턴스가 1개만 존재하는 디자인 패턴

**Injector**

- Dependency injection을 담는 컨테이너 역할

**Compiler**

- 템플릿에 앵귤러JS만의 지시어나 표현식을 처리(parsing)하는 역할





##### 참고

[앵귤러 튜토리얼](https://opentutorials.org/module/1590/9876)

<http://soomong.net/blog/2014/01/20/translation-ultimate-guide-to-learning-angularjs-in-one-day/>

