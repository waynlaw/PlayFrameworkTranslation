<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# ScalaTest를 사용한 애플리케이션 테스트 하기

애플리케이션을 위한 테스트를 작성하는 것 또한 관련있는 프로세스이다. 플레이는 테스트를 위한 대용 애플리케이션이나 도움을 주는 기능들 ScalaTest를 통합된 라이브러리로 제공한다. [ScalaTest + Play](http://scalatest.org/plus/play)가 애플리케이션 테스트를 쉽게 만들어준다.

## 살펴보기

테스트들은 "test"폴더 내에 존재한다.  <!-- 템플릿으로 사용할 수 있는 두개의 샘플 테스트 파일들이 테스트 폴더 내에 존재한다. -->

테스트들을 플레이 콘솔에서 실행할 수 있다.

* 모든 테스트를 실행하기 위해서는 `test`라고 입력한다.
* 하나의 테스트 클래스를 실행하기 위해서는, `test-only` 명령과 함께, 테스트 하려는 클래스의 이름을 적는다. 예: `test-only my.namespace.MySpec`.
* 실패한 테스트만을 실행하기 위해서는 `test-quick`라고 입력한다.
* 테스트를 지속적으로 실행하기 위해서는 물결표시를 앞에 붙여준다. 예: `~test-quick`.
* `FakeApplication`과 같이 테스트에 도움을 주는 기능을 콘솔에서 접근하기 위해서는, `test:console`를 입력한다.

플레이에서의 테스트는 SBT를 기반으로 되어있으며, 테스트에 대한 자세한 정보는 [SBT 테스트하기](http://www.scala-sbt.org/0.13.0/docs/Detailed-Topics/Testing) 에서 확인할 수 있다.

## ScalaTest + Play 사용하기

_ScalaTest + Play_를 사용하기 위해서는, `build.sbt` 파일을 변경하여, 빌드에 아래와 같이 추가할 필요가 있다.:

```scala
libraryDependencies ++= Seq(
  "org.scalatest" %% "scalatest" % "2.2.1" % "test",
  "org.scalatestplus" %% "play" % "1.2.0" % "test",
)
```

ScalaTest를 빌드에 명시적으로 추가할 필요는 없다. ScalaTest의 선호하는 버전이 _ScalaTest + Play_의 의존성에 의해 자동으로 받아질 것이다. 그러나 플레이의 버전에 맞춰 _ScalaTest + Play_의 버전을 선택하길 원한다면,  _ScalaTest + Play_의 [버전, 버전, 버전](http://www.scalatest.org/plus/play/versions)페이지에 나온 방법을 사용하여 할 수 있다.

[_ScalaTest + Play_](http://scalatest.org/plus/play)에서는, [`PlaySpec`](http://doc.scalatest.org/plus-play/1.0.0/index.html#org.scalatestplus.play.PlaySpec)트레잇을 확장해서 테스트 클래스를 정의할 수 있다. 예제는 아래와 같다.:

@[scalatest-stackspec](code-scalatestplus-play/StackSpec.scala)

`PlaySpec`을 사용하는 대신에 [고유한 기반 클래스들을 정의하기](http://scalatest.org/user_guide/defining_base_classes)를 사용할 수 있다.

플레이 자체를 사용하여 테스트 클래스들을 바로 실행할 수 있다. 또는 IntelliJ IDEA ([Scala 플러그인 사용하기](http://blog.jetbrains.com/scala/)) 나, Eclipse ([Scala IDE](http://scala-ide.org/)와 [ScalaTest Eclipse plugin](http://scalatest.org/user_guide/using_scalatest_with_eclipse)사용하기)를 사용해서 실행할 수 있다. [[IDE page|IDE]]에서 더 상세한 정보를 얻을 수 있다.

### Matchers

ScalaTest의 [`MustMatchers`](http://doc.scalatest.org/2.1.5/index.html#org.scalatest.MustMatchers)안에 `PlaySpec`이 포함되어 있다. ScalaTest의 Matcher DSL을 사용하여 검증하는 코드를 작성할 수 있다.:

```scala
import play.api.test.Helpers._

"Hello world" must endWith ("world")
```

[`MustMatchers`](http://doc.scalatest.org/2.1.5/index.html#org.scalatest.MustMatchers)문서를 확인하여 보다 상세한 정보를 얻을 수 있다.

### Mockito

가짜 객체를 사용하여 외부의 의존성과 독립된 테스들을 만들 수 있다. 예를 들면, 만일 클래스가 `DataService` 클래스에 의존성을 가지고 있다면, `DataService` 객체를 생성하지 않고 적절한 데이터를 클래스에 제공할 수 있다.

ScalaTest는 [Mockito](https://code.google.com/p/mockito/)와의 통합을 [`MockitoSugar`](http://doc.scalatest.org/2.1.5/index.html#org.scalatest.mock.MockitoSugar) 트레잇을 통해 제공한다.

Mockito를 사용하기 위해서는, `MockitoSugar`를 테스트 클레스 안에 섞어넣은 다음 가짜 의존성을 위해Mockito라이브러리를 사용해야 한다.:

@[scalatest-mockito-dataservice](code-scalatestplus-play/ExampleMockitoSpec.scala)

@[scalatest-mockitosugar](code-scalatestplus-play/ExampleMockitoSpec.scala)

가짜 객체를 만드는 것은 클래스의 공개된 함수들을 테스트 하는데 특별히 유용하다. 가짜 객체로 공개되지 않은 함수들을 테스트 하는 것도 가능은 하지만, 몹시 힘들것이다.

## 유닛 테스트 모델들

플레이는 특 데이터베이스에 접근하기 레이어 모델을 필요로 하지 않는다. 하지만 만일 애플리케이션이 Anorm 이나 Slick을 사용한다면, 내부적으로 모델이 데이터 베이스에 빈번하게 접근하게 될것이다.

```scala
import anorm._
import anorm.SqlParser._

case class User(id: String, name: String, email: String) {
   def roles = DB.withConnection { implicit connection =>
      ...
    }
}
```

유닛 테스트를 위해서, 이러한 접근은 `roles` 함수를 가상으로 까다롭게 만들도록 한다.

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

@[scalatest-userservicespec](code-scalatestplus-play/UserServiceSpec.scala)

## 유닛 테스팅 컨트롤러들

컨트롤러는 플레이의 오브젝트로 정의되어 있다. 그렇기 때문에 테스트하기가 까다롭다. 플레이에서는 [`getControllerInstance`](api/scala/index.html#play.api.GlobalSettings@getControllerInstance)를 통해서 [[의존성 주입|ScalaDependencyInjection]] 을 함으로써 문제를 완화할 수 있다. 컨트롤러를 테스트할 수 있는 다른 방법은 트레잇을 [명시적인 타입의 자신 접근](http://www.naildrivin5.com/scalatour/wiki_pages/ExplcitlyTypedSelfReferences)과 함께 컨트롤러에 사용함으로써 가능다.:

@[scalatest-examplecontroller](code-scalatestplus-play/ExampleControllerSpec.scala)

그리고 트레잇을 테스트다.

@[scalatest-examplecontrollerspec](code-scalatestplus-play/ExampleControllerSpec.scala)

JSON의 몸체와 같은 POST 요청을 테스트 하려면, 위에서와 같이 (`apply(fakeRequest)`)을 사용해서는 할 수 없다.; 대신 `testController`의 `call()`을 사용해야 다. :

@[scalatest-examplepost](code-scalatestplus-play/ExamplePostSpec.scala)

## 유닛 테스트의 EssentialAction

[`Action`](api/scala/index.html#play.api.mvc.Action)이나 [`Filter`](api/scala/index.html#play.api.mvc.Filter)의 테스트는 [`EssentialAction`](api/scala/index.html#play.api.mvc.EssentialAction)테스트를 필요로 한다. ([[ EssentialAction에 대한 추가적인 정보|HttpApi]])

이것을 위해서는, [`Helpers.call`](api/scala/index.html#play.api.test.Helpers@call) 을 아래와 같이 사용하십시오.:

@[scalatest-exampleessentialactionspec](code-scalatestplus-play/ExampleEssentialActionSpec.scala)
