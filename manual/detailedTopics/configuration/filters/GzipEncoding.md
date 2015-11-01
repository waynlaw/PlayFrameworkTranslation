<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# gzip 인코딩 환경설정하기

플레이는 gzip 응답을 사용할 수 있게 하는 gzip 필터를 제공한다.

## gzip 필터 활성화 하기

gzip 필터를 활성화 하기 위해 `build.sbt`에 `libraryDependencies`에 플레이 필터 프로젝트를 추가하자.

@[content](code/filters.sbt)

이제 여러분의 필터에 실제로 여러분의 프로젝트의 최상단에 `Filters` 클래스를 생성할 gzip 필터를 추가했다.

Scala
: @[filters](code/GzipEncoding.scala)

Java
: @[filters](code/detailedtopics/configuration/gzipencoding/Filters.java)

## gzip 필터 환경설정하기

gzip 필터는 `application.conf`에서 환경설정할 수 있는 여러 옵션을 제공한다. 사용가능한 환경설정 옵션을 살펴보기 위해 플레이 필터 [`reference.conf`](resources/confs/filters-helpers/reference.conf)를 살펴보자.

## gzip된 응답 처리하기

불리언으로 요청 헤더나 응답 헤더의 함수를 승인하는 `shouldGzip` 파라미터를 사용하여 응답에 대한 처리를 구현할 수 있다.

예를들어, 아래 코드는 오직 gzip을 사용한 HTML 응답만 준다.

@[should-gzip](code/GzipEncoding.scala)
