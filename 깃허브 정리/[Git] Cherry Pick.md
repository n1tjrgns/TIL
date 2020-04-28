## [Git] Cherry Pick

#### 체리픽(Cherry Pick)이란?

- 특정 커밋의 변경사항을 가져오는 작업



#### 체리픽(Cherry Pick)을 사용해야 하는 경우

1. 여러 브랜치로 작업을 하는 중, 다른 브랜치에 있는 기능을 현재 브랜치에서 확인하고 싶을 때
2. 누군가 작업도중 Commit Revert를 사용해 파일을 지운 경우



- 이때 체리픽을 사용하지 않고 merge를 해서 사용하려하면 conflict가 발생한다



#### 방법

팀원 중 한명이 나의 브랜치의 커밋을 Revert 후 push 했다고 가정해보자.

나는 다음 작업을 위해 원격 브랜치를 우선 pull로 가져올 것이다. 하지만 그럴 경우 분명 있어야 할 코드가 없어진 것을 확인할 수 있을것이다.

Revert 역시 하나의 commit log이기 때문이다. 

이럴 경우, 사라진 내 커밋을 체리픽을 사용해서 추가해줘야한다.

인텔리제이나, 소스코드와 같은 깃 툴을 사용한다면 commit log에서 가져오고 싶은 변경사항을 우클릭해서 체리픽을 눌러주기만 하면 된다.

![1561790360262](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1561790360262.png)



git bash를 사용해 feature/project-10 라는 브랜치에 feature/project-34의 커밋을 가져오고 싶은 경우 우선 feature/project-10 브랜치로 이동 한다. 그리고 feature/project-34 브랜치에서 내가 가져오고 싶은 커밋의 SHA1 ID를 복사한다.(SHA1 ID는 보통 ex) 0dab39s 의 형태로 생겼다.) 

SHA는 커밋을 나타내는 해시 값이다.

SHA1 ID를 알았다면 체리픽을 실행한다.

만약 conflict가 발생할 경우 해당 파일을 열어 충돌 부분은 수정한 후 커밋을 하면 된다.

```git
git checkout feature/project-10
git cherry-pick 0dab39s
error : could not apply 0dab39s ... commit
```

sample.java 수정

```git
git add sample.java
git commit
```

