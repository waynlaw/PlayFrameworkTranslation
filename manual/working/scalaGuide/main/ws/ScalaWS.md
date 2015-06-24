<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 플레이 WS API

때로는 다른 HTTP 서비스들을 플레이 애플리케이션내에서 호출하려 할 수 있다. 플레이는 이러한 기능을 비동기 HTTP 호출을 제공하는 [WS library](api/scala/index.html#play.api.libs.ws.package)를 통해서 지원한다.

WS API를 사용하는데 중요한 두가지 항목이 있다.: 요청을 만들고, 응답을 처리하는 것이다. 우리는 GET과 POST HTTP 요청 모두에 대해서 우선 알아보고, 이후에 WS에서 응답을 처리하는 방법에 대해 알아볼 것이다. 그리고 최종적으로는 일반적인 사용 방법에 대해 논의할 것이다.

## 요청 만들기

WS를 사용하기 위해서는 `build.sbt` 파일에 `ws`를 추가해야 한다.:

```scala
libraryDependencies ++= Seq(
  ws
)
```

WS를 사용하려는 어떤 컨트롤러나 구성요소가 있다면, `WSClient`에 대한 의존성을 정의해야 한다.:

@[의존성](code/ScalaWSSpec.scala)

아래의 모든 예제에서, 우리는 만들어진 `WSClient`를 `ws`라 지칭할 것이다.

HTTP 요청을 만들기 위해서는, URL을 지정하기 위해 `ws.url()`을 사용해야 한다.

@[simple-holder](code/ScalaWSSpec.scala)

이것은 헤더 설정과 같은 다양한 HTTP 옵션을 설정하는데 사용하는 [WSRequestHolder](api/scala/index.html#play.api.libs.ws.WS$$WSRequestHolder)을 반환할 것이다. 복잡한 요청을 하기 위해서 연달아 호출할 수 있다.

@[complex-holder](code/ScalaWSSpec.scala)

사용을 원하는 HTTP 함수에 대응되는 함수를 호출하여 종료할 수 있다. 이것은 연결을 마치고, `WSRequestHolder`에 정의된 모든 옵션을 사용한다.

@[holder-get](code/ScalaWSSpec.scala)

이것은 서버에서 받은 데이터를 담고있는 [Response](api/scala/index.html#play.api.libs.ws.WSResponse)가 있는 `Future[WSResponse]`를 반환한다.

### 인증 요청하기

만일 HTTP인증을 사용하기를 원한다면, 빌더내에 사용자 이름과 암호, [AuthScheme](api/scala/index.html#play.api.libs.ws.WSAuthScheme)를 정의하여야 한다. AuthScheme을 위해 검증된 케이스 오브젝트들은 `BASIC`, `DIGEST`, `KERBEROS`, `NONE`, `NTLM`, `SPNEGO` 이다.

@[auth-request](code/ScalaWSSpec.scala)

### 리다이렉트를 따르는 요청하기

만일 HTTP 요청의 결과가 302나, 301 리다이렉트라면, 다른 호출을 하지않고 자동으로 리다이렉트를 따를 수 있다.

@[redirects](code/ScalaWSSpec.scala)

### 요청 인자들과 함께 요청하기

인자들은 키와 값의 튜플로 정의될 수 있다.

@[query-string](code/ScalaWSSpec.scala)

### 추가적인 헤더와 함께 요청하기

헤더는 키와 값의 쌍으로 정의될 수 있다.

@[headers](code/ScalaWSSpec.scala)

특정한 형식으로 일반 텍스트를 보낸다면, 내용의 종류를 명시적으로 지정하고 싶을 수 있다.

@[content-type](code/ScalaWSSpec.scala)

### 가상 호스트로 요청하기

가상 호스트는 문자열로 정의할 수 있다.

@[virtual-host](code/ScalaWSSpec.scala)

### 시간 제한과 함께 요청하기

만일 시간제한을 지정하기를 원한다면 `withRequestTimeout`를 밀리초 값으로 지정할 수 있다.

@[request-timeout](code/ScalaWSSpec.scala)

### 폼 데이터 제출하기

URL폼 형식의 데이터를 보내려면 `post` 내에 `Map[String, Seq[String]]`를 전달해야 한다.

@[url-encoded](code/ScalaWSSpec.scala)

### JSON 데이터 제출

JSON 데이터를 전달하는 가장 쉬운 방법은 [[JSON|ScalaJson]] 라이브러리를 사용하는 것이다.

@[scalaws-post-json](code/ScalaWSSpec.scala)

### XML 데이터 제출하기

XML 데이터를 전송하는 가장 쉬운 방법은 XML 리터럴들을 사용하는 것이다. XML 리터럴들은 편리하지만 아주 빠른것은 아니다. 효과적인 사용을 위해서는 XML 뷰 템플릿이나, JAXB 라이브러리 사용을 고려해 보아야 할것이다.

@[scalaws-post-xml](code/ScalaWSSpec.scala)

## 응답 처리하기

[Response](api/scala/index.html#play.api.libs.ws.Response)을 사용하는 것은 [Future](http://www.scala-lang.org/api/current/index.html#scala.concurrent.Future) 내에서 쉽게 처리될 것이다.

아래에 주어진 예제에서는 간결함을 위해서 일반적인 의존성을 한번만 보여줄 것이다.

`Future`에서 작업이 끝나게 되면, 암묵적인 실행 문맥이 사용가능해진다. 이것은 콜백이 실행되도록 지정된 쓰레드풀에서 동작한다. 대부분은 기본적인 플레이 실행 문맥으로도 충분하다.:

@[scalaws-context](code/ScalaWSSpec.scala)

아래의 예제는 케이스 클래스의 직렬화/역직렬화를 보여준다.:

@[scalaws-person](code/ScalaWSSpec.scala)

### JSON으로 응답 처리하기

`response.json`을 호출하여 응답을 [JSON object](api/scala/index.html#play.api.libs.json.JsValue) 로 처리할 수 있다.

@[scalaws-process-json](code/ScalaWSSpec.scala)

JSON 라이브러리는 암시적인 [`Reads[T]`](api/scala/index.html#play.api.libs.json.Reads)을 class로 변경해주는 [[유용한 기능|ScalaJsonCombinators]]을 가지고 있다.:

@[scalaws-process-json-with-implicit](code/ScalaWSSpec.scala)

### XML로 응답 처리하기

`response.xml`을 호출하여 응답을 [XML literal](http://www.scala-lang.org/api/current/index.html#scala.xml.NodeSeq)로 처리할 수 있다.

@[scalaws-process-xml](code/ScalaWSSpec.scala)

### 큰 응답 처리하기

`get()` 또는 `post()`를 호출하게 되면 응답이 이용할 수 있게 되기 전에, 요청의 몸체가 메모리에 올라가게 된다. 만일 크고 수 기가바이트의 파일을 내려받는다면, 원하지 않는 가비지 컬렉션이나, 메모리 부족 에러를 보게 될 것이다.

`WS`는 [[iteratee|Iteratees]]를 사용하여 응답을 순회하며 차례대로 처리할 수 있게 해준다. `WSRequestHolder`의 `stream()`과 `getStream()`함수는 `Future[(WSResponseHeaders, Enumerator[Array[Byte]])]`을 반환한다. 이 Enumerator는 응답의 내용을 담고있다.

여기에 반환된 응답의 바이트 수를 세는 예제가 있다.:

@[stream-count-bytes](code/ScalaWSSpec.scala)

일반적으로는 이러한 방식으로 큰 데이터를 처리하기를 원하지는 않을 것이다. 더 일반적인 사용 방식은 응답의 내용을 다른 장소에 전달하는 것이다. 예를 들어, 데이터를 파일에 쓰려고 하는 경우를 보자.:

@[stream-to-file](code/ScalaWSSpec.scala)

다른 일반적인 사용은 응답의 내용을 다른 요청의 응답으로 제공해 주는 것이다.:

@[stream-to-result](code/ScalaWSSpec.scala)

`POST`와 `PUT`은 수동으로 `withMethod`함수를 호출해 주어야 한다. 예:

@[stream-put](code/ScalaWSSpec.scala)

## 일반적인 패턴과 사용 예

### WS 호출을 연이어 하기

테스트에 사용하는 것은 신뢰할 수 있는 환경에서 WS 호출을 연이어 하는 좋은 방법이다. 가능한 실패를 다루기 위해서 [Future.recover](http://www.scala-lang.org/api/current/index.html#scala.concurrent.Future)와 함께 테스트를 사용해야 한다.

@[scalaws-forcomprehension](code/ScalaWSSpec.scala)

### 컨트롤러 내에서 사용하기

컨트롤러에서 요청을 만드는 경우에는, 응답을 `Future[Result]`로 변환할 수 있다. 이것은 [[비동기 결과들 다루기|ScalaAsync]]에 나온 것처럼, 플레이의 `Action.async` 액션 생성기와 함께 사용할 수 있다.

@[비동기-결과](code/ScalaWSSpec.scala)

## WSClient 사용하기

WSClient는 AsyncHttpClient를 감싸고 있다. 이것은 서로다른 프로파일이나 가상화와 함께 여러개의 클라이언트를 정의하는데 유용하다.

WS를 이용한 주입 없이, 코드에서 바로 WSClient를 정의할 수 있다. 그리고 그것을 `WS.clientUrl()`와 함께 암시적으로 사용할 수 있다. 보안 TLS 설정을 위해 클라이언트를 설정할 때에는 항상 `NingAsyncHttpClientConfigBuilder`를 사용해야 하는것에 주의해야 한다.:

@[암시적-클라이언트](code/ScalaWSSpec.scala)

> 주의: 만일 NingWSClient 오브젝트를 생성한다면, 그것은 WS 모듈의 생명주기를 사용하지 않을 것이며, `Application.onStop`에서 자동으로 닫지 않을 것이다. 대신 작업이 끝났을 때, 반드시 클라이언트는 수동으로 `client.close()`를 불러 종료해야 한다. 그러면 AsyncHttpClient에 의해 사용된 ThreadPoolExecutor를 해제할 것이다. 닫는것에 실패하게 되면 결국 메모리 부족 예외로 이어질 수 있다. (개발 환경에서 애플리케이션을 자주 재시동하는 경우에는 특히 그렇다.)

또는 직접적으로:

@[direct-client](code/ScalaWSSpec.scala)

또는 특정한 클라이언트를 자동으로 결합하기 위해서, 마그넷 패턴을 사용할 수 있다.:

@[pair-magnet](code/ScalaWSSpec.scala)

기본적으로는 설정은 `application.conf`에서 이루어진다. 하지만 설정에서 빌더를 직접적으로 설정할 수 있다.:

@[programmatic-config](code/ScalaWSSpec.scala)

[async client](http://sonatype.github.io/async-http-client/apidocs/reference/com/ning/http/client/AsyncHttpClient.html)를 사용하여 접근할 수 있다.

@[underlying](code/ScalaWSSpec.scala)

이것은 몇가지 경우에 중요하다.  WS는 클라이언트에 접근하는데 몇가지 제약을 가지고 있다.:

* `WS`는 직접 업로드를 하는 멀티 파트 형식을 지원하지 않는다. [RequestBuilder.addBodyPart](http://asynchttpclient.github.io/async-http-client/apidocs/com/ning/http/client/RequestBuilder.html)를 활용하여 사용할 수 있다.
* `WS` 스트리밍 내용 업로드를 지원하지 않는다. 이 경우에는 AsyncHttpClient에 의해 제공되는 `FeedableBodyGenerator`를 사용해야 한다.

## WS 설정하기

WS클라이언트를 설정하기 위해서는 `application.conf`내의 아래와 같은 속성을 사용해야 한다:

* `play.ws.followRedirects`: 301과 302 우회를 따르도록 클라이언트를 설정한다. *(기본은 **true**)*.
* `play.ws.useProxyProperties`: http 프록시 설정을 사용한다. (http.proxyHost, http.proxyPort) *(기본은 **true**)*.
* `play.ws.useragent`: User-Agent 헤더 항목을 설정한다.
* `play.ws.compressionEnabled`: gzip/deflater encoding을 위해 true로 설정해야 한다. *(기본은 **false**)*.

### WS를 SSL과 함께 설정하기

WS가 SSL/TLS (HTTPS)을 통한 HTTP를 사용하도록 설정하려면, [[WS SSL 설정하기|WsSSL]]를 참고해야 한다.

### 시간제한 설정하기

WS는 3가지의 시간제한이 존재한다. 시간제한에 도달하면 WS요청은 중단된다.

* `play.ws.timeout.connection`: 원격지 호스트에 접속하기까지의 대기 시간 *(기본은 **120 초**)*.
* `play.ws.timeout.idle`: 요청이 없는 상태로 기다리는 시간 (접속은 되었지만 더 많은 데이터를 기다리는 것) *(기본은 **120 초**)*.
* `play.ws.timeout.request`: 요청을 받는 총 시간 (원격 호스트가 데이터를 보내고 있는 중이어도, 시간제한에 걸릴 수 있다.) *(기본은 **120 초**)*.

특별한 접속을 위해 `withRequestTimeout()`를 통해 시간제한은 덮어써질 수 있다. ("요청 만들기" 항목을 참고).

### AsyncClientConfig 설정하기

아래의 고급 설정이 AsyncHttpClientConfig를 설정할 수 있다. 더 많은 정보를 위해서는 [AsyncHttpClientConfig 문서](http://asynchttpclient.github.io/async-http-client/apidocs/com/ning/http/client/AsyncHttpClientConfig.Builder.html)를 참고하자.

* `play.ws.ning.allowPoolingConnection`
* `play.ws.ning.allowSslConnectionPool`
* `play.ws.ning.ioThreadMultiplier`
* `play.ws.ning.maxConnectionsPerHost`
* `play.ws.ning.maxConnectionsTotal`
* `play.ws.ning.maxConnectionLifeTime`
* `play.ws.ning.idleConnectionInPoolTimeout`
* `ws.ning.webSocketIdleTimeout`
* `play.ws.ning.maxNumberOfRedirects`
* `play.ws.ning.maxRequestRetry`
* `play.ws.ning.disableUrlEncoding`
