<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# specs2를 사용해서 애플리케이션 테스트하기

애플리케이션을 위한 테스트를 만드는 것도 연관된 프로세스라고 할 수 있다. 플레이는 기본적인 테스트 프레임워크를 제공해주며, 또한 애플리케이션을 쉽게 테스트 할 수 있는 대용 애플리케이션이나 도움을 주는 기능을 제공한다.

## 개요

테스트들을 위한 경로는 "test" 폴더이다. 테스트 폴더에는 간단한 두가지 테스트 파일이 있으며 템플릿처럼 사용할 수 있다.

테스트들을 플레이 콘솔에서 실행할 수 있다.

* 모든 테스트를 실행하기 위해서는 `test`라고 입력한다.
* 하나의 테스트 클래스를 실행하기 위해서는, `test-only` 명령과 함께, 테스트 하려는 클래스의 이름을 적는다. 예: `test-only my.namespace.MySpec`.
* 실패한 테스트만을 실행하기 위해서는 `test-quick`라고 입력한다.
* 테스트를 지속적으로 실행하기 위해서는 물결표시를 앞에 붙여준다. 예: `~test-quick`.
* `FakeApplication`과 같이 테스트에 도움을 주는 기능을 콘솔에서 접근하기 위해서는, `test:console`를 입력한다.

플레이에서의 테스트는 SBT를 기반으로 되어있으며, 테스트에 대한 자세한 정보는 [SBT 테스트하기](http://www.scala-sbt.org/0.13.0/docs/Detailed-Topics/Testing) 에서 확인할 수 있다.

## specs2 사용하기

플레이의 specs2 지원을 사용하기 위해서는, 플레이 specs2 의존성을 빌드의 테스트 의존성에 추가해야 한다.:

```scala
libraryDependencies += specs2 % Test
```

[specs2](http://etorreborre.github.io/specs2/)에서는, 테스트들은 다양한 코드 경로들을 통해 테스트 아래에서 시스템을 실행할 수 있는 명세들에 정리되어 있다.

명세들은 [`Specification`](https://etorreborre.github.io/specs2/api/SPECS2-3.4/index.html#org.specs2.mutable.Specification) 트레잇을 상속받아야 하며, 형식에 맞춰져야 한다.

@[scalatest-helloworldspec](code/specs2/HelloWorldSpec.scala)

명세들은 IntelliJ IDEA ([Scala plugin](http://blog.jetbrains.com/scala/))을 사용)이나 Eclipse ([Scala IDE](http://scala-ide.org/)를 사용)해서 실행할 수 있다. 더 상세한 정보를 위해서는 [[IDE page|IDE]]를 참고하면 된다.

메모: [presentation compiler](https://scala-ide-portfolio.assembla.com/spaces/scala-ide/support/tickets/1001843-specs2-tests-with-junit-runner-are-not-recognized-if-there-is-package-directory-mismatch#/activity/ticket:)에 존재하는 버그로 인해서, Eclipse에서 사용하기 위해서는 테스트들은 반드시 특정한 형식으로 정의되어야 한다.:

* 패키지는 반드시 경로와 같아야 한다.
* 명세는 반드시 `@RunWith(classOf[JUnitRunner])`어노테이션과 함께 있어야 한다.

아래에 Eclipse를 위한 올바른 명세가 있다.:

```scala
package models // 이 파일은 반드시 "models"이라는 폴더 아래에 있어야 한다.

import org.specs2.mutable._
import org.specs2.runner._
import org.junit.runner._

@RunWith(classOf[JUnitRunner])
class ApplicationSpec extends Specification {
  ...
}
```

### Matchers

만일 예제를 사용하는 경우에는, 반드시 예제 결과를 반환하여야 한다. 일반적으로 `must`를 포함하는 구문을 보게 될 것이다.:

```scala
"Hello world" must endWith("world")
```

`must`키워드 뒤에 오는 표현식은 [`matchers`](https://etorreborre.github.io/specs2/guide/SPECS2-3.4/org.specs2.guide.Matchers.html)로 알려져 있다. matcher들은 반드시 일반적으로 성공이나 실패인 예제 결과를 반환해야 한다. 예제는 결과를 반환하지 않고는 컴파일 되지 않을 것이다.

가장 유용한 matcher들은 [match results](https://etorreborre.github.io/specs2/guide/SPECS2-3.4/org.specs2.guide.Matchers.html#out-of-the-box)이다. 이것들은 동일성을 확인하는데 사용되며, Option이나 Either를 확인하고, 심지어 예외가 던져졌는지를 확인할수도 있다.

또한 테스트들에서 XML과 JSON을 비교해주기 위한 [optional matchers](https://etorreborre.github.io/specs2/guide/SPECS2-3.4/org.specs2.guide.Matchers.html#optional)도 존재한다.

### Mockito

Mock은 외부의 의존성이 없이 독립적인 유닛 테스트를 하기 위해서 사용된다. 예를 들어 만일 외부의 `DataService`클래스에 의존성을 가진 클래스가 있다고 하면, `DataService` 객체를 생성하지 않고도 적절한 데이터를 제공할 수 있다.

[Mockito](https://code.google.com/p/mockito/)는 스펙2에 [mocking library](https://etorreborre.github.io/specs2/guide/SPECS2-3.4/org.specs2.guide.UseMockito.html)의 기본으로 통합되어 있다.

Mockito를 사용하기 위해서는 다음의 구문을 추가해야 한다.:

```scala
import org.specs2.mock._
```

그러면 클래스와 유사하게 동작하는 가상의 객체를 만들 수 있을 것이다.:

@[specs2-mockito-dataservice](code/specs2/ExampleMockitoSpec.scala)

@[specs2-mockito](code/specs2/ExampleMockitoSpec.scala)

가짜 객체를 만드는 것은 클래스의 공개된 함수들을 테스트 하는데 특별히 유용하다. 가짜 객체로 공개되지 않은 함수들을 테스트 하는 것도 가능은 하지만, 몹시 힘들것이다.

## 모델을을 유닛 테스팅 하기


플레이는 특별한 데이터베이스에 접근하기위해서 레이어 모델을 필요로 하지 않는다. 하지만 만일 애플리케이션이 Anorm 이나 Slick을 사용한다면, 내부적으로 모델이 데이터 베이스에 빈번하게 접근하게 될것이다.

```scala
import anorm._
import anorm.SqlParser._

case class User(id: String, name: String, email: String) {
   def roles = DB.withConnection { implicit connection =>
      ...
    }
}
```

유닛 테스트를 위해서, 이러한 접근은  `roles` 함수를 가상으로 까다롭게 만들도록 한다.

일반적인 접근 방법은 모델을 데이터베이스와 독립적으로 유지하며, 가능한 더 많은 기능을 넣을 수 있다. 그리고 추상화된 데이터 베이스 접근이 저장소 레이어를 통해 이루어진다.

@[scalatest-models](code/models/User.scala)

@[scalatest-repository](code/services/UserRepository.scala)

```scala
class AnormUserRepository extends UserRepository {
  import anorm._
  import anorm.SqlParser._

  def roles(user:User) : Set[Role] = {
    ...
  }
}
```

그리고 그것들을 서비스들을 통해 접근할 수 있다.:

@[scalatest-userservice](code/services/UserService.scala)

이 방법으로는, `UserRepository`를 가상으로 만들어 `isAdmin` 함수를 테스트할 수 있다. 그것을 접근하고 서비스를 통해 전달할 수도 있다.:

@[scalatest-userservicespec](code/specs2/UserServiceSpec.scala)

## 컨트롤러들을 유닛 테스트 하기

컨트롤러는 플레이의 오브젝트로 정의되어 있다. 그렇기 때문에 테스트하기가 까다로울 수 있다. 플레이에서는 [`getControllerInstance`](api/scala/index.html#play.api.GlobalSettings@getControllerInstance)를 통해서 [[의존성 주입|ScalaDependencyInjection]] 을 함으로써 문제를 완화할 수 있다. 컨트롤러를 테스트할 수 있는 다른 방법은 트레잇을 [명시적인 타입의 자신 접근](http://www.naildrivin5.com/scalatour/wiki_pages/ExplcitlyTypedSelfReferences)과 함께 컨트롤러에 사용함으로써 가능하다.:

@[scalatest-examplecontroller](code/specs2/ExampleControllerSpec.scala)

and then test the trait:

@[scalatest-examplecontrollerspec](code/specs2/ExampleControllerSpec.scala)

## Unit Testing EssentialAction

Testing [`Action`](api/scala/index.html#play.api.mvc.Action) or [`Filter`](api/scala/index.html#play.api.mvc.Filter) can require to test an [`EssentialAction`](api/scala/index.html#play.api.mvc.EssentialAction) ([[more information about what an EssentialAction is|HttpApi]])

For this, the test [`Helpers.call`](api/scala/index.html#play.api.test.Helpers@call) can be used like that:

@[scalatest-exampleessentialactionspec](code/specs2/ExampleEssentialActionSpec.scala)
