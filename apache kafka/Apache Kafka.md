## 카프카 (Kafka)

아파치 카프카(Apache Kafaka) 아파치 소프트웨어 재단이 스칼라로 개발한 오픈 소스 메시지 브로커 프로젝트이다.  `pub-sub모델` 의 메세지 큐이며, 분산환경에 특화되어 설계되어 있다.

그로인해 기존의 RabbitMQ와 같은 다른 메세지 큐와의 성능 차이가 있다.(훨씬 빠르다.)

이 외에도 클러스터 구성, fail-over, replication과 같은 여러가지 특징을 가지고 있다.



> **pub-sub 모델** : 메시지를 보내고(Publish : 발행) 받는 (Subscribe : 구독) 형태의 통신
>
> Publisher는 메세지를 topic을 통해 카테고리화 한다.
>
> Receiver는 해당 topic을 구독(Subscribe) 함으로써 메세지를 읽어 올 수 있다.
>
> Publisher는 topic에 대한 정보만 알고 있고, Subscriber도 topic만 바라본다.
>
> Publisher와 Subscriber는 서로 모르는 상태다.



#### 카프카의 구성요소 및 특징

- topic과 partition
- Producer, Consumer
- broker, zookeeper
- consumer group
- replication





#### Topic과 Partition

메세지는 topic으로 분류되고, topic은 여러개의 partition으로 나눠진다.

partition내의 한 칸은 log 라고 부른다. 데이터는 한 칸의 로그에 순차적으로 append 된다.

메세지의 상대적인 위치를 나타내는게 offset이다. 배열의 index라고 생각하면 편하다.

![1560752051177](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1560752051177.png)



위 그림을 보면 하나의 Topic에 여러개의 Partition이 있는 것을 볼 수 있다.

> 이렇게 사용하는 이유는??



#### 하나의 topic에 하나의 partition  VS  하나의 topic에 여러개의 partition 

메세지는 카프카의 해당 토픽에 쓰여진다. 쓰는 과정도 시간이 소비된다.

몇 백만건의 메세지가 동시에 카프카에 쓰여진다고 가정하면, 하나의 파티션에 순차적으로 append 될 텐데, 당연히 오래 걸릴 것이다.

그렇기 때문에 여러개의 파티션을 두어 분산저장을 하는 것이다.

쓰는 과정이 `병렬로 처리`되기 때문에 시간이 그 만큼 절약된다.

> 주의해야 할 점은 한 번 늘린 파티션은 절대로 줄일 수 없기 때문에 운영중에, 파티션을 늘려야 하는건 충분히 고려해봐야한다.

또한, 파티션을 늘렸을 때 메세지가 Round-robin 방식으로 쓰여진다. 즉, 순차적으로 메세지가 쓰여지지 않는다는 말이다.

해당 토픽의 소비자가 만약 메세지의 순서가 엄청 중요한 모델(증권 시스템)이라면 순서를 보장해 주지 않아 상당히 위험하다.



#### Producer, Consumer

`Producer`는 메세지를 생산하는 주체이다. 메세지를 만들고 Topic에 메세지를 쓴다.

Producer는 Consumer의 존재를 알지 못한다. 단순히 카프카에 메세지를 쓸 뿐이다.

만약 여러개의 토픽에 여러 파티션을 나누고, 특정 메세지에 따라 특정 파티션에 저장하고 싶다면, `key 값`을 통해 분류 할 수 있다. 하지만 구현하기 매우 어렵다.



`Consumer`는 소비자로써 메세지를 소비하는 주체이다. 마찬가지로 Producer의 존재를 알지 못한다.

해당 topic을 구독함으로써, 자기가  스스로 조절해가며 소비 할 수 있다.

소비를 했다는 표시는 해당 topic내의 각 파티션에 존재하는 offset의 위치를 통해 이전에 소비했던 offset 위치를 기억하고 관리한다. 혹여 Consumer가 죽었다 살아나도, 마지막으로 읽었던 위치에서 부터 다시 읽어들일 수 있다.

덕분에 fail-over에 대한 신뢰가 존재한다.



#### Consumer Group

![1560753146965](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1560753146965.png)



Consumer group은 말 그대로 consumer들의 묶음이다. 기본적인 룰이 존재하는데, `반드시 해당 topic의 partition은 그 consumer group과 1:n 매칭을 해야한다.`

기본적인 룰 때문에 아래와 같은 상황이 존재한다.

1. partition 3개 : consumer 2개 => consumer 중 하나는 2개의 파티션을 소비한다.
2. partition 3개 : consumer 3개 => 각각 1:1 매칭
3. partition 3개 : consumer 4개 => consumer 1개는 아무것도 하지 않을 수 있다. 



그렇기 때문에 파티션을 늘릴 때는, Consumer의 개수도 고려해야한다.

보통은 개수를 같이 맞춰주는 것을 권장하지만, 실제 메세지가 쌓이는 속도보다 처리하는 속도가 훨씬 빠르다면 굳이 1:1 매핑보다는 파티션 개수 => 컨슈머 개수로 설정하는 것도 나쁘지 않다. 



##### Consumer group이 존재하는 이유는?

consumer group은 하나의 topic에 대한 책임을 가지고 있다.

같은 그룹내의 특정 consumer가 down 되면, 특정 consumer가 맡았던 partition에 대한 소비를 할 수 없는 상황이 된다. `Rebalance`된 상황이라고 한다. 파티션 재조정을 통해서 다른 consumer가 partition의 소비를 이어서 하게된다.

이는 offset 정보를 그룹내에서 서로간 공유하고 있기 때문에, down되기 직전의 offset위치를 알고 그 다음부터 소비하는 것이다.



#### Broker, Zookeeper

broker는 카프카의 서버를 칭한다. broker.id=1..n으로 함으로써 동일한 노드내에서 여러개의 broker 서버를 띄울 수도 있다.

zookeeper는 이러한 분산 메세지 큐의 정보를 관리해 주는 역할을 한다.

kafak를 띄우기 위해서는 zookeeper가 반드시 실행되어야 한다.



#### Replication

local에 broker3대를 띄우고 (replica-factor=3)으로 복제되는 경우를 살펴보자.

복제는 수평적으로 scale-out 이다. broker 3대에서 하나의 서버만 leader가 되고 나머지 둘은 follwer가 된다.

producer가 메세지를 쓰고, consumer가 메세지를 읽는 건 오로지 leader가 전적으로 역할을 담당한다.



> 그럼 나머지 follower 들은?

leader와 항상 싱크를 맞춘다. 만약 leader가 죽을 경우, `나머지 follower중에 하나가 leader로 선출`되어 메세지의 쓰기/읽기를 처리한다.



##### 설정하는 방법

Producer config 정보에서 ack(acknowlegement)옵션이 있다. 메세지를 보내고 잘 받았는지 확인받는 메세지다.

`ack 옵션에 따라서 네트워크를 몇번을 타야하는지를 결정할 수 있다. `

ack=1 로 하면, producer가 메세지를 leader한테 보내고 쓰여지고, 나머지 follower들이 똑같이 메세지를 다 복사할 때 까지 기다린다. 복사까지 완벽하게 되면 그제서야 ack 옵션을 producer에게 보낸다.

위와 같이 구성하면 leader가 뻗더라도 복제된 데이터가 follwer들에게 있어, 메시지 유실이 전혀 없지만, 복제할 때 까지 네트워크를 타고 흐르는 시간을 기달려야 한다.

그래서 보통은 default로 한다.  leader한테만 쓰여지면 바로 ack를 받을 수 있도록 설정한다.



##### 참고

[위키백과 정의](https://ko.wikipedia.org/wiki/%EC%95%84%ED%8C%8C%EC%B9%98_%EC%B9%B4%ED%94%84%EC%B9%B4)

[https://medium.com/@umanking/%EC%B9%B4%ED%94%84%EC%B9%B4%EC%97%90-%EB%8C%80%ED%95%B4%EC%84%9C-%EC%9D%B4%EC%95%BC%EA%B8%B0-%ED%95%98%EA%B8%B0%EC%A0%84%EC%97%90-%EB%A8%BC%EC%A0%80-data%EC%97%90-%EB%8C%80%ED%95%B4%EC%84%9C-%EC%9D%B4%EC%95%BC%EA%B8%B0%ED%95%B4%EB%B3%B4%EC%9E%90-d2e3ca2f3c2](https://medium.com/@umanking/카프카에-대해서-이야기-하기전에-먼저-data에-대해서-이야기해보자-d2e3ca2f3c2)

