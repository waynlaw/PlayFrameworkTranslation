<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 서브 프로젝트와 함께 작업하기

복잡한 프로젝트는 부득이하게 하나의 플레이 애플리케이션으로 구성할 수 없다. 큰 프로젝트를 다수의 작은 애플리케이션으로 쪼개거나 심지어 어떤 로직은 플레이 애플리케이션과 함께 동작할 필요가 없는 표준 자바나 스칼라 라이브러리로 분리하고 싶을 것이다.

[SBT documentation on multi-project builds](http://www.scala-sbt.org/release/docs/Getting-Started/Multi-Project)문서를 읽어보는 것이 많은 도움이 될 것이다. 서브프로젝트는 자체 빌드 파일을 가지지 않으며 현재 프로젝트의 빌드 파일을 공유한다.

## 간단한 라이브러리 서브프로젝트 추가하기

간단한 라이브러리 프로젝트에 의존하는 애플리케이션을 만들 수 있다. `build.sbt` 파일에 다른 sbt 프로젝트 정의를 추가하면 된다.

```
name := "my-first-application"

version := "1.0"

lazy val myFirstApplication = (project in file("."))
    .enablePlugins(PlayScala)
    .aggregate(myLibrary)
    .dependsOn(myLibrary)

lazy val myLibrary = project
```

마지막 라인에 소문자 `project`는 스칼라 매크로이며 프로젝트의 이름과 폴더를 결정하기 위해서 할당되어진다.

`myFirstApplication` 프로젝트는 기초 프로젝트로 선언되었다. 만약에 서브 프로젝트가 없다면 이는 이미 함축되어있다. 그러나 서브프로젝트가 선언되면 보통 aggregate와 dependsOn을 선언해줄 필요가 있다.
(aggregate는 기초 프로젝트가 실행될 때 서브 프로젝트에서 컴파일/테스트 처럼 수행되는 것이며, dependsOn은 서브 프로젝트를 메인 프로젝트 클래스패스에 추가하는 것이다.)

위의 예제는 애플리케이션의 `myLibrary` 폴더에 서브 프로젝트를 정의한다. 이 서브프로젝트는 표준 sbt 프로젝트로 기본 레이아웃을 사용한다.

```
myProject
 └ build.sbt
 └ app
 └ conf
 └ public
 └ myLibrary
   └ build.sbt
   └ src
     └ main
       └ java
       └ scala
```

`myLibrary`는 자체 `build.sbt` 파일을 가지며, 이는 자체 설정과 의존성등을 선언할 수 있다.

빌드에서 서브프로젝트를 활성화 했을 때 컴파일, 테스트 실행 할 때 개별적으로 이 프로젝트를 가리킬 수 있다. 단지 플레이 콘솔 프롬프트에 모든 프로젝트가 출력되도록 `projects` 명령을 사용하면 된다.

```
[my-first-application] $ projects
[info] In file:/Volumes/Data/gbo/myFirstApp/
[info] 	 * my-first-application
[info] 	   my-library
```

알파벳순으로 첫 번째 오는 변수 이름을 가진 것이 기본 프로젝트이다. aaaMain의 변수 값으로 만들어서 당신의 메인 프로젝트를 만들고 싶을 수도 있다. 현재 프로젝트를 변경하기 위해 `project` 명령어를 사용하자.

```
[my-first-application] $ project my-library
[info] Set current project to my-library
>
```

개발 모드에서 플레이 애플리케이션을 실행하려고 하면, 의존적인 프로젝트가 자동으로 재컴파일 될 것이고, 만약 컴파일 할 수 없다면 브라우저에서 아래와 같은 결과를 보게 될 것이다.

[[subprojectError.png]]

## 공통 변수와 코드 공유하기

서브 프로젝트와 최상위 프로젝트에서 몇가지 공동 설정과 코드를 공유하고 싶다면 최상위 프로젝트의 디렉터리 `project`의 스칼라 파일에 넣을 수 있다. 예를 들어 `project/Common.scala` 파일을 가지고 있다면 아래와 같이 할 수 있다.

```scala
import sbt._
import Keys._

object Common {
  val settings: Seq[Setting[_]] = Seq(
    organization := "com.example",
    version := "1.2.3-SNAPSHOT"
  )

  val fooDependency = "com.foo" %% "foo" % "2.4"
}
```

그리고 `build.sbt` 파일 각각에 파일에 선언된 것을 참조할 수 있다.

```scala
name := "my-sub-module"

Common.settings

libraryDependencies += Common.fooDependency
```

## 웹 애플리케이션을 다수의 부분으로 분리하기

플레이 애플리케이션은 단지 기본 환경설정을 가진 표준 sbt 프로젝트 이므로 다른 플레이 애플리케이션에 의존할 수 있다. 일치하는 `build.sbt`파일이 있다면 당신의 프로젝트가 의존하는게 자바 프로젝트 이든지 스칼라 프로젝트 이든지 `PlayJava`나 `PlayScala`플러그인을 추가하여 플레이 애플리케이션의 서브 모듈로 만들 수 있다.

> **주의:** 이름 충돌을 회피하기 위해서 서브 프로젝트에서 포함시킨 Assets 컨트롤러가 메인 프로젝트와는 다른 네임스페이스를 사용하는지 당신의 컨트롤러를 확인하라.

## 라우트 파일 분리하기

작은 조각으로 라우트 파일을 분리하는 것도 가능하다. 이는 강건하고 재사용가능한 멀티-모듈 플레이 애플리케이션을 생성하고 싶다면 매우 도움이 되는 기능이다.

### 아래 빌드 환경설정에 대해 살펴보자

`build.sbt`:

```scala
name := "myproject"

lazy val admin = (project in file("modules/admin")).enablePlugins(PlayScala)

lazy val main = (project in file("."))
    .enablePlugins(PlayScala).dependsOn(admin).aggregate(admin)
```

`modules/admin/build.sbt`

```scala
name := "myadmin"

libraryDependencies ++= Seq(
  "mysql" % "mysql-connector-java" % "5.1.18",
  jdbc,
  anorm
)
```

### 프로젝트 구조

```
build.sbt
app
  └ controllers
  └ models
  └ views
conf
  └ application.conf
  └ routes
modules
  └ admin
    └ build.sbt
    └ conf
      └ admin.routes
    └ app
      └ controllers
      └ models
      └ views
project
  └ build.properties
  └ plugins.sbt
```

> **주의:** 환경설정과 라우트 파일의 이름은 전체 프로젝트 구조에서 유니크 해야한다. 특히, 하나의 `application.conf`파일과 하나의 `routes`파일이 있어야 한다. 서브프로젝트의 추가적인 라우트와 환경설정을 정의하기 위해 서브프로젝트에 특화된 이름을 사용하자. 예를들어 `admin`의 라우트 파일은 `admin.routes`로 정의하자. 개발 모드에서 서브 프로젝트를 위한 설정의 특정 집합을 사용하기 위해 빌드 파일에 `Keys.devSettings += ("play.http.router", "admin.Routes")`처럼 이들 셋팅을 명시하는 것이 더 좋다.

`conf/routes`:

```
GET /index                  controllers.Application.index()

->  /admin admin.Routes

GET     /assets/*file       controllers.Assets.at(path="/public", file)
```

`modules/admin/conf/admin.routes`:

```
GET /index                  controllers.admin.Application.index()

GET /assets/*file           controllers.admin.Assets.at(path="/public", file)

```

### Assets와 컨트롤러 클래스는 모두 `controllers.admin` 패키지에 정의되어야 한다

`modules/admin/controllers/Assets.scala`:

```scala
package controllers.admin
object Assets extends controllers.AssetsBuilder
```

> **주의:** 자바 사용자는 아주 약간의 변경이 필요하다.

```java
// Assets.java
package controllers.admin;
import play.api.mvc.*;

public class Assets {
  public static Action<AnyContent> at(String path, String file) {
    return controllers.Assets.at(path, file);
  }
}
```

그리고 컨트롤러를 

`modules/admin/controllers/Application.scala`:

```scala
package controllers.admin

import play.api._
import play.api.mvc._
import views.html._

object Application extends Controller {

  def index = Action { implicit request =>
    Ok("admin")
  }
}
```

### ```admin```의 리버스 라우팅

정식 컨트롤러 호출의 경우


```
controllers.admin.routes.Application.index
```

`Assets`의 경우

```
controllers.admin.routes.Assets.at("...")
```

### 브라우저를 통해서

```
http://localhost:9000/index
```

위의 경로는 다음 컨트롤러를 실행한다.

```
controllers.Application.index
```

그리고

```
http://localhost:9000/admin/index
```

위의 경로는 다음 컨트롤러를 실행한다.

```
controllers.admin.Application.index
```
