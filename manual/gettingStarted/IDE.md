<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# IDE 설정하기

플레이를 사용하는 것은 쉽다. 코드를 변경할 때마다 플레이가 자동으로 컴파일을 하고 새로고침을 하기 때문에 고급 IDE를 사용하지 않아도 되며 단순 텍스트 편집기 만으로도 쉽게 작업할 수 있다.

그러나 최신 자바, 스칼라 IDE를 사용하면 자동 완성, 지속적인 컴파일, 리팩토링 지원, 디버깅과 같이 더 생산적인 기능들을 활용할 수 있다.

## 이클립스

### sbteclipse 설정하기

플레이는 [sbteclipse](https://github.com/typesafehub/sbteclipse) 4.0.0 이상의 버전이 필요하다.

```scala
addSbtPlugin("com.typesafe.sbteclipse" % "sbteclipse-plugin" % "4.0.0")
```

프로젝트에 스칼라 소스가 포함되어있다면 [Scala IDE](http://scala-ide.org/)를 설치해야 한다.

스칼라 IDE를 설치하고 싶지 않고 자바 소스만 사용한다면 다음과 같이 설정하라:

```scala
EclipseKeys.projectFlavor := EclipseProjectFlavor.Java           // 자바 프로젝트. 스칼라 IDE를 사용하지 않음.
EclipseKeys.createSrc := EclipseCreateSrc.ValueSet(EclipseCreateSrc.ManagedClasses, EclipseCreateSrc.ManagedResources)  // 뷰나 라우트를 위해 생성된 .scala파일 대신 .class파일을 사용함
EclipseKeys.preTasks := Seq(compile in Compile)                  // 이클립스 파일을 생성하기 전에 프로젝트를 컴파일 하여, 뷰나 라우트를 위한 .class파일을 최신으로 갱신함
```

### 환경설정 생성하기

플레이는 [Eclipse](https://eclipse.org/)환경설정을 간단하게 해주는 명령어를 제공한다. 플레이 애플리케이션을 이클립스 프로젝트로 변경하기 위해서는 `eclipse` 명령어를 사용하라:

```bash
[my-first-app] $ eclipse
```

이용 가능한 jar 소스를 확인하려면 아래처럼 옵션을 줄 수 있다.(이는 시간이 좀 더 소모될 수 있으며, 일부 소스 파일이 빠질 수 있다.)

```bash
[my-first-app] $ eclipse with-source=true
```

> 서브-프로젝트를 모아서 사용한다면, `build.sbt`안에 `skipParents`를 적절하게 설정해야 한다:

```scala
EclipseKeys.skipParents in ThisBuild := false
```

또는 플레이 콘솔에 다음과 같이 입력해야 하라:

```bash
[my-first-app] $ eclipse skip-parents=false
```

You then need to import the application into your Workspace with the **File/Import/General/Existing project…** menu (compile your project first).

그리고 애플리케이션을 **File/Import/General/Existing project…** 메뉴로 워크스페이스를 불러와야 한다.
[[images/eclipse.png]] 

디버그 하려면, 애플리케이션을 `activator -jvm-debug 9999 run` 명령어로 시작하고 이클립스의 프로젝트에서 마우스 우클릭 후에 **Debug As**, **Debug Configurations**를 선택하라. **Debug Configurations** 다이얼로그의 **Remote Java Application**를 우클릭 한 뒤 **New**를 선택하라.  **Port**를 9999로 변경하고 **Apply**를 클릭하라. 이제 실행중인 애플리케이션에 연결하기 위해서 **Debug**를 선택할 수 있다. 디버깅 세션을 멈추는 것이 서버를 멈추는 것은 아니다.

> **팁**: 파일이 변경될 때 직접 컴파일하는 기능을 활성화하려면 `~run`을 사용해서 애플리케이션을 실행할 수 있다. 이 방법은 `view`내에 새 템플릿을 생성하는 것은 자동으로 감지 되며, 파일이 변경되면 자동으로 컴파일 된다. 단지 `run`명령어로 실행한 경우에는 브라우저를 매번 `새로고침`해주어야 한다.

만일 클래스패스의 변경처럼 애플리케이션의 중요한 변경이 있으면, 설정 파일을 다시 생성하기 위해 `eclipse`명령어를 사용하라.

> **팁**: 팀으로 작업하는 경우라면 이클립스 환경설정 파일을 형상관리 도구에 올리지 않도록 주의히라!

생성된 환경설정 파일은 프레임워크 설치에 대한 절대 참조를 포함하고 있다. 이 설정 값은 고유한 값이며, 팀으로 작업하는 경우에는 각 개발자가 자신만의 이클립스 설정을 별도로 유지해야 한다.

## 인텔리J

[인텔리J IDEA](https://www.jetbrains.com/idea/)를 사용하면 프롬프트를 사용하지 않아도 플레이 애플리케이션을 빠르게 생성할 수 있다. IDE 이외에 어떤 설정도 필요하지 않으며, SBT 빌드 도구는 적절한 라이브러리를 다운로드하고 프로젝트에 빌드에 필요한 의존성을 해결한다.

Before you start creating a Play application in IntelliJ IDEA, make sure that the latest [Scala Plugin](http://www.jetbrains.com/idea/features/scala.html) is installed and enabled in IntelliJ IDEA. Even if you don't develop in Scala, it will help with the template engine and also resolving dependencies.
인텔리J에서 플레이 애플리케이션을 생성하기 전에, 최신 버전의 [Scala Plugin](http://www.jetbrains.com/idea/features/scala.html)이 인텔리J에 설치되고 활성화 되었는지 확인하라. 스칼라를 사용해서 개발하지 않더라도, 템플릿 엔진 사용에 도움이 되며, 의존성을 해결해 줄 것이다.

플레이 애플리케이션 생성하기:

1. ***New Project*** 마법사를 열고, ***Scala*** 하위의 ***Play 2.x*** 선택한 뒤 ***Next***를 클릭하라.
2. 프로젝트의 정보를 입력하고 ***Finish***를 클릭하라.

인텔리J는 SBT를 사용하여 빈 프로젝트를 생성한다.

또한 기존의 플레이 프로젝트를 불러올 수 있다.

플레이 프로젝트 불러오기:

1. 프로젝트 마법사를 열고, ***Import Project***를 선택하라.
2. 열린 창에서 불러오길 원하는 프로젝트를 선택한 다음, ***OK***를 클릭하라.
3. 마법사의 다음 페이지에서, ***Import project from external model*** 옵션을 선택하고, ***SBT project***를 선택한 다음 ***Next***를 클릭하라. 
4. 마법사의 다음 페이지에서, 필요한 불러오기 옵션을 더 선택한 다음 ***Finish***를 클릭하라. 

Check the project's structure, make sure all necessary dependencies are downloaded. You can use code assistance, navigation and on-the-fly code analysis features.
프로젝트 구조를 확인하고 모든 필요한 의존성이 다운로드 되었는지 확인하라. 코드 지원이나 실시간 코드 분석, 탐색 기능을 사용할 수 있다.

생성된 프로젝트를 실행하고 결과를 기본 브라우저의 `http://localhost:9000`에서 확인할 수 있다.

1. 프로젝트 트리에서 애플리케이션을 우클릭 하라.
2. 메뉴의 목록에서 ***Run Play2 App***를 선택하라.

기본 Run/Debug 환경설정을 사용하여 플레이 애플리케이션을 쉽게 디버깅 할 수 있다.

보다 상세한 정보를 위해 아래 주소의 플레이 프레임워크 2.x 튜토리얼을 확인하라:

<https://confluence.jetbrains.com/display/IntelliJIDEA/Play+Framework+2.0> 

### 에러 페이지에서 소스코드로 이동하기

Using the `play.editor` configuration option, you can set up Play to add hyperlinks to an error page. Since then, you can easily navigate from error pages to IntelliJ, directly into the source code (you need to install the Remote Call <https://github.com/Zolotov/RemoteCall> IntelliJ plugin first).
`play.editor` 옵션을 사용하여 플레이에서 에러 페이지로 하이퍼링크를 추가할 수 있다. 그리고 에러 페이지에서 인텔리J의 소스코드로 쉽게 직접 이동할 수 있다. (우선 인텔리J Remote Call 플러그인 을 설치하라. <https://github.com/Zolotov/RemoteCall>).

Remote Call 플러그인을 설치하고 다음 옵션과 함께 어플리케이션을 실행하라:
`-Dplay.editor=http://localhost:8091/?message=%s:%s -Dapplication.mode=dev`

## 넷빈즈

### 환경설정 생성하기

Play does not have native [Netbeans](https://netbeans.org/) project generation support at this time, but there is a Scala plugin for NetBeans which can help with both Scala language and SBT:
플레이는 [Netbeans](https://netbeans.org/) 프로젝트 생성기능을 지원하지 않는다. 하지만 넷빈즈를 위한 스칼라 언어와 SBT를 사용하도록 도와주는 스칼라 플러그인이 있다.

<https://github.com/dcaoyuan/nbscala>

또한 넷빈즈 프로젝트를 생성하기 위한 SBT 플러그인이 있다: 

<https://github.com/dcaoyuan/nbsbt>

## ENSIME

### ENSIME 설치하기

<https://github.com/ensime/ensime-emacs>에서 설치하는 방법을 확인할 수 있다.

### 환경설정 생성하기

project/plugins.sbt 파일을 수정하고, 다음 라인을 추가하라. (우선 <https://github.com/ensime/ensime-sbt>에서 플러그인의 최신 버전을 확인할 필요가 있다.)

```scala
addSbtPlugin("org.ensime" % "ensime-sbt" % "0.1.5-SNAPSHOT")
```

플레이를 시작하라:

```bash
$ activator
```

플레이 콘솔에서 'ensime generate'를 입력하라. 플러그인은 플레이 프로젝트 최상단에 .ensime 파일을 생성할 것이다.

```bash
$ [MYPROJECT] ensime generate
[info] Gathering project information...
[info] Processing project: ProjectRef(file:/Users/aemon/projects/www/MYPROJECT/,MYPROJECT)...
[info]  Reading setting: name...
[info]  Reading setting: organization...
[info]  Reading setting: version...
[info]  Reading setting: scala-version...
[info]  Reading setting: module-name...
[info]  Evaluating task: project-dependencies...
[info]  Evaluating task: unmanaged-classpath...
[info]  Evaluating task: managed-classpath...
[info] Updating {file:/Users/aemon/projects/www/MYPROJECT/}MYPROJECT...
[info] Done updating.
[info]  Evaluating task: internal-dependency-classpath...
[info]  Evaluating task: unmanaged-classpath...
[info]  Evaluating task: managed-classpath...
[info]  Evaluating task: internal-dependency-classpath...
[info] Compiling 5 Scala sources and 1 Java source to /Users/aemon/projects/www/MYPROJECT/target/scala-2.9.1/classes...
[info]  Evaluating task: exported-products...
[info]  Evaluating task: unmanaged-classpath...
[info]  Evaluating task: managed-classpath...
[info]  Evaluating task: internal-dependency-classpath...
[info]  Evaluating task: exported-products...
[info]  Reading setting: source-directories...
[info]  Reading setting: source-directories...
[info]  Reading setting: class-directory...
[info]  Reading setting: class-directory...
[info]  Reading setting: ensime-config...
[info] Wrote configuration to .ensime
```

### ENSIME 시작하기

Emacs에서 M-x ensime을 실행하고, 화면에 나타나는 지시를 따라하라.

That's all there is to it. You should now get type-checking, completion, etc. for your Play project. Note, if you add new library dependencies to your play project, you'll need to re-run "ensime generate" and re-launch ENSIME.

### 추가 정보

<https://github.com/ensime/ensime-emacs>에서 ENSIME README를 확인할 수 있으며, 궁금한 사항이 있다면 <https://groups.google.com/forum/?fromgroups=#!forum/ensime>에서 문의할 수 있다.

## 유용한 스칼라 플러그인들

스칼라는 새로운 프로그래밍 언어이기 때문에 IDE의 기본 기능 보다는 플러그인으로 기능이 많이 제공된다.

1. 이클립스 스칼라 IDE: <http://scala-ide.org/>
2. 넷빈즈 스칼라 플러그인: <https://github.com/dcaoyuan/nbscala>
3. 인텔리J IDEA 스칼라 플러그인: <http://confluence.jetbrains.net/display/SCA/Scala+Plugin+for+IntelliJ+IDEA>
4. 인텔리J IDEA의 플러그인은 개발이 진행중이기 때문에 빌드 중 사소한 기능 장애가 있을 수 있다.
5. Nika (11.x) 플러그인 저장소: <https://www.jetbrains.com/idea/plugins/scala-nightly-nika.xml>
6. Leda (12.x) 플러그인 저장소: <https://www.jetbrains.com/idea/plugins/scala-nightly-leda.xml>
7. 인텔리J IDEA 플레이 플러그인 (Leda 12.x에서만 사용할 수 있다): <http://plugins.intellij.net/plugin/?idea&pluginId=7080>
8. ENSIME - Emacs의 스칼라 IDE 모드: <https://github.com/aemoncannon/ensime>
(see below for ENSIME/Play instructions)


