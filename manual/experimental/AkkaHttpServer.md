<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 아카 HTTP 서버 백엔드(실험중)

> **플레이의 실험적인 라이브러리는 아직 실환경에서 사용할 준비가 되진 않았다.** API는 변경될 수 있고, 기능은 적절하게 동작하지 않을 수 있다.

플레이 2 API는 HTTP 서버 백엔드의 최상위에 놓여져 있다. 기본적인 API 서버 백엔드는 [Netty](http://netty.io/) 라이브러리를 사용한다. 플레이 2.4에서는 다른 **실험적인** 백엔드 [Akka HTTP](http://doc.akka.io/docs/akka-stream-and-http-experimental/current/)기반의 백엔드도 또한 사용할 수 있다. Akka HTTP 백엔드는 Netty HTTP 백엔드와 동일한 플레이 API를 제공하는걸 목적으로 한다. 지금 Akka HTTP 백엔드는 약간의 기능들이 빠져있다.

실험적인 Akka HTTP 백엔드는 개념의 기술적인 증거이다. 실환경에서 사용할 수 없고 플레이 API를 모두 구현하지도 않았다. 새로운 백엔드의 목적은 Akka HTTP가 플레이의 미래 버전을 위한 백엔드로 사용가능한지 실험하는 것이다. 백엔드는 또한 우리의 친구인 Akka 프로젝트의 가치있는 테스트 케이스를 제공하는 것도 있다.

## 알려진 이슈

* WebSockets을 지원하지 않는다. 이는 앞으로 Akka HTTP가 웹소켓 지원 이후에 수정될 예정이다.
* HTTPS를 지원하지 않는다.
* `Content-Length`헤더가 없으면 Akka HTTP 서버는 항상 청크 인코딩을 사용한다. Netty 백엔드는 `Content-Length`을 얻기 위해 요청을 자동으로 버퍼시키는 것과는 차이가 있다.
* `X-Forwarded-For`을 지원하지 않는다.
* `RequestHeader.username`을 지원하지 않는다.
* 서버의 셧다운이 조금 무자비 하다. HTTP 서버 액터를 그냥 죽인다.
* 성능을 조정하기 위한 시도가 없었다. 성능은 Netty보다 더 느릴 것이다. 예를들어, 현재 플레이의 `Array[Byte]`와 Akka의 `ByteString` 사이에서 아주 많은 복사가 일어난다. 이는 최적화 될 예정이다.
* 구현체는 Netty로부터 많은 중복 코드를 담고 있다.
* 이 페이지에 씌여진 코드에 대한 더 적절한 문서가 없다.

## 사용하기

Akka HTTP 서버 백엔드를 사용하기 위해 먼저 Netty 서버를 사용하지 않도록 하고, Akka HTTP 서버 플러그인을 프로젝트에 추가하자. 

```scala
lazy val root = (project in file("."))
  .enablePlugins(PlayScala, PlayAkkaHttpServer)
  .disablePlugins(PlayNettyServer)
```

이제 플레이는 자동으로 prod와 dev 모드, 테스트를 실행하기 위해 Akka HTTP 서버를 선택할 것이다.

### 직접 Akka HTTP 서버 선택하기

만약에 어떤 이유에서 클래스패스에 Akka HTTP서버와 Netty서버를 둘다 가져야 한다면, 이에 대해서 직접 선택해야할 필요가 있다. 예를 들어 dev모드에서 `play.server.provider` 시스템 프로퍼티를 사용해서 할 수 있다.

```
run -Dplay.server.provider=play.core.server.akkahttp.AkkaHttpServerProvider
```

### Akka HTTP 서버가 동작하는지 검증하기

Akka HTTP 서버가 동작할 때 모든 요청에 `HTTP_SERVER`태그에 `akka-http`의 값이 붙는다. Netty 백엔드는 이 태그에 값이 없다.

```scala
Action { request =>
  assert(request.tags.get("HTTP_SERVER") == Some("akka-http"))
  ...
}
```

### Akka HTTP 서버 환경설정하기

Akka HTTP 서버는 Play처럼 Typesafe Config로 환경설정을 할 수 있다. Note: 플레이가 개발 모드로 실행 중일 때, 현재 프로젝트의 리소스는 서버의 클래스패스에서 사용할 수 없다. 환경설정은 시스템의 프로퍼티나 JAR 파일의 리소스로 제공되어야 할 것이다.

```
play {

  # Configuration for Play's AkkaHttpServer
  akka {

    # How long to wait when binding to the listening socket
    http-bind-timeout = 5 seconds

  }

}
```
