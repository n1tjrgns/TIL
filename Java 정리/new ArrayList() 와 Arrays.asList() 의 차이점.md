## new ArrayList<>() 와 Arrays.asList() 의 차이점



Arrays.asList로 리스트 선언시 `list.add`원소 추가가 되지 않는 부분을 발견했다.



> 왜?



List는 내부 구조가 배열로 만들어져 있다.

따라서 asList()를 사용해서 반환되는 List도 배열을 갖게 된다.

이 때 asList()를 사용해서 내용을 수정하면 원본 배열도 함께 바뀌게 되고, 원본 배열을 수정하면 그 배열로 만들어두었던 asList()를 사용한 List 내용도 바뀌게 된다.



- 이로인해 Arrays.asList()로 만든 List에 새로운 원소를 추가하거나 삭제 할 수 없다.
- 객체 생성 시, 내부적으로 List를 위한 새로운 메모리 공간을 만드는 것이 아니다.
- 이미 존재하는 fixed sized의 배열 주소값을 가져와 List로 처리하기 때문에 생성된 List도 계속 fixed sized로 존재한다.

