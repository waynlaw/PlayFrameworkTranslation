<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# JSHint 사용하기

JSHint에 대해서는 이 웹페이지에 자세히 나와있다. [JSHint](http://www.jshint.com/about/):

> JSHint는 자바 스크립트 코드에서 잠재적 문제와 에러를 발견하고 여러분의 팀의 코딩 컨벤션을 강화하기 위해 커뮤니티 주도 아래에서 개발된 도구(community-driven tool) 이다. JSHint는 아주 유연하여 여러분의 코드가 예상한대로 동작하도록 상세한 코딩 가이드라인과 환경을 쉽게 적용할 수 있다.

자바 스크립트 파일이 `app/assets`에 있으면 JSHint에 의해 처리되고 에러를 확인한다.

## 자바스크립트 온전함 확인

자바스크립트 코드는 `assets` 명령으로 자동으로 컴파일되거나 개발 모드로 플레이를 실행중이라면 브라우저에서 페이지를 새로고침하면 컴파일된다. 에러가 발생하면 다른 컴파일 에러처럼 브라우저에 표시된다.

## 환경설정

JSHint 처리는 `PlayJava`나 `PlayScala`를 사용할 때 plugins.sbt파일에 간단하게 플러그인 하나를 추가하여 활성화 할 수 있다.


```scala
addSbtPlugin("com.typesafe.sbt" % "sbt-jshint" % "1.0.0")
```

플러그인의 기본 환경설정은 이미 충분하게 되어있다. 그러나 설정에 대해서 정보가 필요하다면 [plugin's documentation](https://github.com/sbt/sbt-jshint#sbt-jshint)를 참조하기 바란다.