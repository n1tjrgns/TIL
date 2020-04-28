## 웹 프론트엔드 성능 비용 Reflow & Repaint

나는 백엔드에 더 관심이 많고  앞으로도 그럴테지만, 웹 개발자이고 웹에 대한 이해가 필요하다고 생각된다.

일을 하다가 궁금한 부분이 생겨서 질문을 했는데, `Reflow & Repaint` 라는걸 알게되어 궁금해서 바로 이 글을 쓰게 되었다.



#### 브라우저 렌더링 프로세스

- Rendering Engine Basic Flow

![1570079233487](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1570079233487.png)



브라우저가 네트워크 계층에서 요청된 데이터를 받아오면, 렌더링 엔진이 움직이기 시작한다.

1. 렌더링 엔진이 HTML 코드를 파싱해 DOM Tree 생성, CSS 코드를 파싱해 스타일 규칙 생성
2.  위 두 가지를 합쳐 `렌더 트리` 생성
3. 렌더 트리로 `배치` 시작
4. 실제 화면 그리기



![1570081976251](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1570081976251.png)

1~3 번은 `Reflow`, 4번은 `Refaint`에 해당하는 단계이다.



특정 엘리먼트의 Color 값 변화

- Repaint 발생

특정 엘리먼트의 포지션의 변화

- 해당 엘리먼트의 Repaint + 레이아웃의 Reflow 발생



즉, 엘리먼트의 폰트 사이즈를 키우는 단순한 작업이더라도

전체 Render Tree의 Repaint, Reflow 유발



#### Repaint

- 엘리먼트 스킨은 변하지만, 레이아웃에는 영향을 미치지 않을 때



#### Reflow

- 문서 내 노드들의 레이아웃, 포지션을 재계산 후 다시 뿌림
- Repaint보다 심각한 퍼포먼스 저하를 유발시키는 프로세스
  - 브라우저 창 크기 수정
  - DOM 트리 수정
  - Style sheet 추가, Style Property 수정, 편집(입력, ContentEditable)
  - 등등



> 색칠공부 이외의 대부분이 Reflow에 해당하는 것 같음..



#### Reflow를 최소화하는 방법

- 애니메이션이 들어간 노드는 position:fixed 또는 position:absolute로 지정해 전체노드에서 분리
- cssText를 활용해 Reflow & Repaint 최소화
- DOM 사용을 최소화해 Reflow 비용 줄이기
- 등등



