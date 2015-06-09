<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 함수형 테스트를 ScalaTest로 작성하기

플레이는 함수형 테스트를 지원하기 위한 몇가지 클래스와 함수들을 제공한다. 대부분의 이런 기능들은 [`play.api.test`](api/scala/index.html#play.api.test.package)패키지나 [`Helpers`](api/scala/index.html#play.api.test.Helpers$)오브젝트 안에서 찾을 수 있다. [_ScalaTest + Play_](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.package) 통합 라이브러리는 ScalaTest를 위해서 이러한 기능들을 기반으로 만들어졌다.

플레이에 내장되어 있는 모든 기능들과 _ScalaTest + Play_ 를 아래의 import를 통해서 접근할 수 있다.

```scala
import org.scalatest._
import play.api.test._
import play.api.test.Helpers._
import org.scalatestplus.play._
```

## 가상 애플리케이션

플레이는 종종 실행중인
[`애플리케이션`](api/scala/index.html#play.api.Application) 을 컨텍스트로 필요로 한다. 보통 [`play.api.Play.current`](api/scala/index.html#play.api.Play$)에서 제공되는 경우이다.

테스트들을 위한 환경을 제공하기 위해서, 플레이는 다른 `전역`객체나, 추가적인 설정, 다른 플러그인들과 함께 설정될 수 있는 [`FakeApplication`](api/scala/index.html#play.api.test.FakeApplication)클래스를 제공한다.

@[scalafunctionaltest-fakeApplication](code-scalatestplus-play/ScalaFunctionalTestSpec.scala)

만일 모든 테스트나 혹은 대부분의 테스트가 동일하고 공유되어도 괜찮은 `FakeApplication`을 필요로 한다면, [`OneAppPerSuite`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.OneAppPerSuite)에 포함시킬 수 있다. 이러한 경우에는 `app`의 필드에서 `FakeApplication`을 가져올 수 있다. 만일 특별한 `FakeApplication`이 필요한 경우에는 `app`을 아래의 예제와 같이 재정의 하여 사용하면 된다.

@[scalafunctionaltest-oneapppersuite](code-scalatestplus-play/oneapppersuite/ExampleSpec.scala)

만일 `FakeApplication`을 공유하지 않고 각각의 테스트마다 고유한 `FakeApplication`을 필요로 한다면, `OneAppPerTest`을 사용하면 된다.

@[scalafunctionaltest-oneapppertest](code-scalatestplus-play/oneapppertest/ExampleSpec.scala)

_ScalaTest + Play_가 `OneAppPerSuite`와 `OneAppPerTest` 모두를 제공하기 때문에, 공유하는 쪽을 선택하는 것이 테스트들을 가장 빠르게 수행되도록 만든다. 만일 애플리케이션 상태가 성공한 테스트들 간에 공유되길 원한다면, `OneAppPerSuite`을 사용할 수 있다. 그렇지 않고 각각의 테스트가 초기 상태를 필요로 한다면, `OneAppPerTest`나 `OneAppPerSuite` 모두를 사용할 수 있지만 각각의 테스트가 종료될 때 상태를 초기화 해주어야 한다. 만일 여러개의 테스트 들이 동일한 애플리케이션을 공유한다면 테스트들이 가장 빠르게 실행될 것이다. 이와 같은 경우에는 [`ConfiguredApp`을 위한 문서](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.ConfiguredApp)의 예제와 같이 [`OneAppPerSuite`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.OneAppPerSuite)내에 주요한 테스트를 두고, [`ConfiguredApp`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.ConfiguredApp) 안에는 내부 테스트를 넣을 수 있다. 위에 나온 어떤 전략을 사용하더라도 테스트들이 가장 빠르게 실행될 수 있을 것이다.

## 서버와 함께 테스트 하기

가끔은 실제 HTTP를 이용하여 테스트를 진행해야 한다. 만일 테스트 클래스내의 모든 테스트들이 동일한 서버 객체를 재사용한다면, [`OneServerPerSuite`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.OneServerPerSuite)(테스트를 위한 `FakeApplication`안에서도 제공될 것이다.) 안에 포함시킬 수 있다.

@[scalafunctionaltest-oneserverpersuite](code-scalatestplus-play/oneserverpersuite/ExampleSpec.scala)

만일 테스트 클래스내의 모든 테스트들이 독립된 서버 객체를 원한다면, [`OneServerPerTest`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.OneServerPerTest)(테스트를 위한 `FakeApplication`안에서도 제공될 것이다.)를 사용하면 된다.

@[scalafunctionaltest-oneserverpertest](code-scalatestplus-play/oneserverpertest/ExampleSpec.scala)

`OneServerPerSuite`와 `OneServerPerTest`트레이트들은 서버가 사용하는 `port` 필드를 통해 포트 넘버를 제공해준다. 기본값으로는 19001이 설정되어 있지만, `port`를 재정의 하거나 시스템 속성값인 `testserver.port`를 설정하여 변경할 수 있다. 이 기능은 서버들을 지속적으로 통합하는데 유용하며, 각각의 빌드마다 포트번호가 동적으로 할당될 수 있다.

또한 [`FakeApplication`](api/scala/index.html#play.api.test.FakeApplication)을 `app`을 재정의하여정, 이전 예제에서 보여준 것과 같이 설정을 변경할 수 있다.

마지막으로 여러개의 테스트 클래스들이 동일한 서버를 공유하도록 하면, `OneServerPerSuite`나 `OneServerPerTest`보다 더 나은 성능을 낼 수 있다. [`ConfiguredServer`를 위한 문서](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.ConfiguredServer)의 예제와 같이 주요 테스트들을 [`OneServerPerSuite`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.OneServerPerSuite)에 정의해 넣을 수 있으며, [`ConfiguredServer`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.ConfiguredServer)에 내부적인 테스트들을 넣을 수 있다.

## 웹 브라우저와 함께 테스트 하기

_ScalaTest + Play_ 라이브러리를 ScalaTest의 [Selenium DSL](http://doc.scalatest.org/2.1.5/index.html#org.scalatest.selenium.WebBrowser)으로 만들면 플레이 애플리케이션을 웹 브라우저에서 테스트하기 쉽게 만들어 준다.

테스트 클래스 내의 모든 테스트들을 동일한 웹 브라우저 객체를 가지고 테스트 하기 위해서는 [`OneBrowserPerSuite`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.OneBrowserPerSuite)을 테스트 클래스내에 넣어야 한다. 또한 [`BrowserFactory`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.BrowserFactory)트레이트을 넣으면 다음의 Selenium웹 드라이버중 하나를 제공해 줄 것이다. [`ChromeFactory`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.ChromeFactory), [`FirefoxFactory`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.FirefoxFactory), [`HtmlUnitFactory`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.HtmlUnitFactory), [`InternetExplorerFactory`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.InternetExplorerFactory), [`SafariFactory`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.SafariFactory).

만일 [`BrowserFactory`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.BrowserFactory) 또한 추가해 넣게 된다면, 다음중 하나를 제공해주는 [`ServerProvider`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.ServerProvider)을 필요로 하게 될 것이다. [`OneServerPerSuite`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.OneServerPerSuite), [`OneServerPerTest`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.OneServerPerTest), or [`ConfiguredServer`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.ConfiguredServer).

테스트 클래스를 `OneServerPerSuite`와 `HtmUnitFactory`에 포함하는 것을 예를 들어 보자.

@[scalafunctionaltest-onebrowserpersuite](code-scalatestplus-play/onebrowserpersuite/ExampleSpec.scala)

만일 각각의 테스트가 새로운 브라우저 객체를 필요로 한다면, [`OneBrowserPerTest`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.OneBrowserPerSuite)를 사용하면 된다. `OneBrowserPerSuite`를 사용하는 것처럼 `ServerProvider` 와 `BrowserFactory`를 추가해야 할 것이다.

@[scalafunctionaltest-onebrowserpertest](code-scalatestplus-play/onebrowserpertest/ExampleSpec.scala)

만일 여러개의 테스트 클래스들이 하나의 동일한 브라우저 객체를 사용해야 한다면, [`OneBrowserPerSuite`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.OneBrowserPerSuite)를 마스터 테스트 케이스에 넣고, [`ConfiguredBrowser`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.ConfiguredBrowser)를 여러 내장된 테스트 들에 넣으면, 모든 테스트들이 동일한 웹 브라우저를 사용하게 될 것이다. 다음의 문서에 예가 나와있다. [`ConfiguredBrowser`트레이트을 위한 문서](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.ConfiguredBrowser).

## 동일한 테스트를 여러 브라우저들에서 실행하는 방법

만일 어플리케이션이 지원하는 모든 브라우저에서 올바르게 동작하는지를 확인하기 위해서, 테스트들을 여러 브라우저들에서 실행하길 원할 수 있다. 이런 경우에는 [`AllBrowsersPerSuite`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.AllBrowsersPerSuite)이나 [`AllBrowsersPerTest`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.AllBrowsersPerTest)와 같은 트레이트을 사용할 수 있다.
이런 트레이트들은 모두 `IndexedSeq[BrowserInfo]`형식의 `browsers`필드를 가지고 있으며, `BrowserInfo`를 받는 추상화된 `sharedTests` 함수를 가지고 있다. `browsers`필드는 테스트를 실행하기를 원하는 브라우저에 대해서 알려주는 기능을 한다. Chrome, Firefox, Internet Explorer, `HtmlUnit`그리고 Safari가 기본값이다. 기본값이 원하는 브라우저들과 맞지 않는 경우에는 `browsers`를 재정의 할 수 있다. 여러개의 브라우저에서 실행하기 원하는 테스트들은 `sharedTests` 함수에 정의할 수 있다. 각각의 테스트의 이름 끝에는 browser.name을 기재하여야 한다. (`sharedTests`에 넘겨진`BrowserInfo`에 있는 브라우저의 이름이 사용 가능하다.) `AllBrowsersPerSuite`를 사용하는 예제는 아래와 같다.


@[scalafunctionaltest-allbrowserspersuite](code-scalatestplus-play/allbrowserspersuite/ExampleSpec.scala)

`sharedTests`에 정의된 모든 테스트 들은 `browsers`필드에 나와있는 모든 브라우저들에서 동작한다. 그렇기 때문에 동작하는 시스템에 따라 오래걸릴 수 있다. 만일 동작하는 시스템에서 지원하지 않는 브라우저에 대한 테스트라면 자동으로 취소된다. 각각의 테스트가 고유한 이름을 가지고 있는지(이는 ScalaTest를 위해 필요하다.)를 확실하게 위해 수작업으로 `browser.name` 을 테스트 이름에 추가해야 한다. 만일 이렇게 하지 않고 둔다면, 중복된-테스트-이름-오류가 테스트들을 실행할 때 나타날 것이다.

[`AllBrowsersPerSuite`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.AllBrowsersPerSuite) 는 각각의 타입의 브라우저 마다 하나씩 생성되며, `sharedTests`에 정의된 모든 테스트들이 그것을 사용할 것이다. 만일 각각의 테스트가 고유한 새 브라우저 인스턴스를 가지길 원한다면, [`AllBrowsersPerTest`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.AllBrowsersPerTest)를 사용해야 한다.:

@[scalafunctionaltest-allbrowserspertest](code-scalatestplus-play/allbrowserspertest/ExampleSpec.scala)

`AllBrowsersPerSuite`와 `AllBrowsersPerTest` 모두가 사용할 수 없는 브라우저 형식에 대해서는 테스트를 취소할 것이며, 테스트 결과에는 취소되었다고 표시될 것이다. 출력을 깨끗하게 하기 위해서는 아래의 예제와 같이 `browsers`를 재정의 하여 이용할 수 없는 브라우저를 제거해야 할 것이다.

@[scalafunctionaltest-allbrowserspersuite](code-scalatestplus-play/allbrowserspersuite/ExampleOverrideBrowsersSpec.scala)

위의 테스트 클래스는 오직 Firefox와 Chrome에 대해서만 공유된 테스트들을 시도할 것이다.(그리고 사용할 수 없는 브라우저에 대해서는 자동으로 취소할 것이다.) 

## PlaySpec

`PlaySpec`은 플레이 테스트들을 위해 편리한 "부모 테스트"인 ScalaTest의 기저 클래스를 제공한다. `PlaySpec`을 확장한 `WordSpec`, `MustMatchers`, `OptionValues` 그리고 `WsScalaTestClient`를 자동으로 얻을 수 있을 것이다.

@[scalafunctionaltest-playspec](code-scalatestplus-play/playspec/ExampleSpec.scala)

이전에 언급된 모든 트레이트을 `PlaySpec`과 함께 사용할 수 있다.

## 서로 다른 테스트들이 다른 fixture들을 원하는 경우

이전에 예제에 나왔던 테스트 클래스들은 대부분 테스트 클래스내의 테스트들이 동일한 fixture를 필요로 했다. 이와 같은 경우가 대부분이며, 일반적으로 문제가 되지 않는다. 하지만 서로다른 테스트들이 다른 fixture들을 원하는 경우에는 [`MixedFixtures`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.MixedFixtures)트레이트을 사용해야 한다. 다음의 파라메터가 없는 함수들을 중 하나를 통해서 각각의 테스트에 fixture를 제공할 수 있다.: [App](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.MixedFixtures$App), [Server](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.MixedFixtures$Server), [Chrome](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.MixedFixtures$Chrome), [Firefox](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.MixedFixtures$Firefox), [HtmlUnit](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.MixedFixtures$HtmlUnit), [InternetExplorer](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.MixedFixtures$InternetExplorer), [Safari](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.MixedFixtures$Safari).

[`MixedFixtures`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.MixedFixtures)를 [`PlaySpec`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.PlaySpec) 에 넣어서 사용할 수는 없다. `MixedFixtures`는 Scala Test [`fixture.Suite`](http://doc.scalatest.org/2.1.5/index.html#org.scalatest.fixture.Suite) 를 필요로 하지만, `PlaySpec`은 그냥 평범한 [`Suite`](http://doc.scalatest.org/2.1.5/index.html#org.scalatest.Suite)를 사용하기 때문이다. 만일 fixture를 섞어 사용할 수 있는 편리한 기저 클래스를 원한다면, 아래의 예제와 같이 [`MixedPlaySpec`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.MixedPlaySpec)를 확장해서 사용면 된다.:

@[scalafunctionaltest-mixedfixtures](code-scalatestplus-play/mixedfixtures/ExampleSpec.scala)

## 템플릿 테스트 하기

템플릿은 표준적인 스칼라의 기능이기 때문에, 테스트에서 실행 해보고 결과를 확인할 수 있다.

@[scalafunctionaltest-testview](code-scalatestplus-play/ScalaFunctionalTestSpec.scala)

## 컨트롤러를 테스트 하기

[`FakeRequest`](api/scala/index.html#play.api.test.FakeRequest)에서 제공되는 `액션`도 호출할 수 있다.

@[scalatest-examplecontrollerspec](code-scalatestplus-play/ExampleControllerSpec.scala)

기술적으로는 여기서는 [`WithApplication`](api/scala/index.html#play.api.test.WithApplication)을 추가하기 위해 어떤 손해도 필요하지 않지만, 그것을 필요하진 않습니다.

## 라우터 테스트 하기

`액션`을 직접 호출하는 대신에 `Router`를 통해서 할 수 있다.

@[scalafunctionaltest-respondtoroute](code-scalatestplus-play/ScalaFunctionalTestSpec.scala)

## 모델을 테스트 하기

SQL 데이터 베이스를 사용한다면, `inMemoryDatabase`을 사용하여, 데이터 베이스 커넥션을 메모리 내의 H2 데이터베이스 객체로 사용할 수 있다.

@[scalafunctionaltest-testmodel](code-scalatestplus-play/ScalaFunctionalTestSpec.scala)

## WS 호출들 테스트 하기

만일 웹서비스를 호출한다면,
[`WSTestClient`](api/scala/index.html#play.api.test.WsTestClient)를 사용할 수 있다. 두가지 호출 방식이 이용가능한데, 각각 Call이나 문자열을 전달받는 `wsCall`과 `wsUrl`이며,  `WithApplication`내에서 호출해야 한다.

```
wsCall(controllers.routes.Application.index()).get()
```

```
wsUrl("http://localhost:9000").get()
```
