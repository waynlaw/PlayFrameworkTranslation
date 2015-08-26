<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 빌드 시스템

플레이의 빌드 시스템은 스칼라와 자바 프로젝트의 고품질 통합 빌드를 위한 [sbt](http://www.scala-sbt.org/)를 사용한다. 우리의 빌드 도구로 `sbt`를 사용하는 것은 이 페이지에서 설명하는 특정 요구사항을 플레이에게 가져온다.

## sbt 이해하기

sbt 함수는 많은 기존 빌드 태스크들과는 완전히 다르다. 근본적으로 sbt는 태스크 엔진이다. 여러분의 빌드는 실행되어질 필요가 있는 태스크의 의존성의 트리로 표현된다. 예를들어 `compile` 태스크는 `source` 태스크에 의존하고, 이 태스크는 `sourceDirectories`태스크와 `sourceGenerators` 태스크에 의존한다.

SBT는 아주 세분화된 작업으로 일반적인 빌드 실행을 분해하고, 트리에서 어떤 시점의 태스크는 빌드에서 재량에 의해서 재정의 될 수 있다. 이는 sbt를 매우 강력하게 만들었을 뿐 아니라 만약 여러분이 다른 빌드 도구를 사용하고 있었다면 사고의 변화가 필요하가.

여기 이 문서에 고차원의 플레이의 sbt 사용사례를 설명했다. 여러분의 프로젝트에 sbt를 더 많이 사용하고 싶다면 sbt를 적절하게 적용하는 방법을 이해하기 위해 [sbt tutorial](http://www.scala-sbt.org/0.13/tutorial/index.html)를 추천한다. 많은 사람들이 유용하다고 생각하는 다른 리소스는 [this series of blog posts](https://jazzy.id.au/2015/03/03/sbt-task-engine.html)를 살펴보자.

## 플레이 애플리케이션 디렉터리 구조

대부분의 사람들이 다음과 같은 디렉터리 구조가 생성하는 `activator new`명령어를 사용하여 플레이를 시작한다.

- `/`: 애플리케이션의 최 상위 폴더
- `/README`: 추후에 함께 배포될 애플리케이션에 대해 설명한 텍스트 파일
- `/app`: 애플리케이션의 코드가 저장될 곳
- `/build.sbt`: 애플리케이션의 빌드에 대해 기술해 놓은 [sbt](http://www.scala-sbt.org/) 설정
- `/conf`: 애플리케이션을 위한 환경설정 파일
- `/project`: 더 나아간 빌드 명세 정보
- `/public`: 애플리케이션이 저장한 정적, 공용 자산이 저장될 곳
- `/test`: 애플리케이션의 테스트 코드가 저장될 곳

이제 우리는 `build.sbt` 파일과 `/project` 디렉터리에 대해서 살펴볼 것이다.

## `/build.sbt` 파일

`activator new foo` 명령어를 사용할 때 빌드 명세 파일 `/build.sbt`는 다음과 같이 생성된다.

@[default](code/build.sbt)

`name`은 애플리케이션의 이름을 정의하고, 애플리케이션의 최 상위 디렉터리의 이름과 동일하게 되며 `activator new` 명령어에 준 인자에서 파생된다.

`version`은 여러분의 빌드가 생성할 actifact를 위한 이름의 일부로 사용될 애플리케이션의 버전을 제공한다.

`libraryDependencies`는 애플리케이션이 의존성이 있는 라이브러리를 명시한다.

sbt 환경설정을 위해 각각 자바나 스칼라를 위해서 `PlayJava`나 `PlayScala` 플러그인을 사용할 필요가 있다.

### 빌드를 위해 스칼라 사용하기

Activator는 또한 `project` 폴더 내의 스칼라 파일으로부터 빌드 요구사항을 생성할 수도 있다. 권장되는 방법은 `build.sbt`를 사용하는 것이지만 직접 스칼라를 사용해야 하는 때가 있다. 만약 스스로 이런 경우가 생겼다면 아마 옛날 프로젝트를 마이그레이션하는 상황일 것이며 다음과 같이 몇 가지 유용한 임포트가 있다. 
당신이 이전 프로젝트를 마이그레이션하는 아마도 때문에, 자신을 발견하는 경우 여기에 몇 가지 유용한 수입은 다음과 같습니다 :

```scala
import sbt._
import Keys._
import play.Play.autoImport._
import PlayKeys._
```

`autoImport`를 나타낸 줄은 자동으로 속성을 선언하는 sbt 플러그인을 임포팅하는 올바른 방법이다. 다음과 같이 sbt-web 플러그인을 가져오고 싶다면 다음 처럼 할 수 있다.

```scala
import com.typesafe.sbt.less.autoImport._
import LessKeys._
```

## `/project` 디렉터리

여러분의 프로젝트를 빌드하는 것과 연관된 모든 것은 `/product` 디렉터리 하위의 애플리케이션 디렉터리에 넣는다. 이는 [sbt](http://www.scala-sbt.org/)의 요구사항이다. 디렉터리 내부에는 다음 2개의 파일이 있다.

- `/project/build.properties`: 이는 사용되는 sbt 버전이 명시된 표시를 위한 파일이다.
- `/project/plugins.sbt`: 플레이 스스로 포함시킨 프로젝트 빌드에 의해 사용되어질 SBT 플러그인

## sbt를 위한 플레이 플러그인 (`/project/plugins.sbt`)

플레이 콘솔과 라이브 재시작처럼 개발 기능의 모든 것은 sbt 플러그인으로 구현된다. `/project/plugins.sbt` 파일에 다음과 같이 등록한다.

```scala
addSbtPlugin("com.typesafe.play" % "sbt-plugin" % playVersion) // 버전은 현재 플레이 버전이다.
```

> `build.properties`나 `plugins.sbt`는 플레이 버전의 변경되면 손수 갱신해주어야 한다는 것을 주의하자.
