# 기본 SSLContext 사용하기

만약 여러분을 위해 WS를 제공하는 SSLContext를 사용하고 싶지 않고 `SSLContext.getDefault`를 사용하고 싶다면 다음과 같이 설정해야 한다.

```
play.ws.ssl.default = true
```

## 디버깅

기본 컨텍스트를 디버깅하고 싶다면 다음과 같이 설정해야 한다.

```
play.ws.ssl.debug {
  ssl = true
  sslctx = true
  defaultctx = true
}
```

만약 기본 SSLContext를 사용한다면 JSSE의 동작을 변경하는 유일한 방법은 [JSSE 시스템 프로퍼티](http://docs.oracle.com/javase/7/docs/technotes/guides/security/jsse/JSSERefGuide.html#Customization)를 조작해야 한다.

> **다음**:  [[SSL 디버깅하기|DebuggingSSL]]
