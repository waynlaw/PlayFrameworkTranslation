<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# specs2를 사용하여 함수형 테스트 작성하기

플레이는 함수형 테스트를 도울 수 있는 몇 가지 도움을 주는 함수들과 클래스를 제공합니다. 이런 대부분의 것들은 [`play.api.test`](api/scala/index.html#play.api.test.package) 패키지나 [`Helpers`](api/scala/index.html#play.api.test.Helpers$) 오브젝트 안에서 찾을 수 있습니다.

이런 함수나 클래스들은 아래와 같이 포함하여 추가할 수 있습니다.:

```scala
import play.api.test._
import play.api.test.Helpers._
```

## 가짜 애플리케이션

플레이는 실행중인 [`Application`](api/scala/index.html#play.api.Application)을 컨텍스트로 요청하는 경우가 자주 있습니다.: 이것은 일반적으로 [`play.api.Play.current`](api/scala/index.html#play.api.Play$)의 형태로 제공됩니다.

테스트를 위한 환경을 제공하기 위해서, 플레이는 다른 전역 오브젝트와 추가적인 설정, 추가적인 플러그인까지도 설정할 수 있는 [`FakeApplication`](api/scala/index.html#play.api.test.FakeApplication) 클래스를 제공해 줍니다.

@[scalafunctionaltest-fakeApplication](code/specs2/ScalaFunctionalTestSpec.scala)

## 애플리케이션 생성하기

애플리케이션을 예제로 전달하기 위해서는 [`WithApplication`](api/scala/index.html#play.api.test.WithApplication)을 사용하십시오. 명시적인 [`Application`](api/scala/index.html#play.api.Application)을 전달할 수 있지만, 기본적으로 [`FakeApplication`](api/scala/index.html#play.api.test.FakeApplication)가 편리합니다.

그 이유는 [`WithApplication`](api/scala/index.html#play.api.test.WithApplication)는 [`Around`](https://etorreborre.github.io/specs2/guide/SPECS2-3.4/org.specs2.guide.Contexts.html#aroundeach) 블럭에 내장되어 있기 때문에, 당신의 데이터를 전달하기 위해서 그것을 덮어쓸 수 있기 때문입니다.:

@[scalafunctionaltest-withdbdata](code/specs2/WithDbDataSpec.scala)

## 서버와 함께하기

간혹 실제 HTTP 스택을 테스트에 넣고 싶어할 수 있습니다. 그런 경우에는 테스트 서버를 [`WithServer`](api/scala/index.html#play.api.test.WithServer)를 사용하여 시작할 수 있습니다.:

@[scalafunctionaltest-testpaymentgateway](code/specs2/ScalaFunctionalTestSpec.scala)

`port`값은 실행중인 서버의 포트 번호를 가지고 있습니다. 기본값은 19001이지만, [`WithServer`](api/scala/index.html#play.api.test.WithServer)에 포트 번호를 전달하거나, 시스템 속성에 `testserver.port` 항목을 설정하여 값을 변경할 수 있습니다. 이 기능은 지속적인 서버의 통합에 유용하기 때문에, 포트들은 빌드마다 동적으로 예약될 수 있습니다.

[`FakeApplication`](api/scala/index.html#play.api.test.FakeApplication)또한 테스트 서버로 전달할 수 있으며, 라우트의 수정이나 WS 호출들을 테스트 할 때 유용합니다.:

@[scalafunctionaltest-testws](code/specs2/ScalaFunctionalTestSpec.scala)

## 브라우져를 사용하여 테스트하기

만일 애플리케이션을 브라우져를 통해서 테스트하길 원한다면, [Selenium WebDriver](http://code.google.com/p/selenium/?redir=1)를 활용할 수 있습니다. 플레이는 WebDriver를 당신을 위해 실행해줄 것이며, [`WithBrowser`](api/scala/index.html#play.api.test.WithBrowser)를 통해 [FluentLenium](https://github.com/FluentLenium/FluentLenium)에 의해서 제공되는 편리한 API들을 넣어줄 것입니다. [`Application`](api/scala/index.html#play.api.Application)의 포트를 변경할 수 있으며, 사용할 웹 브라우져를 선택할 수 있습니다.:

@[scalafunctionaltest-testwithbrowser](code/specs2/ScalaFunctionalTestSpec.scala)

## 플레이 명세서

[`플레이 명세서`](api/scala/index.html#play.api.test.PlaySpecification)는 기본 specs2 명세에서 제공되는 믹스인 중, 플레이의 도움을 주는 함수들과 충돌이 있는 몇가지 믹스인들을 제외한 [`명세`](https://etorreborre.github.io/specs2/api/SPECS2-3.4/index.html#org.specs2.mutable.Specification)의 확장입니다.

@[scalafunctionaltest-playspecification](code/specs2/ExamplePlaySpecificationSpec.scala)

## 뷰 템플릿 테스트하기

템플릿은 스칼라 함수의 표준이기 때문에, 그것을 테스트에서 실행하고 결과를 확인할 수 있습니다.:

@[scalafunctionaltest-testview](code/specs2/ScalaFunctionalTestSpec.scala)

## 컨트롤러 테스트하기

[`FakeRequest`](api/scala/index.html#play.api.test.FakeRequest)에 의해서 제공되는 어떠한 `Action` 코드도 호출할 수 있습니다.:

@[scalafunctionaltest-functionalexamplecontrollerspec](code/specs2/FunctionalExampleControllerSpec.scala)

기술적으로는, [`WithApplication`](api/scala/index.html#play.api.test.WithApplication)은 필요로 하지 않는다. 또한 그것을 넣기 위해 어떤 수정도 필요하지 않는다.

## 라우터 테스트하기

`Action`을 직접 호출하는 대신에, `Router`가 그것을 하도록 할 수 있다.:

@[scalafunctionaltest-respondtoroute](code/specs2/ScalaFunctionalTestSpec.scala)

## 모델 테스트 하기

SQL데이터 베이스를 사용한다면, `inMemoryDatabase`를 사용하여 메모리 내의 H2 데이터 베이스의 접속으로 대신할 수 있다.

@[scalafunctionaltest-testmodel](code/specs2/ScalaFunctionalTestSpec.scala)
