<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 플레이 콘솔 사용하기

## 콘솔 실행하기

플레이 콘솔은 플레이 애플리케이션의 전체 개발 주기를 관리할 수 있도록 해주는 sbt기반의 개발 콘솔이다.

플레이 콘솔을 실행하기 위해 프로젝트의 디렉터리로 이동한 다음 Activator를 실행하라:

```bash
$ cd my-first-app
$ activator
```

[[images/console.png]]

## 도움말 살펴보기

`help` 명령어를 사용하여 사용할 수 있는 명령어에 대한 기본적인 도움말을 살펴볼 수 있다. 또한 명령어를 명시적으로 사용하여 명령어에 대한 도움말을 살펴볼 수 있다:

```bash
[my-first-app] $ help run
```

## 개발 모드로 서버 실행하기

개발 모드로 현재 애플리케이션을 실행하려면 `run` 명령어를 사용하라:

```bash
[my-first-app] $ run
```

[[images/consoleRun.png]]

이 모드에서 서버는 자동 새로고침 기능이 활성화된 상태로 실행되며 이는 요청이 있을 때마다 프로젝트를 확인하고 필요한 코드를 재컴파일 한다. 만일 필요하다면 애플리케이션이 자동으로 재시작될 것이다.

만약 컴파일 과정에서 오류가 발생하면, 컴파일 결과를 브라우저에서 바로 확인할 수 있다:

[[images/errorPage.png]]

서버를 정지시키기 위해서 `Crtl+D`를 입력하면 플레이 콘솔 프롬프트로 복귀할 수 있다.

## 컴파일 하기

플레이에서는 서버를 실행하지 않고도 컴파일을 할 수 있다. `compile` 명령어를 사용하라:

```bash
[my-first-app] $ compile
```

[[images/consoleCompile.png]]

## 대화형 콘솔 실행하기

Type `console` to enter the interactive Scala console, which allows you to test your code interactively:
대화형 스칼라 콘솔을 사용하기 위해 `console` 명령어를 입력하면 코드를 테스트해볼 수 있다.

```bash
[my-first-app] $ console
```


스칼라 콘솔에서 애플리케이션을 실행해 볼 수 있다(예. 데이터베이스 접근):
```bash
scala> new play.core.StaticApplication(new java.io.File("."))
```

[[images/consoleEval.png]] 

## 디버깅하기

콘솔에서 시작할 때 **JPDA** 디버그 포트를 지정하면 자바 디버거를 사용하여 접속할 수 있다. `activator -jvm-debug <port>` 명령어를 사용하라:

```
$ activator -jvm-debug 9999
```

JPDA 포트가 활성화되면, JVM 기동시 아래와 같이 로그가 나타난다:
```
Listening for transport dt_socket at address: 9999
```

## sbt 기능 사용하기

플레이 콘솔은 일반적인 sbt 콘솔이기 때문에 **조건부 실행**과 같은 기능을 사용할 수 있다.

예를 들어 `~ compile`를 사용하면

```bash
[my-first-app] $ ~ compile
```

소스 파일이 변경될 때마다 컴파일을 수행한다.

또한 `~ run`를 사용하면:

```bash
[my-first-app] $ ~ run
```

개발 서버가 실행 중에 자동 컴파일 기능이 활성화 된다.

동일한 방식으로 `~ test`를 사용하면 소스코드가 변경될 때마다 지속적으로 테스트를 수행한다:

```bash
[my-first-app] $ ~ test
```

## 플레이 명령을 직접 사용하기

플레이 콘솔을 수행하지 않고 명령어를 직접 입력할 수 있다. 예를 들어 `activator run`을 입력하면 플레이가 실행된다:

```bash
$ activator run
[info] Loading project definition from /Users/jroper/tmp/my-first-app/project
[info] Set current project to my-first-app (in build file:/Users/jroper/tmp/my-first-app/)

--- (Running the application from SBT, auto-reloading is enabled) ---

[info] play - Listening for HTTP on /0:0:0:0:0:0:0:0:9000

(Server started, use Ctrl+D to stop and go back to the console...)
```

The application starts directly. When you quit the server using `Ctrl+D`, you will come back to your OS prompt.

애플리케이션은 바로 실행될 것이며, `Ctrl+D`를 사용하여 애플리케이션을 중지하면 OS 프롬프트로 빠져 나오게 된다. 물론 여기에서도 **조건부 실행**을 사용할 수 있다

```bash
$ activator ~run
```
