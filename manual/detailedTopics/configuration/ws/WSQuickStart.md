# WS SSL를 위한 빠른 시작

이 섹션은 HTTPS로 원격 웹 서비스를 연결해야 하는 사람들을 위한 것이며, 전체 메뉴얼을 읽고 싶지 않은 사람들을 위한 것이다. 만약 웹 서비스를 설정할 필요가 있거나 클라이언트 인증을 설정해야 한다면 [[다음 섹션|CertificateGeneration]]을 살펴보기 바란다.

## HTTPS를 통해서 원격 서버 연결하기

만약 원격 서버가 아주 잘 알려진 인증 기관에 의해서 서명된 인증을 사용한다면, WS는 어떤 추가적인 환경설정 없이도 외부에서 잘 동작할 것이다.

만약 웹 서비스가 잘 알려진 인증 기관을 사용하지 않는다면, 비밀 CA나 자가서명된 인증서를 사용할 것이다. curl을 사용하는 것으로 이를 쉽게 확인할 수 있다.

```
curl https://financialcryptography.com # uses cacert.org as a CA
```

만약에 다음 에러를 받게 된다면 CA의 인증서를 획득하여 트러스트 스토어에 추가해야 한다.

```
curl: (60) SSL certificate problem: Invalid certificate chain
More details here: http://curl.haxx.se/docs/sslcerts.html

curl performs SSL certificate verification by default, using a "bundle"
 of Certificate Authority (CA) public keys (CA certs). If the default
 bundle file isn't adequate, you can specify an alternate file
 using the --cacert option.
If this HTTPS server uses a certificate signed by a CA represented in
 the bundle, the certificate verification probably failed due to a
 problem with the certificate (it might be expired, or the name might
 not match the domain name in the URL).
If you'd like to turn off curl's verification of the certificate, use
 the -k (or --insecure) option.
```

## 최상위 CA 인증서 얻기

이상적으로 이 대역에서 수행해야 한다. 웹 서비스의 소유자는 위조 할 수없는 방식으로 적절한 사람에게 직접 최상위 CA 인증서를 제공해야 한다.

의사소통이 없는 이런 경우(추천되는 방법은 아니다), 때때로 [`keytool from JDK 1.8`](http://docs.oracle.com/javase/8/docs/technotes/tools/unix/keytool.html)를 사용하여 인증서 체인에서 직접 최상위 CA 인증서를 얻을 수 있다.

```
keytool -printcert -sslserver playframework.com
```

최상위 인증서로 #2를 반환한다.

```
Certificate #2
====================================
Owner: CN=GlobalSign Root CA, OU=Root CA, O=GlobalSign nv-sa, C=BE
Issuer: CN=GlobalSign Root CA, OU=Root CA, O=GlobalSign nv-sa, C=BE
```

내보낼 수 있는 형식으로 인증서 체인을 얻기 위해 -rfc 옵션을 사용하자.

```
keytool -printcert -sslserver playframework.com -rfc
```

이는 PEM 형식으로 인증의 시리얼을 반환할 것이다.

```
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
```

이는 파일으로 복사와 붙여넣기 할 수 있다. 체인에서 제일 마지막 인증은 최상위 CA 인증이 될 것이다.

> **주의**: 모든 웹사이트가 최상위 CA 인증서를 포함하고 있는 것은 아니다. 올바른 인증서인지 확신하기 위해 키 도구나 [certificate decoder](https://www.sslshopper.com/certificate-decoder.html)로 인증서를 복호화해야한다.

## PEM 파일에 트러스트 관리자 가리키기

명확하게 `PEM` 형식을 명시하기 위해 다음을 `conf/application.conf`에 추가하자.

```
play.ws.ssl {
  trustManager = {
    stores = [
      { type = "PEM", path = "/path/to/cert/globalsign.crt" }
    ]
  }
}
```

이는 트러스트 관리자에게 기본 인증서의 `cacerts` 스토어를 무시하라고 이야기하며, 오직 커스텀 CA 인증서를 사용한다.

이후에 WS는 설정되어야 하며, 여러분의 연결이 동작하는지 테스트할 수 있다.

```
WS.url("https://example.com").get()
```

[[예제 환경설정하기|ExampleSSLConfig]] 페이지에서 더 많은 예제를 살펴볼 수 있다. 
