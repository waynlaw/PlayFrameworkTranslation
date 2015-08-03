<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 애플리케이션 테스트 하기

애플리케이션 테스트를 작성하는 작업도 프로세스에 포함될 수 있다. 플레이는 애플리케이션 테스트를 가능한 한 쉽게 만들 수 있도록 헬퍼와 애플리케이션 스텁(일종의 가짜 값을 리턴하는 객체)을 제공한다.  

## 개요

테스트는 "test" 폴더 아래 위치한다. 일종의 템플릿으로써 사용될 테스트 파일 2개를 테스트 폴더에 생성하였다.

액티베이터 콘솔에서 테스트를 실행시킬 수 있다.

* 모든 테스트를 시행하려면 `test`를 입력한다.
* 하나의 테스트 클래스만 시행하려면 `testOnly` 다음 클래스 이름을 입력한다. i.e. `testOnly my.namespace.MyTest`.
* 실패한 테스트만 시행하려면 `testQuick`를 입력한다.
* 테스트를 계속 시행하기 위해서는 물결표시(tilde)를 앞에 붙여 입력한다. i.e. `~testQuick`.
* 콘솔에서 `FakeApplication`와 같은 테스트 헬퍼에 접근하기 위해서는  `test:console`을 입력한다.

플레이 테스트는 [sbt](http://www.scala-sbt.org/)를 기반으로 하고 있다. 전체 설명은 [testing documentation]에서 확인 가능하다.
(http://www.scala-sbt.org/release/docs/Detailed-Topics/Testing.html).

## JUnit 사용하기

플레이 애플리케이션을 테스트하는 기본 방법은 [JUnit](http://www.junit.org/)을 이용하는 방법이다.

@[test-simple](code/javaguide/tests/SimpleTest.java)

> **주의:** `test` 나 `test-only`가 시행되는 매 순간마다 새로운 프로세스가 분할 생성된다. 이 프로세스는 기본 JVM 설정을 사용하고 있다. 커스텀 설정을 `build.sbt`에 추가할 수 있다. 예를 들어:

> ```scala
> javaOptions in Test ++= Seq(
>   "-Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=9998",
>   "-Xms512M",
>   "-Xmx1536M",
>   "-Xss1M",
>   "-XX:MaxPermSize=384M"
> )
> ```

### 어절션(Assertions) & 매쳐(Matchers)

일부 개발자는는 JUnit 어절트 구문 보다 더 능숙한 방법으로 어절트 구문을 작성하고 싶어한다. 편의를 위해 몇몇 인기있는 다른 어절션 스타일의 라이브러리가 포함되어 있다.

Hamcrest matchers:

@[test-hamcrest](code/javaguide/tests/HamcrestTest.java)

### 목(Mocks)

목은 외부 의존성이 필요한 유닛 테스트를 독립적으로 시행하기 위해 사용된다. 예를 들어 테스트 대상인 클래스가 외부 클레스 데이터에 의존적일 때, 이 데이터를 제어하고, 외부 데이터 자원에 대한 접근을 제거하기 위해 목(mock)을 사용할 수 있다.

목(mock)을 사용할 수 있도록 [Mockito](https://code.google.com/p/mockito/) 라이브러리가 프로젝트 빌드시에 포함되어 있다. 

Mockito를 사용하여 클래스나 인터페이스 같은 것을 흉내낼 수 있다:

@[test-mockito-import](code/javaguide/tests/MockitoTest.java)

@[test-mockito](code/javaguide/tests/MockitoTest.java)

## 유닛 테스팅 모델

아래의 데이터 모델을 가지고 있다고 가정해보자:

@[test-model](code/javaguide/tests/ModelTest.java)

Ebean과 같은 데이터 접근 라이브러리는 static 메서드를 이용하여 모델 클레스에서 데이터 접근 로직을 직접 추가할 수 있도록 허용해 놓았다. 이 방법을 통해 다루기 까다로는 데이터 의존성을 흉내낼 수 있다.

데이터베이스에서 가능한한 많은 로직을 격리시켜서 모델을 독립적으로 유지하고, 리파지토리 인터페이스 뒤로 데이터베이스 접근을 추상화 시키는 것이 가장 테스트 용이도를 높일 수 있는 일반적인 접근 방법이다. 

@[test-model-repository](code/javaguide/tests/ModelTest.java)

이후에 모델과 통신하기 위해 리파지토리를 포함한 서비스를 사용하게 된다:

@[test-model-service](code/javaguide/tests/ModelTest.java)

이러한 방법으로, `UserService.isAdmin` 메소드는 `UserRepository` 의존성을 흉내내어 테스트 할 수 있게 된다:

@[test-model-test](code/javaguide/tests/ModelTest.java)

> **주의:** Ebean ORM을 사용파는 애플리케이션의 경우 플레이의 자동 getter/setter 생성에 의존하여 쓰여져 있다. 또한 플레이는 생성된 getters/setters를 사용하기 위해 필드 접근을 다시 재정의하게 된다. Ebean은 더디 체킹을 위해 setter를 호출에 의존성을 가지고 있다. JUnit 테스트에서 이런 패턴을 사용하기 위해서, 테스트에서 플레이의 필드 접근 재정의를 사용 가능하게 해야한다. `build.sbt`에 다음을 추가하도록 한다.


> ```scala
> compile in Test <<= PostCompile(Test)
> ```
>
> You may also need the following import at the top of your `build.sbt`:
>
> ```scala
> import play.Play._
> ```

## 컨트롤러 유닛 테스팅

유용한 프로퍼티들을 추출하기 위해, 플레이의 [test helpers](api/java/play/test/Helpers.html)를 사용하여 컨트롤러를 테스트 할 수 있다. 
You can test your controllers using Play's [test helpers](api/java/play/test/Helpers.html) to extract useful properties.

@[test-controller-test](code/javaguide/tests/ApplicationTest.java)

리버스 라우터에서 액션 참조를 가져와 호출하는 것도 가능하다. 이 작업을 통해 요청 데이터를 흉내내기 위한 `FakeRequest`를 사용할 수 있게 된다.  
You can also retrieve an action reference from the reverse router and invoke it. This also allows you to use `FakeRequest` which is a mock for request data:

@[test-controller-routes](code/javaguide/tests/ApplicationTest.java)

## 뷰 템플릿 유닛 테스팅

템플릿은 표준 스칼라 함수이다. test에서 시행킬 수 있으며, 결과를 체크하면 된다: 

@[test-template](code/javaguide/tests/ApplicationTest.java)
