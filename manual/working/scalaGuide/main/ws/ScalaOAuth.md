<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# OAuth

[OAuth](http://oauth.net/)는 보호된 데이터와 상호작용하는 간단한 방법이다. 또한 이 방법은 사용자가 접근할 수 있는 방법을 제공하는 안전하고 보안이 강화된 방법이다. 예를 들면, 사용자의 정보를 [Twitter](https://dev.twitter.com/docs/auth/using-oauth)에서 접근 할 수 있다.

OAuth는 [OAuth 1.0](http://tools.ietf.org/html/rfc5849)과 [OAuth 2.0](http://oauth.net/2/)인 두가지 서로 아주 다른 버전이 있습니다. 버전 2는 라이브러리나 다른 도움을 주는 기능 없이도 쉽게 만들기에 충분합니다. 그렇기 때문에 플레이는 OAuth 1.0에 대한 지원만을 제공한다.

## 사용법

OAuth를 사용하기 위해서는, 우선 `ws`를 `build.sbt`파일에 추가해야 한다.:

```scala
libraryDependencies ++= Seq(
  ws
)
```

## 필요한 정보

OAuth를 위해서는 애플리케이션을 서비스 제공자에게 등록해야 한다. 콜백 URL이 일치하지 않는다면 요청을 거절하기 때문에 URL을 올바르게 재공했는지를 확인해야 한다. 내부적으로만 사용하는 경우에는, 내부 장비의 `/etc/hosts`에 가상의 도메인을 제공해서 사용할 수 있다.

서비스 제공자는 아래와 같은 정보를 제공해 줄것이다.:

* 애플리케이션 ID
* 보안키
* 요청 토큰 URL
* 접근 토큰 URL
* 인증 URL

## 인증 절차

대부분의 절차는 플레이 라이브러리에 의해 처리된다.

1. (서버에서 서버로 요청)서버에서 요청 토큰을 받는다.
2. 사용자가 자신의 정보를 애플리케이션에게 사용할 수 있는 권한을 제공해줄 서비스 제공자로 이동시킨다. 
3. 서비스 제공자는 사용자를 다시 돌려보내 주고, /검증자/를 전달해 준다.
4. (서버에서 서버로 요청)검증자를 이용하여 /요청 토큰/을 /접근 토큰/으로 변환한다.

이제 보호된 정보에 접근하는 요청마다 /접근 토큰/을 사용할 수 있다.

## 예제

```scala
object Twitter extends Controller {

  val KEY = ConsumerKey("xxxxx", "xxxxx")

  val TWITTER = OAuth(ServiceInfo(
    "https://api.twitter.com/oauth/request_token",
    "https://api.twitter.com/oauth/access_token",
    "https://api.twitter.com/oauth/authorize", KEY),
    true)

  def authenticate = Action { request =>
    request.getQueryString("oauth_verifier").map { verifier =>
      val tokenPair = sessionTokenPair(request).get
      // 검증자 획득; 접근 토큰을 획득, 그것을 보관한 다음 인덱스로 돌아간다.
      TWITTER.retrieveAccessToken(tokenPair, verifier) match {
        case Right(t) => {
          // 인증 토큰을 OAuth 에서 획득 - 처리하기전에 우선 보관한다.
          Redirect(routes.Application.index).withSession("token" -> t.token, "secret" -> t.secret)
        }
        case Left(e) => throw e
      }
    }.getOrElse(
      TWITTER.retrieveRequestToken("http://localhost:9000/auth") match {
        case Right(t) => {
          // 인증되지 않은 토큰을 OAuth 에서 획득 - 처리하기전에 우선 보관한다.
          Redirect(TWITTER.redirectUrl(t.token)).withSession("token" -> t.token, "secret" -> t.secret)
        }
        case Left(e) => throw e
      })
  }

  def sessionTokenPair(implicit request: RequestHeader): Option[RequestToken] = {
    for {
      token <- request.session.get("token")
      secret <- request.session.get("secret")
    } yield {
      RequestToken(token, secret)
    }
  }
}
```

```scala
object Application extends Controller {

  def timeline = Action.async { implicit request =>
    Twitter.sessionTokenPair match {
      case Some(credentials) => {
        WS.url("https://api.twitter.com/1.1/statuses/home_timeline.json")
          .sign(OAuthCalculator(Twitter.KEY, credentials))
          .get
          .map(result => Ok(result.json))
      }
      case _ => Future.successful(Redirect(routes.Twitter.authenticate))
    }
  }
}
```
