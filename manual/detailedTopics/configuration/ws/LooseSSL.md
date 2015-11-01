# 느슨한 옵션

## 우리는 이해했다

SSL을 설정하는 것은 재미가 없다. 무시무시한 경고와 지루한 암호기술 문서를 읽는 것, 몇몇 인증서를 설정하는 것을 포함하여 심지어 HTTPS로 단일 웹 서비스를 설정하는 것은 누군가에게 손해는 되지 않는다.

그럼에도 불구하고 SSL의 모든 보안 기능은 좋은 이유가 있고 심지어 개발자의 의도에 의해 종료시킬수 있고 적절하게 SSL을 설정하는 것보다 더 적은 재미로 이어진다.

## 어떤 것을 종료하기 전에 이것을 읽어보자

### 중간 공격자는 아주 잘 알고 있다

정보 보안 커뮤니티는 대다수의 내부 네트워크가 얼마나 안전하지 않은지와 자신의 장점을 사용하는 것에 대해서 아주 잘 알고 있다. 이 비디오는 [넓은 범위의 가능한 공격](http://2012.video.sector.ca/page/6)에 대해서 자세하게 논의한다.

### 중간 공격자는 일반적이다.

보통 회사는 연간 [중간 공격자](https://sites.google.com/site/cse825maninthemiddle/) 공격을 7 ~ 8회정도 될거라 예상할 수 있다. 몇몇의 경우 300,000이상의 사용자는 [매월마다](https://security.stackexchange.com/questions/12041/are-man-in-the-middle-attacks-extremely-rare) 손상을 입을 수 있다.

### 공격자는 자동화된 결함을 악용하는 도구들을 가지고 있다.

오늘날에 전문화된 해커는 없다. 대부분의 보안 관계자는 결함을 확인하기 위해 100개의 도구가 들어있고, 침투 테스팅을 수행하기 위해 Kali Linux같은 자동화된 리눅스 환경을 사용한다. [Cain & Abel](https://www.youtube.com/watch?v=pfHsRscy540)의 비디오는 비밀번호가 20초도 안되어서 해제되는 것을 보여준다.

해커는 어떤것이 "암호화 될 것인지" 아닌지 살펴보는 것에 지루함을 느낄것이다. 반면에, 해커들은 도구들로 머신을 설정하여 모든 가능한 공격을 수행하고 커피를 마시러 갈 것이다.

### 보안은 점점더 중요해지고 보편화 되고 있다.

더 많은 정보들이 매일 컴퓨터를 통해서 흐르고 있다. 대중과 언론은 그들의 비밀 통신이 차단될 수 있다는 가능성이 증가되고 있음을 신경쓰고 있다. 구글, 페이스북, 야후나 다른 선도적인 회사는 보안된 통신을 최우선으로 만들고 [데이터가 읽혀질 수 없다고](https://www.eff.org/deeplinks/2013/11/encrypt-web-report-whos-doing-what)확신하는 헌신적인 민중들을 가지고 있다.

### 이더넷 / 비밀번호 보호된 WiFi는 의미있는 수준의 보안을 제공하지 않는다.

와이파이 네트워크를 통해서 보내지는 모든 트래픽을 가져오는 [Wifi 파인애플](https://wifipineapple.com/)같은 네트워킹 검사 도구는 대략 $100정도 가격이며, 사람들이 이를 사용하여 [의도치 않은 트래픽 갈취](http://www.troyhunt.com/2013/04/the-beginners-guide-to-breaking-website.html)를 시작하기에 아주 좋은 도구이다.

### 회사들은 부적절한 보안으로 소송을 당할 수 있다.

PCI 협력체는 오직 회사를 걱정해야 하는 어떤 것만은 아니다. FTC는 그들이 정보를 신용카드 정보를 포함하여 안전하게 전송하는데 실패했다는 이유로 [Fandango and Credit Karma](http://www.ftc.gov/news-events/press-releases/2014/03/fandango-credit-karma-settle-ftc-charges-they-deceived-consumers)를 상대로 소송을 제기했다.

### 적절하게 환경설정된 HTTPS 클라이언트는 중요하다.

민감한 회사 기밀 정보도 웹 서비스를 통해 이동한다. WS 클라이언트에서 위험하다고 논의된 문서는 [The Most Dangerous Code in the World: Validating SSL Certificates in Non-Browser Software](http://www.cs.utexas.edu/~shmat/shmat_ccs12.pdf)이며, 목록의 초라한 기본 구성과 노출의 주요 원인으로 보안 옵션을 명시 적으로 비활성화한다. WS 클라이언트는 기본적으로 더 안전한전하게끔 구성되었으며, 여러분에게 유리하도록 구성방법을 제공한 예제가 있다.

## 완화

만약에 느슨한 옵션을 사용해야 한다면, 여러분의 노출을 최소화할 수 있는 여러가지 방법이 있다.

**커스텀 WS클라이언트**: `ConfigFactory.parseString`와 함께 [`DefaultWSConfigParser`](api/scala/index.html#play.api.libs.ws.DefaultWSConfigParser)을 사용하여 서버를 위해 명확하게 [[커스텀 WS클라이언트|ScalaWS]]를 생성할 수 있고, 컨텍스트에서 외부에 사용되지 않음을 보장한다.

**환경 스코핑**: 어떤 느슨한 옵션이 환경설정 파일에 하드코딩되지 않았음을 보장하고 개발 환경에서는 이를 적용하지 않기 위해[HOCON에서의 환경변수](https://github.com/typesafehub/config/blob/master/HOCON.md#substitution-fallback-to-environment-variables) 를 정의할 수 있다.

배포 스크립트나 프로그램에서 `ws.ssl.loose`옵션을 프로덕션 환경에서 비활성화하도록 확인하는 코드를 추가할 수 있다. 실행 모드에서는 [`Application.mode`](api/scala/index.html#play.api.Application) 메소드에서 찾을 수 있다.

## 느슨한 옵션

마지막으로 느슨한 옵션에 관련된 내용이다.

### 인증서 확인 비활성화

> **NOTE**: In most cases, people turn off certificate verification because they haven't generated certificates.  **There are other options besides disabling certificate verification.**
>
> * [[Quick Start to WS SSL|WSQuickStart]] shows how to connect directly to a server using a self signed certificate.
> * [[Generating X.509 Certificates|CertificateGeneration]] lists a number of GUI applications that will generate certificates for you.
> * [[Example Configurations|ExampleSSLConfig]] shows complete configuration of TLS using self signed certificates.
> * If you want to view your application through HTTPS, you can use [ngrok](https://ngrok.com/) to proxy your application.
> * If you need a certificate authority but don't want to pay money, [StartSSL](https://www.startssl.com/?app=1) or [CACert](http://www.cacert.org/) will give you a free certificate.
* If you want a self signed certificate and private key without typing on the command line, you can use [selfsignedcertificate.com](http://www.selfsignedcertificate.com/).

만엑에 위의 내용을 읽어봤지만 여전히 인증서 확인을 명백하게 비활성화하길 원한다면 다음과 같이 설정하자.

```
play.ws.ssl.loose.acceptAnyCertificate=true
```

인증서 확인을 완전하게 비활성화하면, [mitmproxy](http://mitmproxy.org/)와 같은 도구를 사용하여 네트워크상에서 누군가의 공격에 취약할 수 있다.

### 약한 Ciphers 확인 비활성화하기

여기 결점이 있다고 알려진 ciphers가 있고, 1.7에서는 [비활성화](http://sim.ivi.co/2011/08/jsse-oracle-provider-default-disabled.html)되었다. WS는 `ws.ssl.enabledCiphers` 목록에서 약한 chipher를 발견하면 예외를 던질 것이다. 만약 명확하게 약한 chiper를 사용하고 싶다면 다음과 같이 플래그를 설정하자.

```
play.ws.ssl.loose.allowWeakCiphers=true
```

약한 chipher 확인을 비활성화하면, [Flame](http://arstechnica.com/security/2012/06/flame-crypto-breakthrough/)과 같이 위조된 인증서를 사용하는 공격자의 공격에 취약할 수 있다.

### 호스트이름 확인 비활성화하기

호스트이름 확인을 비활성화하고 싶다면 다음과 같이 플래그를 설정하자.

```
play.ws.ssl.loose.disableHostnameVerification=true
```

With hostname verification disabled, a DNS proxy such as `dnschef` can [easily intercept communication](http://tersesystems.com/2014/03/31/testing-hostname-verification/).

호스트이름 확인을 비활성화 하면, `dnschef`와 같은 DNS 프록시는 [쉽게 대화내용을 탈취](http://tersesystems.com/2014/03/31/testing-hostname-verification/)당할 수 있다.

### 프로토콜 비활성화하기

WS는 많은 수의 [보안 이슈](https://www.schneier.com/paper-ssl.pdf)로 "SSLv3", "SSLv2"와 "SSLv2Hello"를 약한 프로토콜로 인식하며, `ws.ssl.enabledProtocols`목록에 있으면 예외를 던진다. 사실상 모든 서버가 `TLSv1`를 지원하므로, 이 오래된 프로토콜을 사용하는 것에 장점은 없다.

명백하게 약한 프로토콜을 원한다면, 다음과 같이 비활성화를 확인하는 플래그를 설정하자.


```
play.ws.ssl.loose.allowWeakProtocols=true
```

SSLv2와 SSLv2Hello (이들은 v1이 아니다)는 일반적으로 사용하고 있지 않고 필드가 [공용 인터넷의 25% 아래](https://www.trustworthyinternet.org/ssl-pulse/)인 사용방법이다. SSLv3는 TLS와 동일한 [보안 이슈](http://www.yaksman.org/~lweith/ssl.pdf)를 가지고 있다고 알려져있다. 레거시 서버에 연결하는 유일한 이유이지만 이렇게하면 그 자체로는 취약하지는 않다.

> **다음**:  [[SSL 테스트하기|TestingSSL]]
