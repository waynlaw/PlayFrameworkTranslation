<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 공용 자산으로 작업하기

이 섹션은 자바스크립트, CSS, 이미지처럼 애플리케이션의 정적 자원을 제공하는 방법에 대해서 설명한다.

플레이에서 공용 리소스를 제공하는 것은 다른 HTTP 요청을 제공하는 것과 동일하다. 클라이언트에게 CSS, 자바스크립트, 이미지 파일을 분배하기 위해 컨트롤러/액션 경로를 통해 일반 리소스와 동일한 라우팅을 사용한다.

## public/ 폴더

규칙에 따라 공용 자산은 애플리케이션의 `public` 폴더에 저장된다. 이 폴더는 원하는 방법으로 구성할 수 있으며, 우리는 다음 구조를 추천한다.

```
public
 └ javascripts
 └ stylesheets
 └ images
```

만약에 이 구조를 따르면 시작하는 것은 아주 간단하다. 그러나 어떻게 동작하는지 한번 이해하고나면 수정하는 것 역시 간단하다.

## WebJars

[WebJars](http://www.webjars.org/)는 액티베이터와 sbt의 일부로 규약에 따른 패키징 매커니즘과 편리함을 제공한다. 예를 들어 간단하게 빌드 파일에 아래처럼 의존성을 추가하여 유명한 [부트스트랩 라이브러리](http://getbootstrap.com/)를 사용한다고 선언할 수 있다.

```scala
libraryDependencies += "org.webjars" % "bootstrap" % "3.2.0"
```

WebJars는 편의를 위해 자동으로 공용 자산과 관련된 `lib` 폴더에 해당 라이브러리를 추출한다. 예를 들어 [RequireJs](http://requirejs.org/)의 의존성을 선언하면, 다음 라인처럼 뷰에서 해당 라이브러리를 참조할 수 있다.

```html
<script data-main="@routes.Assets.at("javascripts/main.js")" type="text/javascript" src="@routes.Assets.at("lib/requirejs/require.js")"></script>
```

`lib/requirejs/require.js` 경로를 주의깊게 살펴보자. `lib`폴더는 WebJar 자산을 지시하며, `requirejs`폴더는 WebJar artifactId와 부합하며, `require.js`는 WebJar의 루트에 필요한 자산을 의미한다.

## 어떻게 공용 자산을 패키징할 할까?

빌드 과정 동안, `public` 폴더의 내용은 애플리케이션의 클래스패스에 추가되고 처리 된다.

애플리케이션을 패키징할 때 모든 하위 프로젝트를 포함하여 애플리케이션의 모든 자산은 `target/my-first-app-1.0.0-assets.jar`에 하나의 jar로 합쳐진다. 이 jar는 배포에 포함되어지며, 그래서 플레이 애플리케이션은 이를 제공할 수 있게 된다. 이 jar는 또한 CDN이나 리버스 프록시의 자산으로 배포 될 수 있다.

## Assets 컨트롤러

플레이는 공용 자산을 제공하기 위한 기본 컨트롤러를 가지고 있다. 기본적으로 이 컨트롤러는 캐싱, Etag, gzip과 압축에 대한 지원을 하고 있다.

이 컨트롤러는 기본적인 플레이 JAR의 `controllers.Assets`로 사용할 수 있으며 2개의 파라미터를 가진 단일 `at` 액션을 정의한다.

```
Assets.at(path: String, file: String)
```

`path` 파라미터는 액션에 의해 관리되는 디렉터리로 정의하거나 수정할 수 있다. `file` 파라미터는 보통은 요청의 경로에서 동적으로 추출된다.

`conf/routes` 파일에 `Assets` 컨트롤러의 실질적인 맵핑은 다음과 같다.

```
GET  /assets/*file        controllers.Assets.at(path="/public", file)
```

`.*` 정규표현식에 매치되도록 `*file`으로 동적 부분을 정의한 것을 주의깊게 살펴보자.

예를들어, 서버로 이런 요청을 보내면

```
GET /assets/javascripts/jquery.js
```

라우터는 `Assets.at` 액션을 다음의 파라미터로 호출할 것이다.

```
controllers.Assets.at("/public", "javascripts/jquery.js")
```

이 액션은 해당 파일이 존재하면 파일을 찾아서 제공할 것이다.

## 공용 자산을 위한 리버스 라우팅

라우팅 파일에 맵핑된 모든 컨트롤러를 위해 리버스 컨트롤러는 `controllers.routes.Assets`를 생성한다. 공용 리소스를 가져오기 위해 사용할 URL으로 이를 사용할 수 있다. 예를 들어 템플릿에서 다음과 같이 사용할 수 있다.

```html
<script src="@routes.Assets.at("javascripts/jquery.js")"></script>
```

이는 다음과 같은 URL을 생성할 것이다.

```html
<script src="/assets/javascripts/jquery.js"></script>
```

리버스 라우팅을 위해 `folder` 파라미터를 명시하지 않았음을 주의깊게 보자. 왜냐하면 라우트 파일은 `folder` 파라미터를 고정시킨 `Assets.at` 액션을 위한 단일 맵핑을 정의한다. 그래서 해당 폴더를 명시할 필요가 없다.

그럼에도 불구하고 다음처럼 `Assets.at`을 위해 2개의 맵핑을 정의하면

```
GET  /javascripts/*file        controllers.Assets.at(path="/public/javascripts", file)
GET  /images/*file             controllers.Assets.at(path="/public/images", file)
```

리버스 라우팅을 사용할 때 각각의 파라미터를 명시해주어야 한다.

```html
<script src="@routes.Assets.at("/public/javascripts", "jquery.js")"></script>
<img src="@routes.Assets.at("/public/images", "logo.png")" />
```

## 공용 자산을 위한 리버스 라우팅과 핑거프린팅

sbt-web은 플레이에서 고도의 자산 파이프라인의 개념을 빌드 파일에 구성할 수 있다.

```scala
pipelineStages := Seq(rjs, digest, gzip)
```

위는 순서대로 RequireJs 최적화 (`sbt-rjs`), 다이제스터 (`sbt-digest`) 그리고 압축에 대한 (`sbt-gzip`) 것이다. 다른 태스크와 다르게, 이 태스크들은 선언된 순서대로 실행된다.

근본적으로 자산의 핑거프린트는 정적 자산을 브라우저에서 적극적인 캐싱을 통해 제공 하도록 한다. 이것은 이후에 사이트를 재 방문 했을 때 자원을 다운로드하는 것을 감소시켜 사용자에게 향상된 경험을 준다. 레일즈 또한 [자산 핑거프린팅](http://guides.rubyonrails.org/asset_pipeline.html#what-is-fingerprinting-and-why-should-i-care-questionmark)의 장점에 대해서 설명했다.

위에서 `plugins.sbt`에서 필요한 플러그인을 위한 `pipelineStages`의 정의와 `addSbtPlugin`정의가 시작 지점이다. 그런 다음 자산 버전이 무엇인지 플레이에 선언 해야한다. 다음의 라우트 파일의 내용은 모든 자산이 버전화 되었음을 나타낸다.

```scala
GET  /assets/*file  controllers.Assets.versioned(path="/public", file: Asset)
```

> `file`을 `file: Asset`으로 작성하여 자산을 나타낸 것을 확인하라.

예를들어 `scala.html`뷰의 내부에서 리버스 라우팅을 사용할 수 있다.

```html
<link rel="stylesheet" href="@routes.Assets.versioned("assets/css/app.css")">
```

자산의 핑거프린팅 사용을 적극 권장한다.

## Etag 지원

`Assets` 컨트롤러는 자동적으로 **ETag** HTTP 헤더를 관리한다. ETag값은 다이제스트나 리소스 이름 또는 파일의 마지막 수정 일자로 생성된다(만약 `sbt-digest`가 자산 파이프라인에 사용되고 있으면). 만약 리소스 이름이 파일을 포함하면 JAR파일의 마지막 수정 일자가 사용된다.

웹 브라우저가 **Etag**로 명시적인 요청을 하면 서버는 **304 NotModified**로 응답할 수 있다.

## Gzip 지원

만약 동일한 이름이지만 `.gz` 접사를 사용하는 자원을 발견하면 `Assets` 컨트롤러는 후자를 제공하고 HTTP 헤더에 다음을 추가한다.

```
Content-Encoding: gzip
```

빌드에 `sbt-gzip` 플러그인을 포함하고 `pipelineStages`에 위치시키는 것이 gzip 파일을 생성하기 위해 필요한 전부이다.

## `Cache-Control` 지시자

일반적으로 캐싱의 목적으로 Etag를 사용하면 충분하다. 그러나 특별한 리소스를 위해 커스텀 `Cache-Control` 헤더를 명시하고 싶다면, `application.conf`파일에 명시할 수 있다.

```
# Assets configuration
# ~~~~~
"assets.cache./public/stylesheets/bootstrap.min.css"="max-age=3600"
```

## 관리되는 자산

플레이 2.3 부터 관리되기 시작한 자산은 [sbt-web](https://github.com/sbt/sbt-web#sbt-web)기반의 플러그인으로 처리된다. 2.3부터 플레이는 기본적으로 자산처리를 커피스크립트, LESS, 자바스크립트 linting(클로저 컴파일러), RequireJs 최적화의 형태로 자산 처리를 관리한다. 다음 섹션에서 sbt-web과 어떻게 해서 2.2와 동등한 기능을 성취할 수 있는지 설명할 것이다. 플레이는 많은 플러그인을 sbt-web에서 사용할 수 있으므로 자산 처리 기술에 한계가 없음을 기억하라. 사용 가능한 플러그인이 무엇인지에 대해 더 자세히 학습하기 위해 [sbt-web](https://github.com/sbt/sbt-web#sbt-web) 프로젝트를 살펴보자.

대부분의 플러그인이 sbt-web의 [js-engine plugin](https://github.com/sbt/sbt-js-engine)을 사용한다. js-engine은 JVM내에서 아주 좋은 [Trireme](https://github.com/apigee/trireme#trireme) 프로젝트나 뛰어난 성능을 위해 직접 [Node.js](http://nodejs.org/)를 통해 Node API로 작성된 플러그인을 실행할 수 있다. 이 도구들은 오직 개발 주기에만 사용되어야 하며 플레이 애플리케이션의 실환경 실행과는 연관이 없음을 기억하라. 만약 설치된 Node.js를 가지고 있다면 다음의 환경 변수를 선언하는 것이 좋다. 예를들어 유닉스에서 `SBT_OPTS`을 다음과 같이 선언할 수 있다.

```bash
export SBT_OPTS="$SBT_OPTS -Dsbt.jse.engineType=Node"
```

위의 선언은 sbt-web 플러그인이 실행될 때 Node.js를 사용하도록 한다.