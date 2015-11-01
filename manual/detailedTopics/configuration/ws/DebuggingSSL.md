<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# SSL 연결 디버깅하기

HTTPS 연결에서의 이벤트가 통과하지 못하는 경우, JSSE 디버깅은 번거로울 수 있다.

WS SSL은 [Debugging Utilities](http://docs.oracle.com/javase/7/docs/technotes/guides/security/jsse/JSSERefGuide.html#Debug)와 Troubleshooting Security](http://docs.oracle.com/javase/7/docs/technotes/guides/security/troubleshooting-security.html)에 정의된 JSSE 옵션을 켜는 환경설정 옵션을 제공한다.

환경설정을 위해 `application.conf`의 `play.ws.ssl.debug`프로퍼티를 설정하자.

```
play.ws.ssl.debug = {
    # Turn on all debugging
    all = false
    # Turn on ssl debugging
    ssl = false
    # Turn certpath debugging on
    certpath = false
    # Turn ocsp debugging on
    ocsp = false
    # Enable per-record tracing
    record = false
    # hex dump of record plaintext, requires record to be true
    plaintext = false
    # print raw SSL/TLS packets, requires record to be true
    packet = false
    # Print each handshake message
    handshake = false
    # Print hex dump of each handshake message, requires handshake to be true
    data = false
    # Enable verbose handshake message printing, requires handshake to be true
    verbose = false
    # Print key generation data
    keygen = false
    # Print session activity
    session = false
    # Print default SSL initialization
    defaultctx = false
    # Print SSLContext tracing
    sslctx = false
    # Print session cache tracing
    sessioncache = false
    # Print key manager tracing
    keymanager = false
    # Print trust manager tracing
    trustmanager = false
    # Turn pluggability debugging on
    pluggability = false
}
```

> 주의: 이 기능은 JVM의 전역 시스템 프로퍼티인 `java.net.debug`의 설정을 변경한다. 게다가 이 기능은 [실행시간에 정적 프로퍼티들을 변경하며](http://tersesystems.com/2014/03/02/monkeypatching-java-classes/), 오직 개발 환경에서 사용하도록 되어있다.

## 장황한 디버깅

WS의 동작을 살펴보기 위해, 디버그 출력을 위해 SLF4J 로거를 설정 해야 한다.

```
logger.play.api.libs.ws.ssl=DEBUG
```

## 동적 디버깅

만약 동적으로 WSClient 인스턴스와 동작한다면 빌더 패턴을 사용하여 디버깅을 설정하기 위해 `SSLDebugConfig` 클래스를 사용할 수 있다.

```
val debugConfig = SSLDebugConfig().withKeyManager().withHandshake(data = true, verbose = true)
```

## 참고자료

오라클은 디버깅 JSSE 이슈에 대해 많은 글을 가지고 있다.

* [Debugging SSL/TLS connections](http://docs.oracle.com/javase/7/docs/technotes/guides/security/jsse/ReadDebug.html)
* [JSSE Debug Logging With Timestamp](https://blogs.oracle.com/xuelei/entry/jsse_debug_logging_with_timestamp)
* [How to Analyze Java SSL Errors](http://www.smartjava.org/content/how-analyze-java-ssl-errors)
