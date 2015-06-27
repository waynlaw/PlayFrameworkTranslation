<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 플레이에서 OpenID 지원하기

OpenID는 사용자들이 하나의 계정으로 여러 서비스를 접속할 수 있도록 하는 프로토콜이다. 웹 개발자로써, [Google account](https://developers.google.com/accounts/docs/OpenID)와 같이 사용자가 이미 가지고 있는 계정을 사용해서 로그인을 할 수 있도록 OpenID를 제공할 수 있다. 기업에서는 OpenID를 회사내의 SSO 서버에 연결하기위해 사용할 수 있다.

## nutshell의 OpenID 흐름

1, 사용자가 자신의 OpenID(URL)를 제공한다.
2. 서버가 사용자를 인증하기 위한 주소로 연결해주기 위해서, URL의 정보를 검사한다.
3. 사용자가 OpenID 제공자로부터 인증을 받은 다음, 다시 서버로 돌아온다.
4. 서버가 사용자를 다시 돌려보내준 곳으로부터 정보를 받고, 그 정보 제공자로부터 정보가 올바른지 확인한다.

만일 모든 사용자가 동일한 OpenID 제공자를 이용한다면, 첫번째 단계는 생략될 수 있다. (예를 들어, 완전히 구글 계정으로부터 정보를 받기로 결정하기로 한것과 같은 경우)

## 사용법

OpenID를 사용하기 위해서는, 우선 `ws`를 `build.sbt`파일에 추가해야 한다.:

```scala
libraryDependencies ++= Seq(
  ws
)
```

이제 어떤 OpenID를 사용하기를 원하는 컨트롤러나 구성요소에 [OpenIdClient](api/scala/index.html#play.api.libs.openid.OpenIdClient)에 대한 의존성을 선언해주어야 한다.:

@[의존성](code/ScalaOpenIdSpec.scala)

이후의 예제에서 생성한 `openIdClient`를 `OpenIdClient`라고 부를 것이다.

## 플레이에서의 OpenID

OpenID API는 두개의 중요한 함수를 가지고 있다.:

* `OpenIdClient.redirectURL`는 사용자를 보내야할 URL 주소를 계산한다. 그것은 사용자의 OpenID 페이지를 비동기로 가져오는 과정에 연관이 있으므로, `Future[String]`를 반환한다. 만일 OpenID가 잘못되었다면, 반환된 `Future`는 실패할 것이다.
* `OpenIdClient.verifiedId`는 `RequestHeader`를 필요로 하며, 검증된 OpenID와 사용자 정보를 확정하기 위한 조사를 필요로 한다. 정보의 신뢰성을 확인하기 위해서 OpenID 서버로 비동기 요청을 하고 [UserInfo](api/scala/index.html#play.api.libs.openid.UserInfo)의 Future를 반환할 것이다. 만일 정보가 올바르지 않거나, 서버의 확인이 자체가 거짓이라면(예를 들어 이동한 URL이 위조된 주소인 경우) 반한된 `Future`는 실패할 것이다.

만일 `Future`가 실패하면, 사용자를 로그인 페이지로 다시 보내거나 `BadRequest`를 반환하는 것과 같은 대안을 정의할 수 있다.

여기에 사용 예가 있다. (컨트롤러로 부터):

@[흐름](code/ScalaOpenIdSpec.scala)

## 추가 속성

사용자의 OpenID는 그의 신원을 제공해 줍니다. 이 프로토콜은 또한 이메일 주소나, 성, 이름과 같은 [추가 속성](http://openid.net/specs/openid-attribute-exchange-1_0.html)을 제공해 줍니다.

*선택적인* 속성과 *필수적인* 속성을 OpenID 서버에 요청할 수 있다. 필수적인 속성을 요청할 때 사용자가 그 정보들을 제공할 수 없다면 서비스에 로그인 할 수 없게된다.

이동할 URL에서 추가적인 속성은 요청을 할 수 있다.:

@[추가 속성](code/ScalaOpenIdSpec.scala)

속성은 OpenID 서버가 제공한 `UserInfo` 에서 얻을 수 있을 것이다.
