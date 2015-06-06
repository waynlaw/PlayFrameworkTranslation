<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# Using CoffeeScript
# 커피스크립트 사용하기

[CoffeeScript](http://coffeescript.org/)는 자바크스립트를 컴파일하는 작지만 우아한 언어이며, 자바스크립트를 작성하는데 멋진 문법을 제공한다.

플레이에서 컴파일할 자산은 `app/assets` 디렉터리에 저장되어야 한다. 이 디렉터리안에 파일들은 빌드중에 처리되며, 커피스크립트 소스는 표준 자바스크립트 파일로 컴파일된다. 생성된 자바스크립트 파일은 다른 관리되지 않는 자원 처럼 `public/` 폴더에 동일하게 보통의 리소스처럼 배포되며, 컴파일된 내역을 사용하는게 다른 자바스크립트와 별반 다르지 않음을 의미한다.

예를 들어 커피스크립트 소스 파일 `app/assets/javascripts/main.coffee`은 보통의 자바스크립트 리소스 `public/javascripts/main.js`처럼 사용가능하다.

커피스크립트 소스는 `assets` 명령으로 자동으로 컴파일되거나 개발 모드로 플레이를 실행중이라면 브라우저에서 페이지를 새로고침하면 컴파일된다. 컴파일 에러가 발생하면 브라우저에 다음과 같이 표시된다.

[[images/coffeeError.png]]

## 레이아웃

프로젝트에서 커피스크립트를 사용을 위한 레이아웃 예제를 살펴보자.

```
app
 └ assets
    └ javascripts
       └ main.coffee   
```

템플릿에서 컴파일된 자바스크립트 파일을 사용하기 위해 아래 문법을 사용할 수 있다.

```html
<script src="@routes.Assets.at("javascripts/main.js")">
```

## 환경설정

커피스크립트 컴파일은 `PlayJava`나 `PlayScala` 플러그인을 사용할 때 처럼 plugins.sbt 파일에 간단하게 플러그인을 추가해 활성화 할 수 있다.

```scala
addSbtPlugin("com.typesafe.sbt" % "sbt-coffeescript" % "1.0.0")
```

커피스크립트의 환경설정에 대해서 자세히 알고 싶다면 [플러그인 문서](https://github.com/sbt/sbt-coffeescript#sbt-coffeescript)를 참조하자.

