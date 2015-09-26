# 환경설정 예제

TLS는 매우 혼란스러울 수 있다. 여기 이를 도와줄 수 있는 몇가지 설정이 있다.

## 내부의 웹 서비스로 연결하기

최신 TLS 구현체로 설정되어있는 단일 내부 웹 서비스와 대화하기 위해 WS를 사용한다면 외부의 CA를 사용할 필요는 없다. 내부 인증서는 잘 동작할 것이며, 아마 틀림없이 CA 시스템보다 [더 안전](http://www.thoughtcrime.org/blog/authenticity-is-broken-in-ssl-but-your-app-ha/)할 것이다.

[[인증서 생성하기|CertificateGeneration]]섹션으로부터 스스로 서명된 인증서를 생성하고, 클라이언트에게 CA의 공개 인증서를 신뢰할 수 있는지 물어보자.

```
play.ws.ssl {
  trustManager = {
    stores = [
      { type = "JKS", path = "exampletrust.jks" }
    ]
  }
}
```

## 클라이언트 인증과 내부 웹 서비스 연결하기

클라이언트 인증을 사용한다면 비킬키와 적절한 공개 키를 포함하고 있는 X.509 인증서로 구성된 PrivateKeyEntry를 포함한 키 관리자를 키 스토어에 포함시킬 필요가 있다. [[인증서 생성하기|CertificateGeneration]] 섹션에 있는 "클라이언트 인증 설정하기"를 살펴보자.

```
play.ws.ssl {
  keyManager = {
    stores = [
      { type = "JKS", path = "client.jks", password = "changeit1" }
    ]
  }
  trustManager = {
    stores = [
      { type = "JKS", path = "exampletrust.jks" }
    ]
  }
}
```

## 몇몇의 외부 웹 서비스 연결하기

몇몇의 외부 웹 서비스와 대화한다면, 여러 스토어를 하나의 클라이언트로 설정하면 더 편리할 수 있다.

```
play.ws.ssl {
  trustManager = {
    stores = [
      { type = "PEM", path = "service1.pem" }
      { path = "service2.jks" }
      { path = "service3.jks" }
    ]
  }
}
```

클라이언트 인증이 필요하다면, 여러 스토어에 키 관리자를 설정할 수 있다.

```
play.ws.ssl {
    keyManager = {
    stores = [
      { type: "PKCS12", path: "keys/service1-client.p12", password: "changeit1" },
      { type: "PKCS12", path: "keys/service2-client.p12", password: "changeit2" },
      { type: "PKCS12", path: "keys/service3-client.p12", password: "changeit3" }
    ]
  }
}
```

## 비밀과 공개 서버

만약에 동일한 프로파일로 비공개와 공개 서버에 접근하기 위해 WS를 사용한다면 적절한 기본 JSSE 트러스트 스토어를 포함하시키는 것을 원할지도 모르겠다.

```
play.ws.ssl {
  trustManager = {
    stores = [
      { path: exampletrust.jks }     # Added trust store
      { path: ${java.home}/lib/security/cacerts } # Fallback to default JSSE trust store
    ]
  }
}
```


