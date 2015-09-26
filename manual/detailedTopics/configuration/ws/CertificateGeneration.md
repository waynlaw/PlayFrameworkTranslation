# X.509 인증서 생성하기

## X.509 인증서

공개 키 인증서는 신원 확인 문제를 위한 해결책이다. 단독 암호화는 보안 연결을 설정하기에 충분하지만, 여러분이 서버와 대화하는 것이 여러분이 생각하는 그 대화인지는 보장하지는 못한다. 원격 서버의 신원을 확인하는 어떤 방법 없이, 공격자는 여전히 스스로 원격서버를 잠잠하게 할 수 있고 원격서버에게로 보안 연결을 수행할 수 있다. 공개 키 인증서는 이런 문제를 해결한다.

공개키 인증서에 대해서 생각하는 올바른 방법은 여권 시스템이다. 인증서는 위조를 어렵게 하는 방법에 대한 정보로 운반자에 대한 정보를 발행하는데 사용할 수 있다. 이것이 인증서 확인이 중요한 이유이다. 아무 인증서나 승인하는 것은 심지어 공격자의 인증서도 마구잡이로 승인될 수 있음을 의미한다.

## 키도구 사용하기

키도구는 여러가지 버전이 있다.

* [1.8](http://docs.oracle.com/javase/8/docs/technotes/tools/unix/keytool.html)
* [1.7](http://docs.oracle.com/javase/7/docs/technotes/tools/solaris/keytool.html)
* [1.6](http://docs.oracle.com/javase/6/docs/technotes/tools/solaris/keytool.html)

아래 예제는 키도구 1.7을 사용하며 1.6은 CA 사용방법 또는 호스트이름을 위한 인증서를 만드는데 필요한 인증서 확장의 최소 요구사항을 지원하지 않는다.

## 랜덤 비밀번호 생성하기

랜덤 피밀번호는 pwgen을 사용하여 생성한다. (맥 사용자라면 `brew install pwgen`)

@[context](code/genpassword.sh)

## 서버 환경설정

호스트이름 확인을 위해 DNS 호스트이름을 적용한 서버가 필요할 것이다. 예제로 우리는 호스트이름은 `example.com`이라고 가정한다.

### 서버 CA 생성하기

첫 번째 단계는 example.com 인증서를 서명하기 위해 인증 기관을 생성하는 것이다. 최상위 CA 인증서는 추가적인 속성(ca:true, keyCertSign)을 가지며 명시적으로 CA 인증서를 만들고, 트러스트 스토어에 이를 보존한다.

@[context](code/genca.sh)

### example.com 인증서 생성하기

example.com 인증서는 `example.com` 서버와 핸드쉐이크하여 전달된다.

@[context](code/genserver.sh)

다음과 같은 내용을 보게 될 것이다.

```
Alias name: example.com
Creation date: ...
Entry type: PrivateKeyEntry
Certificate chain length: 2
Certificate[1]:
Owner: CN=example.com, OU=Example Org, O=Example Company, L=San Francisco, ST=California, C=US
Issuer: CN=exampleCA, OU=Example Org, O=Example Company, L=San Francisco, ST=California, C=US
```

> **주의**: 더 많은 정보를 위해 [[HTTPS 환경설정하기|ConfiguringHttps]] 섹션을 살펴보자.

### Nginx example.com 인증서 환경 설정하기

example.com이 TLS 종료 지점으로 자바를 사용하지 않고 nginx를 사용하고 있다면 PEM형식의 인증서로 전환할 필요가 있다.

불행하게도, 키도구는 비밀 키 정보를 전환할 수 없으며, 그래서 openssl이 비밀 키를 가져오기위해 설치되어야 한다.

@[context](code/genserverexp.sh)

이제 `example.com.crt`(the public key certificate)과 `example.com.key` (the private key)를 가지고 있으니 HTTPS 서버를 설정할 수 있다.

예를들어 nginx에서 이들을 사용하려면 `nginx.conf`에 다음과 같이 설정할 것이다.

```
ssl_certificate      /etc/nginx/certs/example.com.crt;
ssl_certificate_key  /etc/nginx/certs/example.com.key;
```

만약에 클라이언트 인증(아래의 **클라이언트 환경설정하기** 에서 다룬다)을 사용한다면, 또한 아래 내용도 추가할 필요가 있다.

```
ssl_client_certificate /etc/nginx/certs/clientca.crt;
ssl_verify_client on;
```

서버에서 확인하는 것으로 인증서가 여러분이 예상하는 것인지 확인할 수 있다.

```
keytool -printcert -sslserver example.com
```

> **주의**: 더 많은 정보를 위해 [[프론트엔드 HTTP서버 설정하기|HTTPServer]] 섹션을 살펴보자.

## 클라이언트 환경설정하기

클라이언트를 설정하는 것은 트러스트 스토어 환경설정하기, 클라이언트 인증 환경설정하기 2가지로 나뉘어있다.

### 트러스트 스토어 환경설정하기

어떤 클라이언트는 서버의 example.com 인증서가 신뢰할만한 것인지 살펴볼 필요가 있다. 그러나 비밀키는 살펴볼 필요가 없다. 클라이언트의 외부에서 인증서를 포함하여 트러스트 스토어를 생성하자. 대부분의 자바 클라이언트는 JKS 형식의 트러스트 스토어를 가지고 있는 것을 선호한다. 

@[context](code/gentruststore.sh)

examplecs를 위해 `trustedCertEntry`를 살펴보아야 한다.

```
Alias name: exampleca
Creation date: ...
Entry type: trustedCertEntry

Owner: CN=exampleCA, OU=Example Org, O=Example Company, L=San Francisco, ST=California, C=US
Issuer: CN=exampleCA, OU=Example Org, O=Example Company, L=San Francisco, ST=California, C=US
```

`exampletrust.jks` 스토어는 트러스트관리자에서 유용하다.

```
play.ws.ssl {
  trustManager = {
    stores = [
      { path = "/Users/wsargent/work/ssltest/conf/exampletrust.jks" }
    ]
  }
}
```

> **주의**: 더 많은 정보를 위해 [[Configuring Key Stores and Trust Stores|KeyStores]] 섹션을 살펴보자.

### 클라이어늩 인증 설정하기

클라이언트 인증은 불분명하고 서투르게 문서화될 가능성이 있지만 다음 단계를 필요로한다.

1. 서버는 클라이언트 인증서로 서명 할 것으로 예상하여 CA를 제시하고, 클라이언트 인증서를 요청한다. 이 경우 `CN=clientCA`이다. ([디버깅 예제](http://docs.oracle.com/javase/7/docs/technotes/guides/security/jsse/ReadDebug.html)를 살펴보자)
2. 클라이언트는`chooseClientAlias`와 `certRequest.getAuthorities`를 사용하여 `clientCA`로 서명한 인증서를 KeyManager에서 찾는다.
3. 키관리자는 서버에게 `client` 인증서를 반환한다.
4. 서버는 핸드쉐이크로 추가적인 ClientKeyExchange를 한다.

클라이언트 CA를 생성하고 클라이언트 인증서를 서명하는 단계는 대략적으로 서버의 인증서 생성과 유사하지만 단일 스크립트로 제공되어 편리하다.

@[context](code/genclient.sh)

`client`가 필요한데, 다음과 같이 살펴볼 수 있다.

```
Your keystore contains 1 entry

Alias name: client
Creation date: ...
Entry type: PrivateKeyEntry
Certificate chain length: 2
Certificate[1]:
Owner: CN=client, OU=Example Org, O=Example Company, L=San Francisco, ST=California, C=US
Issuer: CN=clientCA, OU=Example Org, O=Example Company, L=San Francisco, ST=California, C=US
```

그리고 키 관리자에 `client.jks`를 집어넣자.

```
play.ws.ssl {
  keyManager = {
    stores = [
      { type = "JKS", path = "conf/client.jks", password = $PW }
    ]
  }
}
```

> **주의**: 더 많은 정보를 위해 [[키 스토어와 트러스트 스토어 환경설정하기|KeyStores]] 섹션을 살펴보자.

## 인증서 관리 도구

명령어 라인 도구보다 그래픽 도구로 인증서를 검사하고 싶다면 [Keystore Explorer](http://keystore-explorer.sourceforge.net/)나 [xca](http://sourceforge.net/projects/xca/)를 사용할 수 있다. [Keystore Explorer](http://keystore-explorer.sourceforge.net/)는 특히 JKS 형식을 살펴보는 것이 아주 편리하다. 직접 설치하는 것이 잘 동작하고, export 정책에 대해 일부 조정이 필요하다.

키도구보다 더 복잡한 명령어 라인 도구를 사용하고 싶다면, JKS나 멀티파트 PEM 형식의 인증서를 이해하는 [java-keyutil](https://code.google.com/p/java-keyutil/)를 사용할 수 있다.

## 인증서 설정

### Secure

최상의 보안을 원한다면 서명 알고리즘 (키도구에서, `-sigalg EC`일 것이다.)으로 [ECDSA](http://blog.cloudflare.com/ecdsa-the-digital-signature-algorithm-of-a-better-internet)를 사용하는 것을 고려해보자. ECDSA는 또한 "ECC SSL 인증서"로 알려져 있다.

### Compatible

이전 시스템과의 호환성을 위해 서명 알고리즘으로 2048비트 키의 RSA나 SHA256을 사용하자. 만약 스스로 CA 인증서를 생성할 것이라면, 최상위로 4096비트를 사용하자.

## 참고자료

* [JSSE Reference Guide To Creating KeyStores](http://docs.oracle.com/javase/8/docs/technotes/guides/security/jsse/JSSERefGuide.html#CreateKeystore)
* [Java PKI Programmer's Guide](http://docs.oracle.com/javase/7/docs/technotes/guides/security/certpath/CertPathProgGuide.html)
* [Fixing X.509 Certificates](http://tersesystems.com/2014/03/20/fixing-x509-certificates/)


