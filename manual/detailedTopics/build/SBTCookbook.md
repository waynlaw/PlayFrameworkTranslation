<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# SBT 쿡북

## `play run`에 훅 액션

`PlayRunHook`을 확장하여 `play run` 명령을 액션에 적용할 수 있다.
이 트레이트는 다음과 같은 메소드를 재공한다.

 * `beforeStarted(): Unit`
 * `afterStarted(addr: InetSocketAddress): Unit`
 * `afterStopped(): Unit`

`beforeStarted` 메소드는 플레이 애플리케이션이 시작되기 전에 호출되지만 모든 "before run" 태스크가 완료된 이후에 호출된다.

`afterStarted` 메소드는 플레이 애플리케이션이 실행되기 이전에 호출된다.

`afterStopped` 메소드는 플레이 프로세스가 종료되기 이전에 호출된다.

> **주의:** 다음 예제는 플레이 run 훅으로 어떻게 명령어를 시작하고 종료할 수 있는지 설명한다.
> 가까운 미래에 [sbt-web](https://github.com/sbt/sbt-web)은 SBT 빌드와 Grunt를 통합하는 더 나은 방법을 제공할 것이다.

애플리케이션이 시작되기 전에 `grunt로 웹 애플리케이션을 빌드 하고 싶을 수 있다.
먼저 `PlayRunHook`을 확장하여 `project/` 디렉터리에 스칼라 object를 생성할 필요가 있다.
이름은 Grunt.scala`로 명시하자.

```scala
import play.PlayRunHook
import sbt._

object Grunt {
  def apply(base: File): PlayRunHook = {

    object GruntProcess extends PlayRunHook {

      override def beforeStarted(): Unit = {
        Process("grunt dist", base).run
      }
    }

    GruntProcess
  }
}
```

그 후에 `build.sbt파일에 이 훅을 등록해야 한다.

```scala
import Grunt._
import play.PlayImport.PlayKeys.playRunHooks

playRunHooks <+= baseDirectory.map(base => Grunt(base))
```

이는 애플리케이션이 시작하기 전에 `baseDirectory`에서 `grunt dist` 명령어를 실행할 것이다.

이제 우리는 어떤일이 일어났을 때 웹 애플리케이션을 재빌드하고 변화를 감지하기 위해 `grunt watch` 명령어를 수행하길 원한다.

```scala
import play.PlayRunHook
import sbt._

import java.net.InetSocketAddress

object Grunt {
  def apply(base: File): PlayRunHook = {

    object GruntProcess extends PlayRunHook {

      var process: Option[Process] = None

      override def beforeStarted(): Unit = {
        Process("grunt dist", base).run
      }

      override def afterStarted(addr: InetSocketAddress): Unit = {
        process = Some(Process("grunt watch", base).run)
      }

      override def afterStopped(): Unit = {
        process.map(p => p.destroy())
        process = None
      }
    }

    GruntProcess
  }
}
```

이전에 `grunt watch`를 수행하여 애플리케이션이 시작되었고 애플리케이션이 멈췄을때 우리는 grunt 프로세스를 파괴한다. `build.sbt`에는 아무 변경도 없어도 된다.

## 컴파일러 옵션 추가하기

예를 들어 feature 경고에 대해서 자세한 사항을 얻기 위해 feature flag를 추가하고 싶을 것이다.

```
[info] Compiling 1 Scala source to ~/target/scala-2.10/classes...
[warn] there were 1 feature warnings; re-run with -feature for details
```

간단하게 `-feature`를 `scalacOptions` 속성에 추가하자.

```scala
scalacOptions += "-feature"
```

## 추가적인 자산 디렉터리 추가하기

예를들어 `pictures` 폴더를 추가하여 추가적인 자산 디렉터리를 포함시킬 수 있다.

```scala
unmanagedResourceDirectories in Assets <+= baseDirectory { _ / "pictures" }
```

이는 이 폴더와 함께 `routes.Assets.at`을 사용하는 것을 허락한다.

## 문서 비활성화 하기

컴파일 시간을 줄이기 위해 문서 생성을 비활성화 할 수 있다.

```scala
sources in (Compile, doc) := Seq.empty

publishArtifact in (Compile, packageDoc) := false
```

첫 번째 줄은 문서 생성을 비활성화 하고 두 번째 줄은 문서 artifact 발행을 하지 않을 것이다.

## ivy 로깅 레벨 환경설정하기

기본적으로 `ivyLoggingLevel`는 `UpdateLogging.DownloadOnly`로 설정되어 있다. 다음과 같은 값으로 변경할 수 있다.

 * `UpdateLogging.Quiet` 오직 에러만 나타내어 준다.
 * `UpdateLogging.FULL` 전부를 로그로 보여준다.

예를 들어 오직 에러만 확인하고 싶다면 다음과 같이 할 수 있다.

```scala
ivyLoggingLevel := UpdateLogging.Quiet
```

## 테스트를 병렬로 수행하고 포크하기

기본적으로 병렬 실행은 비활성화 되어있고 포크하기는 활성화 되어있다. `fork in Test`에서 `parallelExecution in Test` 설정으로 이 행동에 대해서 설정할 수 있다.

```scala
parallelExecution in Test := true

fork in Test := false
```
