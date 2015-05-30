<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 새로운 애플리케이션 생성하기

## activator 명령어로 새로운 애플리케이션 생성하기

`activator` 명령어를 새로운 플레이 애플리케이션을 생성하는데 사용할 수 있다. 액티베이터로 새로운 애플리케이션의 기반이 되는 템플릿을 선택할 수 있다. 평범한 플레이 프로젝트를 위한 이 템플릿들의 이름은 스칼라 기반의 플레이 애플리케이션은 `play-scala`, 자바 기반의 플레이 애플리케이션은 `play-java`이다.

> 이 시점에 자바나 스칼라로 템플릿을 선택하는 것이 이후에 언어를 변경할 수 없다는 의미를 가지는 것은 아니다. 예를 들어 새로운 애플리케이션을 기본 자바 애플리케이션 템플릿을 사용하여 생성했더라도 언제든지 원할 때 스칼라 코드를 추가할 수 있다.

새로운 플레이 스칼라 애플리케이션을 생성하기 위해, 다음을 수행하라:

```bash
$ activator new my-first-app play-scala
```

새로운 플레이 자바 애플리케이션을 생성하기 위해, 다음을 수행하라:

```bash
$ activator new my-first-app play-java
```

위에서 어느 경우에나 애플리케이션에 사용하고싶은 이름으로 `my-first-app`을 변경할 수 있다. 액티베이터는 이 이름을 생성한 애플리케이션의 디렉터리 이름으로 사용할 것이다. 만약에 원한다면 이후에 이 이름을 변경할 수 있다.

[[images/activatorNew.png]]


이전에 애플리케이션을 생성했다면 `activator` 명령어를 [[플레이 콘솔|PlayConsole]]로 들어가는데 사용할 수 있다.

```bash
$ cd my-first-app
$ activator
```

> 다른 액티베이터 템플릿을 사용하고 싶으면, `activator new`를 수행할 수 있다. 이는 애플리케이션의 이름을 보여주고 적절한 템플릿을 선택하고 살펴볼 기회를 준다.

## 액티베이터 UI로 새로운 애플리케이션 생성하기

새로운 플레이 애플리케이션은 또한 액티베이터 UI로 생성할 수 있다. 액티베이터 UI를 사용하기 위해, 다음을 수행하라:

```bash
$ activator ui
```

액티베이터 UI를 사용하기 위한 문서를 [여기](https://typesafe.com/activator/docs)에서 읽을 수 있다.

## 액티베이터 없이 새로운 애플리케이션 생성하기

또한 액티베이터를 설치하지 않고 sbt를 직접 사용하여 새로운 플레이 애플리케이션을 생성할 수 있다.

> 우선 필요한 [sbt](http://www.scala-sbt.org/)를 설치하라.

새로운 애플리케이션을 위해 새로운 디렉터리를 생성하고 sbt 빌드 스크립트에 2가지를 설정하라.

`project/plugins.sbt`에 다음과 같이 추가하라:

```scala
// Typesafe 저장소
resolvers += "Typesafe repository" at "https://repo.typesafe.com/typesafe/releases/"

// 플레이 프로젝트를 위한 플레이 sbt 플러그인 사용
addSbtPlugin("com.typesafe.play" % "sbt-plugin" % "%PLAY_VERSION%")
```

확실하게 하기 위해 `%PLAY_VERSION%`을 사용하고자 하는 정확한 버전으로 변경하라. 만약 스냅샷 버전을 사용하고 싶으면, 추가적인 리졸버를 명시해야 한다.
```
// Typesafe 스냅샷
resolvers += "Typesafe Snapshots" at "https://repo.typesafe.com/typesafe/snapshots/"
```

사용될 적절한 sbt 버전을 확정하기 위해, 다음처럼 `project/build.properties`에 명시하라:

```
sbt.version=0.13.8
```

자바 프로젝트를 위한 `build.sbt`:

```scala
name := "my-first-app"

version := "1.0"

lazy val root = (project in file(".")).enablePlugins(PlayJava)
```

...또는 스칼라 프로젝트를 위한 `build.sbt`:

```scala
name := "my-first-app"

version := "1.0.0-SNAPSHOT"

lazy val root = (project in file(".")).enablePlugins(PlayScala)
```

이제 디렉터리에서 sbt 콘솔으로 프로젝트를 시작할 수 있다.

```bash
$ cd my-first-app
$ sbt
```

sbt는 프로젝트를 로드하고 의존성을 가져올 것이다.