<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 라이브러리 의존성 관리하기

## 관리되지 않는 의존성

대부분의 사람들은 결국 의존성을 관리하게 된다. 의존성을 관리하게 되면 프로젝트를 더 잘 제어할 수 있겠지만 프로젝트를 시작할 때는 의존성을 관리하지 않는게 더 간단하다.

관리되지 않는 의존성은 프로젝트의 루트에 `lib/` 디렉터리를 만들고 jar 파일을 추가하면 된다. 이렇게 하면 애플리케이션의 클래스패스에 자동으로 추가될 것이다. 

의존성을 관리하지 않을 것이므로 `build.sbt`에 추가하지 않아도 되지만 `lib`외에 다른 디렉터리를 사용하려고하면 환경설정 키를 변경해주어야 한다.

## 관리되는 의존성

플레이는 의존성 관리를 구현하기 위해 아파치 sbt를 통해 아파치 Ivy를 사용하므로 메이븐이나 Ivy와 친근하다면 큰 문제없이 사용할 수 있다.

Most of the time you can simply list your dependencies in the `build.sbt` file. 
대부분은 `build.sbt`파일에 의존성을 간단하게 추가할 수 있다.

의존성을 정의하는 것은 다음 코드와 같다. `group`과 `artifact`, `revision`을 정의한다

```scala
libraryDependencies += "org.apache.derby" % "derby" % "10.4.1.3"
```

또는 부가적으로 `configuration`을 사용할 수도 있다.

```scala
libraryDependencies += "org.apache.derby" % "derby" % "10.4.1.3" % "test"
```

여러개의 의존성은 아래처럼 중복하여 추가할 수 있는데 스칼라 시퀀스를 사용하는 것이다.

```scala
libraryDependencies ++= Seq(
  "org.apache.derby" % "derby" % "10.4.1.3",
  "org.hibernate" % "hibernate-entitymanager" % "3.6.9.Final"
)
```

물론 Ivy를 통해 sbt는 모듈의 다운로드 위치를 알고 있다. 모듈이 기본 저장소 중 하나에 있다면 sbt는 잘 동작할 것이다.

### `%%`로 적절한 스칼라 버전 얻기

If you use `groupID %% artifactID % revision` instead of `groupID % artifactID % revision` (the difference is the double `%%` after the `groupID`), sbt will add your project’s Scala version to the artifact name. This is just a shortcut. 

만약에 `groupID %% artifactID % revision` 대신 `groupID % artifactID % revision` 를 사용하면 sbt는 아티펙트 이름으로 프로젝트의 스칼라 버전을 추가할 것이다. (차이점은 `groupID`이후에 `%%`이다.)

`%%` 없이 다음처럼 작성할 수도 있었지만

```scala
libraryDependencies += "org.scala-tools" % "scala-stm_2.9.1" % "0.3"
```

빌드를 위한 `scalaVersion`이 `2.9.1`라고 가정하면 다음처럼 작성할 수 있다.

```scala
libraryDependencies += "org.scala-tools" %% "scala-stm" % "0.3"
```

### 리졸버

sbt는 표준 메이븐2 저장소와 기본으로 타입세이프 릴리즈 저장소(<https://repo.typesafe.com/typesafe/releases>)를 사용한다. 의존성이 이 저장소에 없다면 Ivy가 이를 찾을 수 있도록 리졸버를 추가할 필요가 있다.

개인적인 리졸버를 추가하기 위해 `resolvers` 설정 키를 사용하자.

```scala
resolvers += name at location
```

예를 들면 다음과 같다.

```scala
resolvers += "sonatype snapshots" at "https://oss.sonatype.org/content/repositories/snapshots/"
```

sbt는 여러분의 개인 메이븐 저장소도 추가할 수 있다.


```scala
resolvers += (
    "Local Maven Repository" at "file:///"+Path.userHome.absolutePath+"/.m2/repository"
)
```

