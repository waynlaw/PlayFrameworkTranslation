<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# Actions, Controllers 와 Results

## Actions은 무엇인가?

플레이 애플리케이션 받은 대부분의 request는 `Action` 이 처리한다. 

단일 action은 기본적으로 request 파라미터를 처리하는 Java 메소드이다. 그리고 클라이언트에게 전송될 결과를 생성한다.

@[simple-action](code/javaguide/http/JavaActions.java)

action은 web 클라이언트에게 전송할 HTTP response를 대표하는 `play.mvc.Result` 값을 리턴한다. 이 예시에서  `ok`는 **text/plain** response body를 포함한 **200 OK** response를 생성한다.

## Controllers 

controller는 약간의 action 메소드를 묶어놓은 `play.mvc.Controller`를 상속한 클래스 이다.

@[full-controller](code/javaguide/http/full/Application.java)

action을 정의하는 가장 간단한 문법은, 위에서 보듯이 파라미터가 없는 메소드에서 `Result`값을 리턴하는 것이다.

action 메소드는 파라미터를 가질 수 있다:

@[params-action](code/javaguide/http/JavaActions.java)

이 파라미터는 `Router`를 통해 생성되고, request URL를 이용해 값을 채워나갈 것이다. 이 파라미터 값은 URL path나 URL 쿼리스트링에서 추출될 것이다.

## Results

간단한 결과값을 만들어보자: web 클라이언트에게 전송한 상태 코드, HTTP header set, body와 status code를 가진 HTTP 결과물

이 결과물은 `play.mvc.Result` 에 정의되어 있다. 그리고 `play.mvc.Results` 클래스는 표준 HTTP result를 생성하기 위한 몇가지 helper를 제공하고 있다. 예를 들어 이전 장에서 우리가 사용하던 `ok` 메소드가 그렇다:

@[simple-result](code/javaguide/http/JavaActions.java)

여기 다양한 result를 생성하는 몇가지 예제들이 있다:

@[other-results](code/javaguide/http/JavaActions.java)

`play.mvc.Results` 클래스에서 이런 종류 helper들을 모두 찾아볼 수 있다.

## Redirects are simple results too

새로운 URL로 브라우저를 리다이렉팅 하는 것 또한 다른 종류의 간단한 result이다. 하지만 이런 result type들은 response body를 가지지 않는다. 

redirect result를 생성하기 위해 몇가지 helper 사용이 가능하다:

@[redirect-action](code/javaguide/http/JavaActions.java)

기본값으로 `303 SEE_OTHER` response type을 사용하고는 하는데, 더 구체적으로 status code를 지정할 수도 있다

@[temporary-redirect-action](code/javaguide/http/JavaActions.java)
