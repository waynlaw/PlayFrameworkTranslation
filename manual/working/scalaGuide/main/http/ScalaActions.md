<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# Actions, Controllers 그리고 결과들

## 무엇이 Action인가?

플레이 애플리케이션이 받는 대부분의 요청은 `Action`으로 다룬다. 

`play.api.mvc.Action`는 기본적으로 요청을 처리하고, 클라이언트로 전달할 결과를 만드는 `(play.api.mvc.Request => play.api.mvc.Result)` 함수이다.

@[echo-action](code/ScalaActions.scala)

action은 웹 클라이언트로 전송할 HTTP 응답을 나타내는 `play.api.mvc.Result` 값을 반환한다. 이 예제에서는 `Ok`가 **text/plain**를 응답 내용으로 담고있는 **200 OK** 응답을 생성한다.

## Action 구성하기

`play.api.mvc.Action` 컴페니언 오브젝트는 Action 값을 생성하기 위한 몇가지 도움 함수를 제공한다.

가장 간단한 것은 단지 `Result`를 반환하는 표현식 블럭을 전달인자로 받는 것이다.

@[zero-arg-action](code/ScalaActions.scala)

이것은 Action을 생성하는 가장 간단한 방법이지만, 오는 요청을 참조할 수는 없다. Action을 호출하는 HTTP요청에 접근하는 것은 때때로 유용하다.

그러므로 `Request => Result`함수를 인자로 받는 Action 생성자가 있다.

@[request-action](code/ScalaActions.scala)

`request`를 `implicit`으로 하는 것은 요청을 필요로 하는 다른 API에 암시적으로 전달할 때 유용하다.

@[implicit-request-action](code/ScalaActions.scala)

Action값을 생성하는 마지막 방법은 추가적인 `BodyParser`를 전달인자로 지정하는 것이다.

@[json-parser-action](code/ScalaActions.scala)

Body parser는 이 메뉴얼의 마지막에서 다룰 것이다. 우선 지금 알아두어야 할 것은 Action값을 생성하는 다른 함수가 기본 값으로 **임의의 내용 body parser**를 기본으로 사용한다는 것이다.

## Controllers는 Action 생성자이다.

`Controller`는 단지 `Action`을 생성하는 싱글톤 오브젝트이다.

가장 간단한 Action 생성기를 정의 하는 방법은 전달인자가 없이 `Action`값을 리턴하는 함수를 만드는 것이다.

@[full-controller](code/ScalaActions.scala)

당연히 Action을 생성하는 함수는 파라메터를 가질 수 있으며, 이 파라메터는 `Action` 클로져에 의해 참조될 수 있다.

@[parameter-action](code/ScalaActions.scala)

## 간단한 결과들

현재는 간단한 결과에만 주목하자. 상태코드를 포함하여 웹 클라이언트로 전달할 HTTP헤더와 몸체를 포함한 HTTP 결과이다.

`play.api.mvc.Result`로 이 결과들을 정의한다.

@[simple-result-action](code/ScalaActions.scala)

몇가지 도움 함수들을 통해서 위의 예제에서의 `Ok`과 같은 기본적인 결과들을 생성할 수 있다.

@[ok-result-action](code/ScalaActions.scala)

이것인 위와 동일한 결과를 생성한다.

아래에 다양한 결과들을 만드는 예제가 있다.

@[other-results](code/ScalaActions.scala)

모든 이런 도와주는 기능들은 `play.api.mvc.Results` 트레잇과 컴페니언 오브젝트에서 찾을 수 있다.

## 리다이렉션 또한 간단한 결과이다.

브라우져를 새로운 URL로 보내는 것 또한 간단한 결과의 한 종류이다. 그러나, 이런 결과 종류는 응답 몸체를 가지지 않는다.

리다이렉션 결과를 생성하는 몇 가지 도움 기능들은 아래와 같다.

@[redirect-action](code/ScalaActions.scala)

기본은 `303 SEE_OTHER` 응답 형식을 사용한다. 하지만 더 상세한 상태코드를 사용하길 원한다면 설정할 수 있다.

@[moved-permanently-action](code/ScalaActions.scala)

## "TODO" 더미 페이지

`TODO`에서 정의한 것과 같이 처럼 빈 `Action`을 사용할 수 있다. 결과는 '아직 구현되지 않음' 표준 결과 페이지 이다.

@[todo-action](code/ScalaActions.scala)
