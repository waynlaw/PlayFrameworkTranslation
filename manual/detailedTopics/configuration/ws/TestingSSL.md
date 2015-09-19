# SSL 테스트하기

Testing an SSL client not only involves unit and integration testing, but also involves adversarial testing, which tests that an attacker cannot break or subvert a secure connection.

SSL 클라이언트를 테스트하는 것은 유닛 테스트 및 통합 테스트만 포함되는 것 뿐만아니라 공격자가 안전한 연결을 끊거나 파멸시킬수 없도록 하는 테스트(adversarial testing)도 포함한다.

## 유닛 테스트

플레이는 `wsCall`과 `wsUrl`을 제공하는 `play.api.test.WsTestClient`를 가지고 있다. 이는 `PlaySpecification`과 `in new WithApplication`에 아주 유용할 수 있다.

```
"calls index" in new WithApplication() {
  await(wsCall(routes.Application.index()).get())	
}
```

```
wsUrl("https://example.com").get()
```

## 통합 테스트

여러분의 클라이언트가 환경설정이 잘 되었는지 확인하고 싶다면 JSSE 설정을 확인하기 위한 API를 가지고 있는 [HowsMySSL](https://www.howsmyssl.com/s/api.html)을 호출할 수 있다.

@[context](code/HowsMySSLSpec.scala)

해제 확인이나 알고리즘 비활성화 같은 개인적인 환경설정을 포함하고있는 테스트를 작성할 경우 SBT로 시스템 프로퍼티를 전달할 필요가 있음을 주의하라.

```
javaOptions in Test ++= Seq("-Dcom.sun.security.enableCRLDP=true", "-Dcom.sun.net.ssl.checkRevocation=true", "-Djavax.net.debug=all")
```

## 적대적 테스팅(Adversarial Testing)

이런 테스트를 작성하는 것은 꽤 쉽고, 신용할 수 있는 프로그래머에 대해서 이런 적대적 테스팅 수행은 매우 만족스러울 수 있다.

> **주의**: 여기서 완벽한 리스트를 제시하지 않으며 가이드 정도만 제공한다. 보안이 가장 중요한 상황에서, 리뷰는 전문적인 정보보안 컨설턴트에 의해서 수행되어야 한다.

### 증명서 확인 테스트

"https://example.com"로 연결하는 테스트를 작성하자. 서버는 subjectAltName이 dnsName이라고 말하는 인증서를 수여해야 하지만 인증서는 신뢰 저장소에 없는 CA 인증서에 의해 서명되어야 한다. 클라이언트는 이를 거절해야 할 것이다.

이는 아주 일반적인 실패다. [mitmproxy](http://mitmproxy.org)나 [Fiddler](http://www.telerik.com/fiddler) 처럼 인증서 확인이 비활성화 되거나 프록시의 인증서가 트러스트 스토어에 명시적으로 추가되었을 때만 동작하도록하는 많은 수의 프록시가 있다.

### 약한 Cipher Suites 테스트

서버는 핸드쉐이크에서 NULL 이나 ANON cipher suite를 초함한 cipher suite를 보내주어야 한다. 만약에 클라이언트가 이를 승인하면 이는 암호화되지 않은 데이터를 보내준 것이다.

> **주의**: 서버의 cipher suites의 더 깊이 있는 테스트를 위해, [sslyze](https://github.com/iSECPartners/sslyze)를 살펴보자.

### 인증서 유효성 검증 테스트

약산 서명(signatures)을 위한 테스트를 위해, 서버는 예를들어 MD2 다이제스트 알고리즘처럼 사인이 된 인증서를 전달해야 한다. 클라이언트는 너무 약하다며 이를 거절해야 한다. 

약한 인증(certificate)을 위한 테스트를 위해, 서버는 클라이언트에게 1024비트 이하의 키 사이즈를 가진 공개 키를 포함하여 인증을 보내야 한다.  클라이언트는 너무 약하다며 이를 거절해야 한다. 

> **주의**: 인증서 유효성 검증의 더 깊이 있는 테스트를 위해, [tlspretense](https://github.com/iSECPartners/tlspretense)와 [frankencert](https://github.com/sumanj/frankencert)를 살펴보자.

### 호스트이름 확인 테스트

"https://example.com"로 테스트를 작성하자. 서버가 subjectAltName의 dnsName이 example.com 아닌 인증서를 수여하면, 연결은 종료되어야 한다.

> **주의**: 더 깊이 있는 테스트를 위해, [dnschef](http://tersesystems.com/2014/03/31/testing-hostname-verification/)를 살펴보자.
