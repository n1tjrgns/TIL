## [프로그래머스] 베스트앨범 (level 3) java

주말 + 평일 틈틈이 풀었음에도 꼬박 일주일이나 걸렸다..

질문으로 도움까지 요청하며 정말 겨우 해결했다.



문제는 [링크](https://programmers.co.kr/learn/courses/30/lessons/42579)로 대체.



```
	// 요구사항 정리
    // 1. 가장 많이 재생된 장르를 찾는다
    // 2. 같은 장르의 노래중 재생 횟수가 높은 노래가 먼저 재생된다.
    // 3. 재생횟수가 같은 경우에는 인덱스가 낮은 노래가 먼저 재생된다.
    public int[] solution(String[] genres, int[] plays) {
        int[] answer = {};
        Map<String,Integer> topPlay = new HashMap<>();
        Map<String,Map<Integer,Integer>> firstSong = new HashMap<>();
        Map<Integer,Integer> countList = null;

        int genLength = genres.length;

        for(int i=0; i<genLength; i++){
            //1번 요구사항을 구하기 위해, 같은 장르이면 +
            if(topPlay.get(genres[i]) == null){
                topPlay.put(genres[i], plays[i]);
            }else{
                topPlay.put(genres[i], topPlay.get(genres[i])+plays[i]);
            }
            //3항 연산자로 나타낸다면
            //topPlay.put(genres[i], topPlay.containsKey(genres[i]) ? topPlay.get(genres[i])+plays[i] : plays[i]) ;

            if(firstSong.containsKey(genres[i])){
                countList = firstSong.get(genres[i]);
            }else{
                countList = new HashMap<>();
            }
            //3항 연산자로 나타낸다면
            //Map<Integer,Integer> countList = firstSong.containsKey(genres[i]) ? firstSong.get(genres[i]) : new HashMap<>();

            countList.put(i,plays[i]);
            firstSong.put(genres[i],countList);
        }


        //리스트에 map의 키 값 담기
        List<String> topPlayKey = new ArrayList<>(topPlay.keySet());

        Collections.sort(topPlayKey, new Comparator<String>() {
            @Override
            public int compare(String o1, String o2) {
                //내림차순, value 값으로 정렬
                return topPlay.get(o2).compareTo(topPlay.get(o1));
                //오름차순
                // return topPlay.get(o1).compareTo(topPlay.get(o2));
            }
        });

        //System.out.println("firstSong : " + firstSong);
        //키값을 정렬했으니 키 값 순서대로 그 안의 map을 정렬하면된다.
        List<Integer> temp = new ArrayList<>();
        for (String key : topPlayKey) {
            List<Map.Entry<Integer, Integer>> list = new ArrayList<>(firstSong.get(key).entrySet());

            Collections.sort(list, new Comparator<Map.Entry<Integer, Integer>>() {
                @Override
                public int compare(Map.Entry<Integer, Integer> o1, Map.Entry<Integer, Integer> o2) {
                    if (o1.getValue() == o2.getValue()){
                        return o1.getKey().compareTo(o2.getKey());
                    }else{
                        return o2.getValue().compareTo(o1.getValue());
                    }
                }
            });

            Map<Integer,Integer> map = new HashMap<>();
            map.entrySet().toArray();

            int index = 0;
            for (Map.Entry<Integer, Integer> entrys : list)
            {
                if(index < 2){
                    int entrysKey = entrys.getKey();
                    temp.add(entrysKey);
                    index++;
                }
            }
        }

        answer = new int[temp.size()];
        for(int i=0; i<temp.size(); i++){
            answer[i] = temp.get(i);
        }

        return answer;
    }
```



우선, 가장 많이 재생된 장르를 찾는것이 첫 번째 목표다.

장르별 재생횟수를 전부 더해준다.

