<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 증명서 유효성 검증 환경설정하기

SSL 연결에서 원격 서버의 신원은 인증 기관에 의해 서명된 X.509 인증서를 사용하여 확인된다.

X.509 인증서의 JSSE 구현체는 [PKI 프로그래머 가이드](http://docs.oracle.com/javase/7/docs/technotes/guides/security/certpath/CertPathProgGuide.html)에 정의되어 있다.

서버에서 사용되는 몇몇 X.509 인증서는 오래되었거나 공격자에 의해 위조될수 있는 서명을 사용하고 있다. 이 때문에, 서명 알고리즘이 사용되었었다면 서버의 신원은 확인할수 없을 수도 있다. 운이 좋게도, 이는 극히 드문 경우이며, 신뢰할 수 있는 리프 인증서의 95%, 신뢰할수 있는 서명된 인증서의 95프로 이상이 [NIST 권장된 키 사이즈](http://csrc.nist.gov/publications/nistpubs/800-131A/sp800-131A.pdf) 사이즈를 사용하고 있다.

WS는 [현재 표준](http://sim.ivi.co/2012/04/nist-security-strength-time-frames.html) 여러분을 위해 자동으로 약한 키와 약한 서명 알고리즘을 비활성화한다.

이 기능은 [jdk.certpath.disabledAlgorithms](http://sim.ivi.co/2013/11/harness-ssl-and-jsse-key-size-control.html)과 유사하지만, WS 클라이언트에 따라 다르고 동적으로 설명할 수 있다, 반면 jdk.certpath.disabledAlgorithms는 JVM에서 전역 접근되며 보안 프로퍼티로만 설정될 수 있고 오직 JDK 1.7과 그 이상에서만 사용가능하다.

여러분의 입맛대로 이를 오버라이드할 수 있지만 최소의 기본 값으로 엄격하게 하는게 좋다. 적절한 서명 이름은 [제공자 문서](http://docs.oracle.com/javase/7/docs/technotes/guides/security/SunProviders.html)에서 살펴볼 수 있다.

## 약한 서명 알고리즘의 인증서 비활성화하기

비활성된 서명 알고리즘의 기본 목록은 아래에 정의되어있다.

```
play.ws.ssl.disabledSignatureAlgorithms = "MD2, MD4, MD5"
```

MD5는 모질라의 추천과 검증 된 충돌 공격(collision attack)을 기반으로 비활성화되었다.

> MD5 certificates may be compromised when attackers can create a fake cert that hashes to the same value as one with a legitimate signature, and is hence trusted. Mozilla can mitigate this potential vulnerability by turning off support for MD5-based signatures. The MD5 root certificates don't necessarily need to be removed from NSS, because the signatures of root certificates are not validated (roots are self-signed). Disabling MD5 will impact intermediate and end entity certificates, where the signatures are validated.
>
> The relevant CAs have confirmed that they stopped issuing MD5 certificates. However, there are still many end entity certificates that would be impacted if support for MD5-based signatures was turned off in 2010. Therefore, we are hoping to give the affected CAs time to react, and are proposing the date of June 30, 2011 for turning off support for MD5-based signatures. The relevant CAs are aware that Mozilla will turn off MD5 support earlier if needed.

SHA-1은 약하다고 간주되며, 새로운 인증서는 [SHA-2 family](https://en.wikipedia.org/wiki/SHA-2)로 digest 알고리즘을 사용해야 한다. 그러나 오래된 인증서는 여전히 유효하게 간주되어야 한다.

## 약한 키 사이즈의 인증서 비활성화하기

WS는 다음과 같이 약한 키 사이즈의 기본 목록을 정의한다.

```
play.ws.ssl.disabledKeyAlgorithms = "DHE keySize < 2048, ECDH keySize < 2048, ECDHE keySize < 2048, RSA keySize < 2048, DSA keySize < 2048, EC keySize < 224"
```

이 설정은 모질라의 추천의 일부와 [keylength.com](https://keylength.com)의 일부를 기반으로 설정되었다.

> The NIST recommendation is to discontinue 1024-bit RSA certificates by December 31, 2010. Therefore, CAs have been advised that they should not sign any more certificates under their 1024-bit roots by the end of this year.
>
> The date for disabling/removing 1024-bit root certificates will be dependent on the state of the art in public key cryptography, but under no circumstances should any party expect continued support for this modulus size past December 31, 2013. As mentioned above, this date could get moved up substantially if new attacks are discovered. We recommend all parties involved in secure transactions on the web move away from 1024-bit moduli as soon as possible.

**주의:** 약한 키 사이즈는 또한 최상위 인증서에 적용되기 때문에, 이 옵션을 설정하는 것은 기본 값을 포함하여 트러스트 관리자나 키 관리자를 설정한 승인된 관계자가 확인해야 한다.

신뢰할 수 있는 리프 인증서의 95%, 신뢰할수 있는 서명된 인증서의 95프로 이상이 [NIST 권장된 키 사이즈](http://csrc.nist.gov/publications/nistpubs/800-131A/sp800-131A.pdf) 사이즈를 사용하고 있고, 기본적으로 안전하다고 간주된다.

## 약한 전역 인증서 비활성화하기

서명 알고리즘과 JVM에 전역적으로 약한 키사이즈를 비활성화 하기 위해 `jdk.certpath.disabledAlgorithms` [보안 프로퍼티](http://sim.ivi.co/2011/07/java-se-7-release-security-enhancements.html)를 사용할 수 있다. 보안 프로퍼티를 설정하는 것은 [[Configuring Cipher Suites|CipherSuites]] 섹션에서 더 깊이 있게 다룬다.

> **주의**이를 이를 설정하면 `jdk.certpath.disabledAlgorithms`프로퍼티는 `disabledKeyAlgorithms` and `disabledSignatureAlgorithms`의 두 설정으로부터 환경설정을 포함시켜야 한다.

## 인증서 유효성 검증 디버깅하기

인증서 유효성 검증의 자세한 내역을 보기위해 다음과 같이 디버깅에 대해 환경설정할 수 있다.

```
play.ws.ssl.debug.certpath = true
```

문서화되지 않은 설정 `-Djava.security.debug=x509`도 또한 도움이 될 수 있다.

## 참고자료

* [Dates for Phasing out MD5-based signatures and 1024-bit moduli](https://wiki.mozilla.org/CA:MD5and1024)
* [Fixing X.509 Certificates](http://tersesystems.com/2014/03/20/fixing-x509-certificates/)
