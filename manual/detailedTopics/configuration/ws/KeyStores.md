<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 트러스트 스토어와 키 스토어 환경설정하기

트러스트 스토어와 키 스토어는 X.509 증명서를 포함한다. 이들 증명서는 공개(혹은 비공개)키를 포함하며, 각기 저마다 트러스트관리자나 키관리자에 의해 조직화되고 관리되어야 한다.

만약에 X.509 증명서를 생성할 필요가 있다면 더 많은 정보를 위해 [[Certificate Generation|CertificateGeneration]]를 살펴보기 바란다.

## 트러스트 관리자 환경설정하기

[트러스트 관리자](http://docs.oracle.com/javase/7/docs/technotes/guides/security/jsse/JSSERefGuide.html#TrustManager)는 신뢰있는 앵커를 유지하기위해 사용된다. 이들은 최상위 증명서로 인증기관에서 발행된 것이다. 이는 원격 인증 증명서(및 연결)이 신뢰되어야 하는지를 결정한다. HTTPS 클라이언트로서  요청의 대부분은 트러스트 관리자를 사용한다.

만약에 모든 환경설정을 하지 않으면 WS가 기본 트러스트 관리자를 사용한다. 이는 `${java.home}/lib/security/cacerts`에 있는 `cacerts` 키 스토어를 가리킨다. 만약에 명시적으로 트러스트 관리자를 환경설정하면 기본 설정을 오버라이드 할 것이고 `cacerts` 스토어는 포함되지 않을 것이다.

만약에 기본 트러스트 스토어를 사용하고 증명서를 포함한 다른 스토어를 추가하고 싶다면 트러스트 관리자에 정의해야 한다. [CompositeX509TrustManager](api/scala/index.html#play.api.libs.ws.ssl.CompositeX509TrustManager)는 적절한 것을 찾을 때까지 각각을 찾아 시도한다. 

```
play.ws.ssl {
  trustManager = {
      stores = [
        { path: ${store.directory}/truststore.jks, type: "JKS" }  # Added trust store
        { path: ${java.home}/lib/security/cacerts, password = "changeit" } # Default trust store
      ]
  }
}
```

> **주의**: 트러스트 스토어는 오직 보통은 JKS나 PEM 공개 키로 CA 증명서만 포함해야 한다. PKCS12 형식은 지원하지만 PKCS12는 트러스트 스토어에 비공개 키를 포함하지 않아야 하며, [참고 문서](http://docs.oracle.com/javase/7/docs/technotes/guides/security/jsse/JSSERefGuide.html#SunJSSE)에 설명되어있다.

## 키 관리자 환경설정하기

[키 관리자](http://docs.oracle.com/javase/7/docs/technotes/guides/security/jsse/JSSERefGuide.html#KeyManager)는 원격 호스트를 위해 현재 키를 사용한다. 키 관리자는 일반적으로 오직 클라이언트 인증을 위해 사용된다(또한 상호간의 TLS로 알려져 있다). 

[CompositeX509KeyManager](api/scala/index.html#play.api.libs.ws.ssl.CompositeX509KeyManager)는 트러스트 관리자와 동일한 방법으로 여러 스토어를 포함할 수 있다.

```
play.ws.ssl {
    keyManager = {
      stores = [
        {
          type: "pkcs12",
          path: "keystore.p12",
          password: "password1"
        },
      ]
    }
}
```

> **NOTE**: A key store that holds private keys should use PKCS12 format, as indicated in the [reference guide](http://docs.oracle.com/javase/7/docs/technotes/guides/security/jsse/JSSERefGuide.html#SunJSSE).

> **주의**: 비밀 키를 고정하는 키 스토어는 반드시 PKCS12 형식을 사용해야 하며, [참고 문서](http://docs.oracle.com/javase/7/docs/technotes/guides/security/jsse/JSSERefGuide.html#SunJSSE)에 설명되어있다.

## 스토어 환경설정하기

스토어는 [KeyStore](http://docs.oracle.com/javase/7/docs/api/java/security/KeyStore.html) 객체에 해당하며, 이는 트러스트 스토어와 키 스토어를 위해 사용된다. 스토어는 [타입](http://docs.oracle.com/javase/7/docs/technotes/guides/security/StandardNames.html#KeyStore)을 가져야 하며, `PKCS12`, [`JKS`](http://docs.oracle.com/javase/7/docs/technotes/guides/security/crypto/CryptoSpec.html#KeystoreImplementation), `PEM` (aka DER 증명서로 인코딩된 Base64)이 그 종류가 될 수 있고 관련된 비밀번호를 가질 수 있다.

스토어는 `path`나 `data` 속성을 가져야한다. `path` 속성은 파일 시스템의 경로이어야 한다.

```
{ type: "PKCS12", path: "/private/keystore.p12" }
```

`data` 속성은 PEM 인코딩된 증명서의 문자열을 포함해야 한다.

```
{
  type: "PEM", data = """
-----BEGIN CERTIFICATE-----
...certificate data
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
...certificate data
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
...certificate data
-----END CERTIFICATE-----
"""
}
```

## 디버깅

키 관리자와 신뢰 관리자를 디버깅하기 위해 다음 플래그를 설정하자.

```
play.ws.ssl.debug = {
  ssl = true
  trustmanager = true
  keymanager = true
}
```

## 참고자료

대부분의 경우 증명서를 한번 설치하고 나면 광범위환 환경설정을 하는 것은 필요하지 않을 것이다. 만약에 환경설정에 어려움이 있다면 다음 블로그 포스트가 아주 유용할 것이다.

* [Key Management](http://docs.oracle.com/javase/7/docs/technotes/guides/security/crypto/CryptoSpec.html#KeyManagement)
* [Java 2-way TLS/SSL (Client Certificates) and PKCS12 vs JKS KeyStores](http://blog.palominolabs.com/2011/10/18/java-2-way-tlsssl-client-certificates-and-pkcs12-vs-jks-keystores/)
* [HTTPS with Client Certificates on Android](http://chariotsolutions.com/blog/post/https-with-client-certificates-on/)
