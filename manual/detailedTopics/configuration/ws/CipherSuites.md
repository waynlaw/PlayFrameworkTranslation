<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# Cipher Suites 환경설정하기

[Cipher suite](https://en.wikipedia.org/wiki/Cipher_suite)는 실제로 키 교환을 설명하는 것, 벌크 암호화, 메시지 인증, 랜덤 수 함수 4가지로 구분된다. cipher suites의 [공식 적인 네이밍 규칙](http://utcc.utoronto.ca/~cks/space/blog/tech/SSLCipherNames)은 없지만, 대부분의 cipher suites는 다음과 같은 순서로 서술한다. 예를들어 "TLS_DHE_RSA_WITH_AES_256_CBC_SHA"는 키 교환은 DHE를 사용하고, 서버 인증서 인증을 위해 RSA를, 스트림 사이퍼를 위해 CBC 모드의 256비트 키의 AES를 메시지 인증을 위해 SHA를 사용한든 것을 나타낸다.

## Ciphers 활성화 환경설정하기

cipher suites의 목록은 1.6, 1.7과 1.8 사이에서 상당한 변화가 있었다.

1.7과 1.8에서는 기본으로 [특출난](http://sim.ivi.co/2011/07/jsse-oracle-provider-preference-of-tls.html) cipher suite를 사용한다.

1.6에서는 특출난 목록은 [정리가 되어있지 않고](http://op-co.de/blog/posts/android_ssl_downgrade/), 이상한 누군가의 앞에서 조금 부서지기 쉬운 cipher suites나, 약하다고 평가되는 다수의 cipher를 포함하고 있다. 활성화할 수 있는 cipher suites의 기본 리스트는 다음과 같다.

```
  "TLS_DHE_RSA_WITH_AES_256_CBC_SHA",
  "TLS_DHE_RSA_WITH_AES_128_CBC_SHA",
  "TLS_DHE_DSS_WITH_AES_128_CBC_SHA",
  "TLS_RSA_WITH_AES_256_CBC_SHA",
  "TLS_RSA_WITH_AES_128_CBC_SHA",
  "SSL_RSA_WITH_RC4_128_SHA",
  "SSL_RSA_WITH_RC4_128_MD5",
  "TLS_EMPTY_RENEGOTIATION_INFO_SCSV" // per RFC 5746
```

cipher suite의 목록을 `ws.ssl.enabledCiphers`의 설정을 사용하여 직접 환경설정 할 수 있다.

```
play.ws.ssl.enabledCiphers = [
  "TLS_DHE_RSA_WITH_AES_128_GCM_SHA256"
]
```

이는 예를들어 오직 DHE나 ECDHE cipher suite가 PFE를 활성화 하는 것 처럼 완전 순방향 보안을 활성화할 때 유용하다.

## 추천: DHE 키 사이즈 증가시키기

Diffie Hellman은 최근에 완벽한 순방향 보안을 제공하는 것 때문에 화재가 되었다. 1.6과 1.7에서는 서버의 DHE 핸드쉐이크는 대부분 1024로 설정되고 이는 약하고 공격자에게 방어가 잘 되지 않을 수 있다.

만약 JDK 1.8을 사용하고 있다면 시스템 프로퍼티 `-Djdk.tls.ephemeralDHKeySize=2048`를 설정하는 것을 추천한다. 이 설정은 핸드쉐이트에서 강력한 키 사이즈를 보장한다. [Ephemeral Diffie-Hellman키의 크기 커스터마이징](http://docs.oracle.com/javase/8/docs/technotes/guides/security/jsse/JSSERefGuide.html#customizing_dh_keys)를 살펴보자.

## 추천: 완벽한 순방향 보안으로 Ciphers 사용하기

TLS 및 DTLS의 안전한 사용을 위한 권장사항에 따라서 다음 chipher suite를 추천한다.

```
play.ws.ssl.enabledCiphers = [
  "TLS_DHE_RSA_WITH_AES_128_GCM_SHA256",
  "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256",
  "TLS_DHE_RSA_WITH_AES_256_GCM_SHA384",
  "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384",
]
```

이 cipher들 중에 몇몇은 오직 JDK 1.8에서만 사용할 수 있다.

## Disabling 전체적으로 약한 Ciphers와 약한 키 사이즈 비활성화하기

`jdk.tls.disabledAlgorithms`는 약한 cipher를 예방하기 위해 사용할 수 있고, 또한 핸드쉐이크에서 사용되어지는 것에 대한 [작은 키 사이즈](http://sim.ivi.co/2011/07/java-se-7-release-security-enhancements.html)를 예방할 수 있다. 이는 오라클 JDK 1.7과 그 이상에서만 사용할 수 있는 [유용한 기능](http://sim.ivi.co/2013/11/harness-ssl-and-jsse-key-size-control.html)이다.

비활성화 알고리즘을 위한 공식 문서는 [JSSE Reference Guide](http://docs.oracle.com/javase/7/docs/technotes/guides/security/jsse/JSSERefGuide.html#DisabledAlgorithms)에 있다.

TLS를 위해, 코드는 프로토콜 이후의 cipher suite의 처음 부분에 연결될 것이다. 예를들어 TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384는 관계있는 cipher로 ECDHE를 가진다. 비활성화 알고리즘을 위해 사용하는 파라미터 이름은 분명하지 않으며, 그러나 [프로바이더 문서](http://docs.oracle.com/javase/7/docs/technotes/guides/security/SunProviders.html)에 목록으로 나타나 있고, [소스 코드](http://grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/7u40-b43/sun/security/ssl/SSLAlgorithmConstraints.java#265)에서 확인할 수 있다.

(X.509 인증서에서 약한 키와 서명 알고리즘으로 검토한) `jdk.tls.disabledAlgorithms`나 `jdk.certpath.disabledAlgorithms`를 활성화하기 위해 프로퍼티 파일을 생성해야 한다.

```
# disabledAlgorithms.properties
jdk.tls.disabledAlgorithms=EC keySize < 160, RSA keySize < 2048, DSA keySize < 2048
jdk.certpath.disabledAlgorithms=MD2, MD4, MD5, EC keySize < 160, RSA keySize < 2048, DSA keySize < 2048
```

그 후에 [java.security.properties](http://bugs.java.com/bugdatabase/view_bug.do?bug_id=7133344)와 함께 JVM을 실행시키자.

```
java -Djava.security.properties=disabledAlgorithms.properties
```

## 디버깅

ciphers와 약한 키의 디버깅하기 위해, 다음 디버깅 설정을 켜자.

```
play.ws.ssl.debug = {
 ssl = true
 handshake = true
 verbose = true
 data = true
}
```

> **다음**: [[Configuring Certificate Validation|CertificateValidation]]
