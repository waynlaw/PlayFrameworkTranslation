<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# SBT 설정에 대해서

## sbt 설정에 대해서

프로젝트의 설정을 위해 `build.sbt`파일을 정의한다. 또한 [sbt documentation](http://www.scala-sbt.org)에서 설명하는 것처럼 프로젝트에 커스텀 설정을 정의할 수도 있다. 특히 [설정](http://www.scala-sbt.org/release/docs/Getting-Started/More-About-Settings)에 대한 이 문서가 많은 도움이 될 것이다.

기본적인 셋팅을 설정하기 위해 `:=` 명령을 사용한다.


```scala
confDirectory := "myConfFolder"     
```

## 자바 애플리케이션을 위한 기본 설정

플레이는 자바-기반 애플리케이션을 위한 적합한 설정의 집합을 기본으로 정의한다. 이를 활성화 하기 위해 프로젝트의 enablePlugins 메소드에 `PlayJava` 플러그인을 추가하자. 이 설정들은 템플릿 생성 등을 위한 기본 임포트를 거의 다 정의해두었다. `Long`과 같은 `java.lang.*`의 타입을 임포트하는 것은 스칼라의 타입 대신에 자바의 타입을 사용하기 위한 것이다. `play.Project.playJavaSettings`은 또한 `java.util.*`을 임포트해서 기본 컬렉션 라이브러리로 자바의 것을 사용한다.

## 스칼라 애플리케이션을 위한 기본 설정

플레이는 스칼라-기반 애플리케이션을 위한 적합한 설정의 집합을 기본으로 정의한다. 이를 활성화 하기 위해 프로젝트의 enablePlugins 메소드에 `PlayScala` 플러그인을 추가하자. 이 설정들은 국제화된 메시지, 코어 API처럼 템플릿 생성 등과 관련된 기본 임포트를 거의 다 정의해두었다.
