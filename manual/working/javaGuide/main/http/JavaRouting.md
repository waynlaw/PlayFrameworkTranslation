<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# HTTP 라우팅

## 내장형 HTTP 라우터

라우터는 incomming HTTP 요청을 각 action의 호출(컨트롤러 클래스의 각 public method)로 전달해주는 컴포넌트 이다.

HTTP 요청은 MVC 프레임워크에서 볼 수 있는 일종의 이벤트이다. 이 이벤트는 2가지의 중요한 정보를 포함하고 있다:

- 쿼리 스트링을 포함한 요청 경로 (예를 들어 `/clients/1542`, `/photos/list`).
- HTTP 메소드 (GET, POST, ...).

라우터 경로는 `conf/routes` 파일에 컴파일 된 상태로 정의되어 있다. 즉 브라우저에서 라우트 에러를 직접 확인할 수 있다는 의미가 된다:

[[images/routesError.png]]

## 의존성 주입

플레이는 2가지 유형의 라우터 생성을 지원하다. 하나는 의존성 주입 라우터이고, 다른 하나는 static 라우터이다. 기본값은 static 라우터이지만, 플레이 seed 액티베이터 템플릿을 이용하여 새로운 플레이 애플리케이션을 생성한다면, 프로젝트에 포함된 `build.sbt` 설정에서 주입 라우터를 사용하고 있다는 것을 확인할 수 있다:


```scala
routesGenerator := InjectedRoutesGenerator
```
플레이 문서에 있는 코드 예시를 보면 주입 라우터 제너레이터 사용법을 예상할 수 있다. 이 것을 사용하지 않는다면, 라우터의 컨트롤러 호출 영역에 `@` 심볼을 붙이거나, 각 액션 메소드를 `static`으로 선언하여 사용한 static 라우터 제너레이트 코드를 응용해도 된다.

## 라우트 파일 문법

`conf/routes`는 라우터에서 사용하는 설정 파일이다. 애플리케이션은 필요한 모든 라우트 경로를 파일에 나열해 놓는다. 각 라우트는 HTTP 메소드와 각 액션 메소드와 연관있는 URI 패턴으로 구성되어 있다.

라우트 정의가 어떻게 되어 있나 보자:

@[clients-show](code/javaguide.http.routing.routes)

> 액션 호출 시, Scala와 같이 파라미터 이름 뒤에 파라미터 타입이 오는 것을 주의한다.

각 라우트는 HTTP 메소드로 사작하고, 그 뒤가 URI 패턴으로 이어지고 있다. 마지막으로 어디를 호출할 지를 적는다.

`#`문자열을 이용해 라우트 파일에 코멘트를 달 수도 있다:

@[clients-show-comment](code/javaguide.http.routing.routes)

## HTTP 메소드

HTTP 메소드는 HTTP가 제공하는 메소드라면 어느것이 든 가능하다  (`GET`, `PATCH`, `POST`, `PUT`, `DELETE`, `HEAD`).

## URI 패턴

URI 패턴은 라우트의 요청 경로를 정의한다. 요청 경로의 일부분은 다이나믹하게 만들어질 수 있다.

### Static 경로

예를 들어 GET /clients/all`와 일치하는 incoming 요청을 라우터에 정의하기 위해서는:

@[static-path](code/javaguide.http.routing.routes)

### Dynamic 파트 

id값을 이용해 클라이언트를 불러오는 라우터를 정의하기 위해서는 Dynamic 파트를 추가해야 한다:

@[clients-show](code/javaguide.http.routing.routes)

> URI 패턴은 하나 이상의 Dynamic 파트를 가질 수 있다.

Dynamic 파트의 기본 매칭 전략은 `[^/]+` 정규식으로 정의되어 있다. 이 정규식을 통해 `:id`로 정의 된 모든 Dynamic 파트는 하나의 URI 경로 세그먼트에 매칭되게 된다.

### 슬래시(/)가 연속된 영역에서 Dynamic 파트 사용하기

슬래시로 구분된 여러개 URI 세그먼트를 매칭해야 하는 경우에도 Dynamic 파트를 사용할 수 있다. 이 경우 `.*` 정규식을 사용하는 `*id` 문법을 이용하여 Dynamic 파트를 정의 할 수 있다:

@[spanning-path](code/javaguide.http.routing.routes)

여기 `GET /files/images/logo.png` 요청같은 경우, `name` Dynamic 파트는 `images/logo.png` 값으로 매칭 될 것이다.

### 커스텀 정슈식을 이용한 Dynamic 파트

Dynamic 파트에서 정규식을 직접 정의할 수도 있다. `$id<regex>` 문법을 사용한다:
    
@[regex-path](code/javaguide.http.routing.routes)

## Action generator 메소드 호출하기

마지막으로, 라우트 호출 영역을 정의하게 된다. 이 영역은 반드시 유효한 액션 메소드로 지정되어야 한다.

메소드 파라미터가 정의되지 않았고, 유효한 메소드 이름만 주어진 경우:

@[home-page](code/javaguide.http.routing.routes)

액션 메소드에 파라미터를 정의하고, URI 경로나 쿼리스트링에서 매칭되는 파라미터 값을 찾을 수 있는 경우.

@[page](code/javaguide.http.routing.routes)

혹은:

@[page](code/javaguide.http.routing.query.routes)

여기서 `show`에 상응하는 메소드가 `controllers.Application` 컨트롤러에 정의 되어 있는 것을 볼 수 있다:

@[show-page-action](code/javaguide/http/routing/controllers/Application.java)

### Parameter 타입들

`String` 타입 파라미터는 optional 파라미터 타입이다. 플레이가 incoming 파라미터를 특정 Scala으로 변환시키기를 원한다면, 명시적으로 타입을 추가할 수 있다:

@[clients-show](code/javaguide.http.routing.routes)

그 다음 컨트롤러에서 해당 액션 메소드 파라미터를 같은 타입으로 사용하면 된다:

@[clients-show-action](code/javaguide/http/routing/controllers/Clients.java)

> **Note:** 파라미터 타입은 suffix(접미사) 문법을 사용하여 정의해야 한다. 제너릭 타입은 Java에서와 같이 `<>`를 사용하는 대신 `[]`을 사용한다. 예를 들어 `List[String]`는 Java의 `List<String>`와 동일한 타입이다.

### 고정된 값으로 정의 된 파라미터

때로는 파라미터 값으로 고정값을 쓰고 싶을 것이다:

@[page](code/javaguide.http.routing.fixed.routes)

### 기본값을 이용한 파라미터

incoming 요청에서 값이 없을 때, 사용하고 싶은 기본 값을 제공하고 싶을 때:

@[clients](code/javaguide.http.routing.defaultvalue.routes)

### Optional 파라미터

모든 요청에서 쓰이지 않는 Optional 파라미터를 정의 할 수 있다:

@[optional](code/javaguide.http.routing.routes)

## Routing 우선순위

많은 라우터들인 같은 경로에 매칭될 수 있다. 충돌이 있을 경우, 정의 된 순서대로 첫번째 라우트가 사용된다.

## Reverse 라우팅

라우터는 Java 호출 만으로 URL을 생성해 사용할 수 있게 해준다. 또한 URI 패턴을 단일 설정 파일 안에 모아서 관리가 가능하게 되며, 애플리케이션을 리팩토링 할 때, 더 확신을 가지고 임할 수 있게 해준다.

라우트 파일에서 사용되는 각 컨트롤의 경우, 라우터가 `routes` 패키지에 자동으로‘reverse controller를 생성해 준다. 해당 컨트롤러와 같은 액션 메소드를 가지고 같은 시그니쳐를 가졌지만 `play.mvc.Result` 대신 `play.mvc.Call`을 리턴한다. 

`play.mvc.Call`은 HTTP 호출을 정의해 주고, HTTP 메소드와 URI 또한 모두 제공해 준다.

예를 들어, 아래의 컨트롤러를 생성하는 경우:

@[controller](code/javaguide/http/routing/reverse/controllers/Application.java)

그리고 `conf/routes` 파일에 해당 컨트롤러를 설정한다:

@[hello](code/javaguide.http.routing.reverse.routes)

`controllers.routes.Application`로 컨트롤러 요청을 우회해 `hello` 액션 메소드로 URL 우회 시킬 수 있다.

@[reverse-redirect](code/javaguide/http/routing/controllers/Application.java)

> **Note:** 각 컨트롤러 패키지에는 `routes`라는 서브 패키지가 있다. 따라서 `controllers.admin.Application.hello` 액션은 `controllers.admin.routes.Application.hello`를 통해 우회가 가능하다.