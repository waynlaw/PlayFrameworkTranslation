<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 인증서 폐지 설정하기

JSSE에서 인증서 폐지는 인증서 폐기 리스트(CRLs)와 OCSP 2가지 방법을 통해 할 수 있다.

인증서 폐기는 서버의 [Heartbleed](http://heartbleed.com)에서 비공개 키가 제대로 동작하지 않는 상황에서 매우 유용할 수 있다.

인증서 폐기는 JSSE에서 기본적으로 비활성화된다. 2가지 방법이 정의되어있다.

* [PKI Programmer's Guide, Appendix C](http://docs.oracle.com/javase/6/docs/technotes/guides/security/certpath/CertPathProgGuide.html#AppC)
* [Enable OCSP Checking](https://blogs.oracle.com/xuelei/entry/enable_ocsp_checking)

OCSP를 활성화하기 위해, 다음 시스템 프로퍼티를 명령어 라인에서 설정해야 한다.

```
java -Dcom.sun.security.enableCRLDP=true -Dcom.sun.net.ssl.checkRevocation=true
```

위의 명령을 수행한 이후에 클라이언트에서 인증서 폐기를 활성화할 수 있다.

```
play.ws.ssl.checkRevocation = true
```

`checkRevocation`을 설정하는 것은 자동으로 내부의 `ocsp.enable` 보안 프로퍼티를 설정할 것이다.

```
java.security.Security.setProperty("ocsp.enable", "true")
```

그리고 이는 HTTPS 요청을 생성할 때 OCSP 확인하는 것을 설정한다.

> 주의 : OCSP를 활성화하는 것은 OCSP 응답자로 주고 받아져야 한다. 이는 확실하게 HTTP 호출의 오버헤드를 주게 되어, 호출을 만드는데 [33% 감소](http://blog.cloudflare.com/ocsp-stapling-how-cloudflare-just-made-ssl-30)가 있을 수 있다. 완화 기술, OCSP 스템플링은 JSSE에서 지원되지 않는다.

또는 정적 CRL 목록을 사용하고 싶다면, URL의 목록을 정의할 수 있다.

```
play.ws.ssl.revocationLists = [ "http://example.com/crl" ]
```

## 디버깅하기

인증서 폐기가 활성화되었는지 테스트하기 위해 다음 옵션을 설정하자.

```
play.ws.ssl.debug = {
 certpath = true
 ocsp = true
}
```

그러면 출력에서 다으모가 같은 것들을 볼 수 있을 것이다.

```
certpath: -Using checker7 ... [sun.security.provider.certpath.RevocationChecker]
certpath: connecting to OCSP service at: http://gtssl2-ocsp.geotrust.com
certpath: OCSP response status: SUCCESSFUL
certpath: OCSP response type: basic
certpath: Responder's name: CN=GeoTrust SSL CA - G2 OCSP Responder, O=GeoTrust Inc., C=US
certpath: OCSP response produced at: Wed Mar 19 13:57:32 PDT 2014
certpath: OCSP number of SingleResponses: 1
certpath: OCSP response cert #1: CN=GeoTrust SSL CA - G2 OCSP Responder, O=GeoTrust Inc., C=US
certpath: Status of certificate (with serial number 159761413677206476752317239691621661939) is: GOOD
certpath: Responder's certificate includes the extension id-pkix-ocsp-nocheck.
certpath: OCSP response is signed by an Authorized Responder
certpath: Verified signature of OCSP Response
certpath: Response's validity interval is from Wed Mar 19 13:57:32 PDT 2014 until Wed Mar 26 13:57:32 PDT 2014
certpath: -checker7 validation succeeded
```

## 참고자료

* [Fixing Certificate Revocation](http://tersesystems.com/2014/03/22/fixing-certificate-revocation/)
