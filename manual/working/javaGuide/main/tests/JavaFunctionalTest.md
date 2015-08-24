<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 함수적인 테스트 작성하기

플레이는 함수적으로 테스트하는 것을 돕기위해 몇가지 편리한 메서드와 클레스를 제공하고 있다. 대부분의 것들은 [`play.test`](api/java/play/test/package-summary.html) 패키지나 [`Helpers`](api/java/play/test/Helpers.html) 클래스에서 찾아볼 수 있다.

아래 내용을 import 하여 메서드와 클래스를 추가할 수 있다:

```java
import play.test.*;
import static play.test.Helpers.*;
```

## FakeApplication

플레이에서는 [`Application`](api/java/play/Application.html)을 컨텍스트로 시행하는 과정이 수시로 필요하다: 일반적으로 [`play.Play.application()`](api/java/play/Play.html)에서 이 애플리케이션을 제공하고 있다.

플레이는 테스트 환경을 제공하기 위해, 서로 다른 글로벌 객체, 추가 설정, 심지어는 추가 플러그인까지 설정 가능한 [`FakeApplication`](api/java/play/test/FakeApplication.html) 클래스를 제공하고 있다.

@[test-fakeapp](code/javaguide/tests/FakeApplicationTest.java)

[[dependency injection|JavaDependencyInjection]]를 지원하기 위해 Guice를 사용한다면, FakeApplication를 사용하는 대신 [[built directly|JavaTestingWithGuice]]를 테스팅을 위한 `Application`으로 사용할 수 있다.

## fake 애플리케이션을 사용한 테스팅

`Application`을 이용하여 테스트를 실행하기 위해, 아래 내용을 시도할 수 있다:

@[test-running-fakeapp](code/javaguide/tests/FakeApplicationTest.java)

[`WithApplication`](api/java/play/test/WithApplication.html)를 확장 하는 것도 가능한데, 자동으로 애플리케이션을 시작하고, 정지하는 것을 보장해 준다.

@[test-withapp](code/javaguide/tests/FunctionalTest.java)

여기서 보듯이 [`FakeApplication`](api/java/play/test/FakeApplication.html)이 각 테스트 메소드마다 시작하고 정지하는 것을 보장해 줄 것이다.

## Guice 애플리케이션 테스트하기

`Application` [[created by Guice|JavaTestingWithGuice]]을 테스트 실행하기 위해, 아래 내용을 시도할 수 있다:

@[test-guiceapp](code/tests/guice/JavaGuiceApplicationBuilderTest.java)

주의. 테스트에서 Guice를 사용시, 다양한 방법으로 `Application` 생성 과정을 커스터마이즈 하는 것이 가능하다.

## 서버로 테스트하기

때로는 테스트에서 실제 HTTP 스택 테스트를 하고 싶을 수도 있다. 테스트 서버를 실행하여 테스트 할 수 있다:

@[test-server](code/javaguide/tests/FunctionalTest.java)

테스트시, [`TestServer`](api/java/play/test/TestServer.html)를 자동으로 멈추거나 시작할 수 있도록 `WithApplication` 클래스를 사용한다. 혹은 [`WithServer`](api/java/play/test/WithServer.html)를 확장하여 사용하는 것도 가능하다:

@[test-withserver](code/javaguide/tests/ServerFunctionalTest.java)

## 브라우저로 테스트 하기

웹브라우저를 통해 테스트 하고 싶다면, [Selenium WebDriver](http://code.google.com/p/selenium/?redir=1)를 사용할 수 있다. 플레이가 WebDriver를 시행한 후, (WebDriver를) [FluentLenium](https://github.com/FluentLenium/FluentLenium)에서 제공하는 편리한 API들로 감싸 줄 것이다.

@[test-browser](code/javaguide/tests/FunctionalTest.java)

그리고 당연히 [`WithBrowser`](api/java/play/test/WithBrowser.html) 클래스가 각 테스트 마다 브라우저를 자동으로 열고 닫아 준다:

@[test-withbrowser](code/javaguide/tests/BrowserFunctionalTest.java)

## 라우터 테스트 하기

`Action`을 직접 호출하는 대신, `Router`를 시행하게 할 수도 있다:

@[bad-route-import](code/javaguide/tests/FunctionalTest.java)

@[bad-route](code/javaguide/tests/FunctionalTest.java)
