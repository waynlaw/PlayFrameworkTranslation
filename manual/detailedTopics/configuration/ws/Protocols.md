<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 프로토콜 환경설정하기

기본적으로 WS SSL은 JVM에서 사용가능한 TLS 프로토콜의 가장 안전한 버전을 사용한다.

* On JDK 1.7 and later, the default protocol is "TLSv1.2".
* On JDK 1.6, the default protocol is "TLSv1".

* JDK 1.7또는 그 이후의 경우 기본 프로토콜은 "TLSv1.2"이다.
* JDK 1.6에서 기본 프로토콜은 "TLSv1"이다.

JSSE에서 모든 프로토콜 목록은 [Standard Algorithm Name Documentation](http://docs.oracle.com/javase/7/docs/technotes/guides/security/StandardNames.html#jssenames)에서 확인할 수 있다.

## 기본 프로토콜 정의하기

만약 다른 [기본 프로토콜](http://docs.oracle.com/javase/7/docs/api/javax/net/ssl/SSLContext.html#getInstance\(java.lang.String\))을 정의하고 싶다면, 클라이언트에서 명시적으로 설정할 수 있다.

```
# Passed into SSLContext.getInstance()
play.ws.ssl.protocol = "TLSv1.2"
```

만약에 사용할 수 있는 프로토콜의 목록을 정의하고 싶으면 명시적으로 다음과 같이 설정할 수 있다.

```
# passed into sslContext.getDefaultParameters().setEnabledProtocols()
play.ws.ssl.enabledProtocols = [
  "TLSv1.2",
  "TLSv1.1",
  "TLSv1"
]
```

만약 JDK 1.8을 사용하고 있다면 또한 `jdk.tls.client.protocols` 시스템 프로퍼티를 설정하여 전역적으로 클라이언트 프로토콜을 활성화하도록 설정할 수 있다.

WS는 "SSLv3", "SSLv2", "SSLv2Hello"를 아주 많은 [보안 문제](https://www.schneier.com/paper-ssl.pdf)가 있는 나약한 프로토콜으로 간주한다. 그리고 `ws.ssl.enabledProtocols` 목록에 있으면 예외를 던지게 된다. 사실상 모든 서버가 `TLSv1`을 지원한다면 이 오래된 프로토콜을 사용하는데는 어떤 장점도 없을 것이다.

## 디버깅하기

환경설정 프로토콜을 위한 디버깅 옵션은 다음과 같다.

```
play.ws.ssl.debug = {
  ssl = true
  sslctx = true
  handshake = true
  verbose = true
  data = true
}
```
