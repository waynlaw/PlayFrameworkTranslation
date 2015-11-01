<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 호스트이름 확인 환경설정하기

호스트이름 확인은 클라이언트가 적절한 서버와 대화하고 미들 어택을 하는 사람에 의해 리다이렉트되지 않았음을 확신하기 위한 [서버 신원 확인](http://tools.ietf.org/search/rfc2818#section-3.1)를 포함한 HTTPS의 작은 알려진 부분이다.

확인은 서버에 의해서 인증이 보내졌다는 것을 확인하고, 요청을 생성하는데 사용하곤 하는 호스트의 일부와 매치되는 인증의 `subjectAltName`필드에서 `dnsName`을 검증하는 것을 포함한다.

WS SSL은 [hostname verifier](http://docs.oracle.com/javase/7/docs/technotes/guides/security/jsse/JSSERefGuide.html#HostnameVerifier) 폴백 인터페이스를 구현한 [DefaultHostnameVerifier](api/scala/index.html#play.api.libs.ws.ssl.DefaultHostnameVerifier)를 사용하여 자동으로 호스트이름 확인을 수행한다.

## 호스트이름 확인자 수정하기

만약에 다른 호스트이름 확인자를 명시해야 한다면 스스로 생성한 [`HostnameVerifier`](http://docs.oracle.com/javase/7/docs/api/javax/net/ssl/HostnameVerifier.html)를 사용하기 위해 `application.conf`에 환경설정할 수 있다.

```
play.ws.ssl.hostnameVerifierClass=org.example.MyHostnameVerifier
```

## 디버깅

호스트이름 확인자는 `dnschef`를 사용하여 테스트할 수 있다. 완벽한 가이드는 이 문서의 범위를 벗어나지만 [Testing Hostname Verification](http://tersesystems.com/2014/03/31/testing-hostname-verification/)에서 예제를 찾을 수 있다.

## 참고자료

* [Fixing Hostname Verification](http://tersesystems.com/2014/03/23/fixing-hostname-verification/)
