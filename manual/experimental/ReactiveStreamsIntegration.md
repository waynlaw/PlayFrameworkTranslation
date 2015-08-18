<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 리액티브 스트림 통합(실험중)

> **플레이의 실험적인 라이브러리는 아직 실환경에서 사용할 준비가 되진 않았다.** API는 변경될 수 있고, 기능은 적절하게 동작하지 않을 수 있다.

[Reactive Streams](http://www.reactive-streams.org/)은 설계 명세와 SPI로 현재 개발중에 있다. 리액티브 스트림은 다른 스트림 구현체가 함께 연결될 수 있도록 허락하는 공통 표준을 제공한다. SPI는 `Publisher`와 `Subscriber`처럼 약간 간단한 인터페이스로 아주 작은 부분이다.

플레이 2.4는 **experimental** `Future`, `Promise`, `Enumerator`나 `Iteratee`를 리액티브 스트림의 `Publisher`와 `Subscriber`로 적용할 수 있는 리액티브 스트림 통합 모듈을 제공한다.

## 알려진 이슈

* 구현체가 리엑티브 스트림 설계 명세의 0.4버전으로 완전하게 갱신되지 못했다. 예를들어 `Publisher`나 `Subscriber`는 사전 이벤트인 `onSubscribe`이벤트 없이 `onComplete` 이벤트를 보내거나 받게 된다. 이는 0.3에서는 허용되는 것이지만 0.4 버전에서는 아니다.
* `Input`이벤트에서 스트림이 `input.EOF`이벤트 잃어버릴수 없다는 것을 보증하고 `Input.Empty`를 위한 적절한 지원을 제공하도록 끌어올릴 필요가 있다. 순간적으로 이터레이티나 이뉴머레이터를 적용할 때 이벤트를 잃어버리는 한계가 있다.
* 성능튜닝이 끝나지 않았다.
* 모든 메인 스트림과 이터레이티 타입사이에서 2-way 변환에 대한 지원이 필요하다.
* 문서가 한계가 있다.
* 모듈의 동작이 자바로 테스트 되었다.

## 사용하기

다음과 같이 리액티브 스트림 통합 라이브러리를 프로젝트에 포함시킬 수 있다.

```scala
libraryDependencies += "com.typesafe.play" %% "play-streams-experimental" % "%PLAY_VERSION%"
```
모듈로의 모든 접근은 [`Streams`](api/scala/index.html#play.play.api.libs.streams.Streams) 객체를 통해서 한다.

여기 단일-요소 `Publisher`로 `Future`를 적용하는 예제가 있다.

```scala
val fut: Future[Int] = Future { ... }
val pubr: Publisher[Int] = Streams.futureToPublisher(fut)
```

`Streams` 객체의 [API 문서](api/scala/index.html#play.play.api.libs.streams.Streams)에서 더 많은 정보를 살펴볼 수 있다.