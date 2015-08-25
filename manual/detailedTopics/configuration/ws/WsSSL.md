<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# WS SSL 환경설정하기

[[Play WS|ScalaWS]]는 코드를 작성할 필요 없이 환경설정 파일으로 부터 완벽하게 HTTPS를 설정할 수 있도록 한다. 이 환경설정 레이어는 자바 보한 소켓 확장(JSSE)를 적충하여 합리적인 기본값으로 이 작업을 수행한다.

JDK 1.8은 보안이 우선인 경우 이전 버전 보다 [훨씬 더 나은 JSSE 구현체](http://docs.oracle.com/javase/8/docs/technotes/guides/security/enhancements-8.html)를 포함하고 있다.

## 컨텐츠 항목

- [[Quick Start to WS SSL|WSQuickStart]]
- [[Generating X.509 Certificates|CertificateGeneration]]
- [[Configuring Trust Stores and Key Stores|KeyStores]]
- [[Configuring Protocols|Protocols]]
- [[Configuring Cipher Suites|CipherSuites]]
- [[Configuring Certificate Validation|CertificateValidation]]
- [[Configuring Certificate Revocation|CertificateRevocation]]
- [[Configuring Hostname Verification|HostnameVerification]]
- [[Example Configurations|ExampleSSLConfig]]
- [[Using the Default SSLContext|DefaultContext]]
- [[Debugging SSL Connections|DebuggingSSL]]
- [[Loose Options|LooseSSL]]
- [[Testing SSL|TestingSSL]]

## 더 읽어볼 것들

JSSE는 복잡한 상품이다. 더 편리하기 위해 JSSE는 다음을 지원한다.

JDK 1.8.

* [JSSE Reference Guide](http://docs.oracle.com/javase/8/docs/technotes/guides/security/jsse/JSSERefGuide.html)
* [JSSE Crypto Spec](http://docs.oracle.com/javase/8/docs/technotes/guides/security/crypto/CryptoSpec.html#SSLTLS)
* [SunJSSE Providers](http://docs.oracle.com/javase/8/docs/technotes/guides/security/SunProviders.html#SunJSSEProvider)
* [PKI Programmer's Guide](http://docs.oracle.com/javase/8/docs/technotes/guides/security/certpath/CertPathProgGuide.html)
* [keytool](http://docs.oracle.com/javase/8/docs/technotes/tools/unix/keytool.html)

JDK 1.7.

* [JSSE Reference Guide](http://docs.oracle.com/javase/7/docs/technotes/guides/security/jsse/JSSERefGuide.html)
* [JSSE Crypto Spec](http://docs.oracle.com/javase/7/docs/technotes/guides/security/crypto/CryptoSpec.html#SSLTLS)
* [SunJSSE Providers](http://docs.oracle.com/javase/7/docs/technotes/guides/security/SunProviders.html#SunJSSEProvider)
* [PKI Programmer's Guide](http://docs.oracle.com/javase/7/docs/technotes/guides/security/certpath/CertPathProgGuide.html)
* [keytool](http://docs.oracle.com/javase/7/docs/technotes/tools/solaris/keytool.html)

JDK 1.6.

* [JSSE Reference Guide](http://docs.oracle.com/javase/6/docs/technotes/guides/security/jsse/JSSERefGuide.html)
* [JSSE Crypto Spec](http://docs.oracle.com/javase/6/docs/technotes/guides/security/crypto/CryptoSpec.html#SSLTLS)
* [SunJSSE Providers](http://docs.oracle.com/javase/6/docs/technotes/guides/security/SunProviders.html#SunJSSEProvider)
* [PKI Programmer's Guide](http://docs.oracle.com/javase/6/docs/technotes/guides/security/certpath/CertPathProgGuide.html)
* [keytool](http://docs.oracle.com/javase/6/docs/technotes/tools/solaris/keytool.html)
