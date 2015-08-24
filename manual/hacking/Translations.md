# 플레이 문서 번역하기

플레이 2.3 이상에서는 플레이 문서를 번역하고 최신으로 유지하기 위해 문서 번역가를 돕은 인프라를 제공하다.

As described in the [[Documentation Guidelines|Documentation]], Play's documentation is written in markdown format with code samples extracted to external files.  Play allows the markdown components of the documentation to be translated, while allowing the original code samples from the English documentation to be included in the translated documentation.  This assists translators in maintaining translation quality - the code samples are kept up to date as part of the core Play project, while the translated descriptions have to be maintained manually.

[[Documentation Guidelines|Documentation]]에서 설명한 것 처럼 플레이의 문서는 마크다운 형식으로 작성되며 코드 샘플은 외부 파일로 추출되어 있다. 플레이는 영어 문서의 원본 코드 샘플을 번역된 문서에 포함될 수 있도록 하면서 문서의 마크다운 컴포넌트가 번역될 수 있도록 한다. 이렇게 해서 번역가가 번역 문서의 품질을 유지할 수 있도록 지원한다. - 번역된 문서는 직접 유지보수할 수 있도록 하면서 코드 샘플은 핵심 플레이 프로젝트의 일부로 최신으로 유지된다.

이에 더하여 플레이는 또한 번역된 문서의 무결성을 검증하기 위한 도구를 제공한다. 이는 번역 문서에서 모든 내부 링크나 코드 조각으로의 링크에 대한 검증도 포함한다.

## 사전 준비사항

우선 `activator`나 `sbt`가 설치되어 있어야 한다. 또한 문서 번역을 위한 브랜치를 위해 플레이의 저장소를 복제하거나 이것으로부터 복사한 것이 있다면 매우 유용할 것이다.

플레이 문서의 릴리즈되지 않은 버전을 번역한다면 우선 여러분의 머신의 로컬에 플레이의 버전을 빌드하고 발행할 필요가 있을 것이다. 플레이 프로젝트의 `framework` 디렉터리에서 다음의 절차를 수행하면 된다. 

```
./build publishLocal
```

## 번역을 진행하기 위한 설정

다음의 구조로 새로운 SBT 프로젝트를 생성하자.

```
translation-project
  |- manual
  | |- javaGuide
  | |- scalaGuide
  | |- gettingStarted
  | `- etc...
  |- project
  | |- build.properties
  | `- plugins.sbt
  `- build.sbt
```

`build.properties`는 SBT 버전을 명시해야 한다.

```
sbt.version=0.13.8
```

`plugins.sbt` 는 플레이 문서의 sbt 플러그인을 포함해야 한다. 

```scala
addSbtPlugin("com.typesafe.play" % "play-docs-sbt-plugin" % "%PLAY_VERSION%")
```

마지막으로 `build.sbt`는 플레이 문서 플러그인을 활성화 해야 한다.

```scala
lazy val root = (project in file(".")).enablePlugins(PlayDocsPlugin)
```

이제 문서 번역을 시작하기 위한 준비를 마쳤다.

## 문서 번역하기

먼저 번역 문서를 위한 서버를 실행하자. 번역문서를 위한 서버는 번역될 문서를 제공하여 주며, 이를 통해 여러분이 번역한 문서가 어떤 모습인지 확인할 수 있다. 이를 위해 `sbt`나 `activator` 둘 중 하나가 설치되어 있어야 하고 sbt를 예로 설명하면 다음과 같다.

```
$ sbt run
[info] Set current project to root (in build file:/Users/jroper/tmp/foo-translation/)
[info] play - Application started (Dev)
[info] play - Listening for HTTP on /0:0:0:0:0:0:0:0:9000

Documentation server started, you can now view the docs by going to http://0:0:0:0:0:0:0:0:9000
```

이제 브라우저에서 <http://localhost:9000>를 열면 기본 플레이 문서를 확인할 수 있을 것이다. 이제 첫 번째 페이지를 번역할 시간이다.

플레이 저장소로부터 여러분의 프로젝트로 마크다운 페이지를 복사하자. 여러분의 디렉터리 구조가 플레이의 디렉터리와 일치시켜야 한다. 그래야 코드 샘플의 동작을 보장할 수 있다.

예를 들어, `manual/scalaGuide/main/http/ScalaActions.md`를 번역할 것이라면, 여러분의 프로젝트에서 번역문서 역시 `manual/scalaGuide/main/http/ScalaActions.md`에 있어야 한다.

> **주의:** 여러분의 프로젝트에 플레이의 전체 문서를 복사하고 시작하고 싶을 수도 있다. 만약 이렇게 하려면 마크다운 파일들만 복사하여야 하며, 코드 샘플은 복사하지 않길 바란다. 만약 코드 샘플을 복사했다면 플레이가 코드 샘플을 오버라이드 할 것이고 여러분을 위해 자동으로 유지보수되는 코드 샘플의 이점을 잃어버리게 될 것이다.

이제 파일에 대해서 번역을 시작할 수 있다.

## 코드 샘플에 대한 처리

플레이 문서는 코드 샘플들이 가득하다. [[Documentation Guidelines|Documentation]]에서 설명한 것처럼, 이 코드 샘플은 마크다운 문서 외부에 있으며 컴파일되고 테스트되는 소스 파일 안에 있다. 이 파일으로 부터 조각들은 문서에서 다음의 문법을 사용하여 포함시킬 수 있다.

```
@[label](code/path/to/SourceFile.java)
```

일반적으로 문서 번역에서 이들 조각을 남겨두고 싶을 것이며, 플레이가 여러분의 번역 문서에 최신의 코드 조각으로 유지되도록 할 것이다. 

In some situations, it may make sense to override them.  You can either do this by putting the code directly in the documentation, using a fenced block, or by extracting them into your projects own compile code samples.  If you do that, checkout the Play documentation sbt build files for how you might setup SBT to compile them.

어떤 경우에는 오버라이드를 원할 수도 있다. 문서에서 펜스 블록으로 코드를 직접 넣을 수도 있고 스스로 컴파일한 코드 샘플을 여러분의 프로젝트로 추출해 낼 수도 있다. 이렇게 하길 원한다면 컴파일을 위해 어떻게 SBT를 설정하여야 하는지 플레이 문서의 sbt 필드 파일을 확인하자.

## 문서 유효성 확인하기

플레이 문서의 sbt 플러그인은 문서에 대한 유효성 검증 태스크를 제공하여 문서 전반에 걸쳐 간단한 테스트를 수행하고 코드 샘플의 참조나 링크의 무결성을 보장할 수 있다. 다음과 같이 수행할 수 있다.

```
sbt validateDocs
```

또한 플레이의 문서의 외부 사이트로의 링크를 확인할 수 있다. 플레이의 문서 링크가 인터넷에서 많은 사이트에 의존하고 있고, 실제로 이 유효성 검증 태스크가 몇몇 사이트에서 DDos 필터에 방아쇠가 되어 별도의 태스크로 분리되어 있다.

```
sbt validateExternalLinks
```

## 번역문서 리포트

Another very helpful tool provided by Play is a translation report, which shows which files have not been translated, and also tries to detect issues, for example, if the translation introduces new files, or if the translation is missing code samples.  This can particularly help when translating a new version of the documentation, since the addition or removal of code samples will often be a good signal that something has changed.

플레이가 제공하는 아주 도움이 되는 다른 도구는 어떤 파일이 번역되지 않았는지를 보여주고, 또한 번역 문서에 새로운 파일이 나타나거나 번역 문서에 코드 샘플이 없는 등의 이슈를 찾아내길 시도하는 번역문서 리포트이다. 특히 코드 샘플의 추가나 삭제가 어떤것이 변경되었다는 좋은 신호일 수 있으므로 문서의 새로운 버전을 번역할 때 많은 도움이 될 것이다.

To view the translation report, run the documentation server (like normal), and then visit <http://localhost:9000/@report> in your browser.  By default it will serve a cached version of the report if it has been generated in the past, you can rerun the report by clicking the rerun report link.

번역문서 리포트를 살펴보기 위해 일반적으로 번역 문서를 위한 서버를 실행시키고, 브라우저에서 <http://localhost:9000/@report>를 살펴보자. 기본적으로 리포트의 캐시된 버전을 제공하며 과거에 생성된 것이라면 리포트 링크 새로수행을 클릭하여 새로 수행할 수 있다.

## playframework.com에 번역문서 배포하기

[playframework.com](http://playframework.com)은 Git 저장소의 외부에서 문서를 제공한다. playframework.com에서 여러분의 번역 문서가 제공되길 원한다면 여러분의 문서를 GitHub 저장소에 넣어두어야 하며, playframework에 이를 추가할 수 있도록 플레이 팀에 연락을 취하길 바란다.

Git 저장소는 매우 특별한 형식일 필요가 있다. 현재 마스터 프랜치는 플레이의 최신 개발 버전의 문서를 위한 것이다. 플레이의 안정된 버전의 문서는 2.3.x 처럼 브랜치가 있어야 한다. 플레이의 특정 릴리즈를 명시한 번역 문서는 2.3.1처럼 이름과 함께 저장소의 태그로 제공되어야 한다.

Once the Play team has configured playframework.com to serve your translation, any changes pushed to your GitHub repository will be picked up within about 10 minutes, as playframework.com does a `git fetch` on all repos it uses once every 10 minutes.
이전에 플레이 팀이 playframework.com에 여러분의 문서를 제공하기 위해 설정했다면, GitHub 저장소로 어떤 변경사항이 푸쉬되면 매 10분마다 모든 저장소에 `git fetch`를 수행하여 약 10분 이내에 반영될 것이다.

## 번역 문서 버전 명시하기

기본적으로 `play-docs-sbt-plugin`는 스스로의 마크다운 파일 대비책과 플레이 문서의 코드샘플의 동일한 버전을 사용한다. 그래서 문서를 수행할 때 `plugins.sbt`에 `2.4.0`을 사용했다면 `2.4.0`의 문서의 코드 샘플을 가지게 될 것이다. `build.sbt`에 `PlayDocsKeys.docsVersion`를 설정하여 이 버전을 관리할 수 있다.

```scala
PlayDocsKeys.docsVersion := "2.3.1"
```

이는 특히 `play-docs-sbt-plugin`에 소개되기 이전인 `2.2.0`이전 처럼 이런 문서를 제공하고 싶은 경우 유용하다. `2.1.x`나 그 이전을 위해 문서는 jar 파일로 패키징되고 발행되지 않으며, 그래서 도구는 이전 버전에서는 동작하지 않는다.
