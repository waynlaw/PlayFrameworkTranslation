<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 소스로 플레이 만들기

초기 베타 릴리즈 이후 최신 개선 사항과 버그 수정에 대한 이점을 위해, 소스로 부터 플레이를 컴파일 하고 싶을 수 있다. 소스를 가져오기 위해 [Git client](http://git-scm.com/)가 필요하다.

## 소스 가져오기

먼저 쉘에서 플레이 소스를 체크아웃한다.

```bash
$ git clone git://github.com/playframework/playframework.git
```

이후에 `playframework/framework` 디렉터리로 이동하고 sbt 빌드 콘솔으로 들어가기 위해 `build` 스크립트를 수행하자.

```bash
$ cd playframework/framework
$ ./build
> publishLocal
```

이렇게 하면 빌드가 되고 기본 스칼라 버전으로 플레이가 발행된다(현재는 2.10.4). 모든 버전을 위한 발행을 하고 싶다면 아래와 같이 빌드를 수행할 수 있다.

```bash
> +publishLocal
```

또는 특정 스칼라 버전의 발행을 위해서 아래와 같이 할 수도 있다.

```bash
> +++2.11.5 publishLocal
```

## 문서 빌드하기

문서는 또한 마크다운 파일으로 `playframework/documentation`에 있다. HTML로 보기 위해 다음과 같이 실행한다.

```bash
$ cd playframework/documentation
$ ./build run
```

[http://localhost:9000/@documentation](http://localhost:9000/@documentation)에서 문서를 볼 수 있다.

플레이 문서를 개발하기 위한 더 자세한 사항을 알고 싶다면 [[Documentation Guidelines|Documentation]]를 살펴보자.

## 테스트 수행하기

sbt 콘솔에서 `test` 태스크를 사용하여 기본 테스트를 수행할 수 있다.

```
> test
```

발행하기 처럼 앞에 `+`와 함께 명령하면 지원하는 모든 스칼라 버전에 대해서 테스트를 수행한다.

플레이 PR 유효성 검증은 기본 테스트에서 약간 더 나아가 스크립트화된 테스트, 문서 코드 샘플 테스트 그리고 플레이 액티베이터 템플릿 테스트를 수행한다. 모든 테스트를 수행하기 위해 `framework/runtests` 스크립트를 실행하자.

```
$ cd playframework/framework
$ ./runtests
```

## 프로젝트에서 사용하기

소스로부터 빌드한 플레이 버전을 사용하여 프로젝트를 컴파일하고 실행하기 위해서는 약간에 환경설정이 필요하다.

기존 플레이 프로젝트로 이동하여 다음처럼 `project/plugins.sbt`를 수정하자.

```
// Change the sbt plugin to use the local Play build (2.4.0-SNAPSHOT)
addSbtPlugin("com.typesafe.play" % "sbt-plugin" % "2.4.0-SNAPSHOT")
```

이전에 한번 수행했었다면 콘솔을 실행하고 프로젝트와 표준적으로 상호작용할 수 있다.

```
$ cd <projectdir>
$ activator
```

## 이클립스에서 코드 사용하기

[Stackoverflow](http://stackoverflow.com/questions/10053201/how-to-setup-eclipse-ide-work-on-the-playframework-2-0/10055419#10055419)에서 코드에서 동작하기 위해 이클립스를 어떻게 설정할 수 있는지에 대한 정보를 찾을 수 있을 것이다.

