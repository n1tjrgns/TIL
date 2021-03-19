[MSA] Reactor Mono vs Flux (feat.레퍼런스 번역내용 정리)

Mono와 Flux에 대해서 파악하기전에 먼저 Reactor 부터 알아보자.

의존성 추가나, 나에게 필요하지 않다고 생각되는 부분은 번역하지 않았다.



Reactor 소개

Reactor는 효율적인  `backpressure` 관리 방법으로,  JVM을 위한 완전한 Non-blocking 리액티브 프로그래밍이다. Java 8 의 함수형 API로써 CompletableFuture, Stream, Duration이 특징이다.

비동기 시퀀스 API를 Flux (N개의 요소)와 Mono (0또는 1개의 요소) 를 통해 구성 가능하도록 제공하고 광범위하게 리액티브 스트림 스펙을 구현한다.

Reactor는 또한  `reactor-netty` 프로젝트를 통한 비동기 프로세스 통신을 지원한다. 이는 MSA 아키텍쳐에 적합하고, Reactor Netty는 Http(Websockets 포함), TCP, UDP를 위한 백 프레셔 네트워크 엔진을 제공한다.

인코딩과 디코딩도 완전하게 지원한다.



전제 조건

Reactor Core는 자바 8 이상에서만 동작한다.

` org.reactivestreams:reactive-streams:1.0.3.` 에 의존성이 있다.



Reactive 프로그래밍 소개

Reactor는 Reactive Programming 패러다임의 구현이며, 다음과 같이 요약할 수 있다.

> Reactor는 데이터 스트림과 전파의 변경과 관련된 비동기 프로그래밍 패러다임이다. 이것은 정적, 동적 데이터 스트림을 프로그래밍 언어로 표현하기 쉽게 해준다.







참고

[프로젝트 리액터 공식 레퍼런스](https://projectreactor.io/docs/core/release/reference/#getting-started-introducing-reactor)