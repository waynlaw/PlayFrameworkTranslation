<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# LESS CSS 사용하기

[LESS CSS](http://lesscss.org/)는 동적 스타일시트 언어이다. LESS CSS는 변수와 믹스인등을 제공하여 CSS 파일을 작성할 수 있어 상당한 유연함을 제공한다.

플레이에서 컴파일 할 수 있는 자산은 `app/assets` 디렉터리에 정의되어야 한다. 이 디렉터리에 파일들은 빌드 과정에서 처리되며 LESS 소스도 표준 CSS파일로 컴파일 된다. 생성된 CSS 파일은 다른 관리되지 않는 자원 처럼 `public/` 폴더에 동일하게 보통의 리소스처럼 배포되며, 컴파일된 내역을 사용하는게 다른 CSS파일과 별반 다르지 않음을 의미한다.

예를 들어 LESS 소스 파일 `app/assets/stylesheets/main.less`은 보통의 자바스크립트 리소스 `public/stylesheets/main.css`처럼 사용가능하다. 플레이는 `main.less`를 자동으로 컴파일한다. 다른 LESS 파일은 `build.sbt파일에 추가해야 컴파일 된다.

```scala
includeFilter in (Assets, LessKeys.less) := "foo.less" | "bar.less"
```

LESS 소스는 `assets` 명령으로 자동으로 컴파일되거나 개발 모드로 플레이를 실행중이라면 브라우저에서 페이지를 새로고침하면 컴파일된다. 컴파일 에러가 발생하면 브라우저에 다음과 같이 표시된다.

[[images/lessError.png]]

## 분리된(partial) LESS 소스 파일로 작업하기

LESS 소스 파일을 몇 개의 라이브러리로 분리할 수 있고 LESS의 `import`기능을 사용할 수 있다.

라이브러리 파일이 하나하나가 컴파일되거나 임포트되는 것을 막기 위해 컴파일러에 의해 스킵되도록 할 필요가 있다. 이렇게 하기 위해 분리된 소스 파일은 `_myLibrary.less`처럼 언더스코어 (`_`) 문자를 접두사로 사용할 수 있다. 아래에 보이는 환경설정이 분리된 라이브러리를 무시하도록 활성화한다.

```scala
includeFilter in (Assets, LessKeys.less) := "*.less"

excludeFilter in (Assets, LessKeys.less) := "_*.less"
```


## 레이아웃

여기 프로젝트에 LESS를 사용한 레이아웃 예제가 있다.

```
app
 └ assets
    └ stylesheets
       └ main.less
       └ utils
          └ reset.less
          └ layout.less    
```

다음은 `main.less` 소스 파일이다.

```css
@import "utils/reset.less";
@import "utils/layout.less";

h1 {
    color: red;
}
```

결과적으로 CSS 파일은 `public/stylesheets/main.css`로 컴파일되고 이 파일을 템플릿에서 평범한 공통 자산으로 사용할 수 있다.

```html
<link rel="stylesheet" href="@routes.Assets.at("stylesheets/main.css")">
```

## 부트스트랩과 함께 LESS 사용하기

[Bootstrap](http://getbootstrap.com/css/) LESS와 함께 사용되는 매우 대중적인 라이브러리이다.

라이브러리 의존성에 [WebJar](http://www.webjars.org/)를 추가해서 부트스트랩을 사용할 수 있다. 예를 들어 build.sbt 파일에 다음과 같이 추가할 수 있다.

```scala
libraryDependencies += "org.webjars" % "bootstrap" % "3.2.0"
```

sbt-web은 자동적으로 자산의 타겟 폴더인 lib 폴더에 WebJar를 추출할 것이다. 그러므로 부트스트랩을 사용하기 위해서 아래처럼 임포트 할 수 있다.

```less
@import "lib/bootstrap/less/bootstrap.less";

h1 {
  color: @font-size-h1;
}
```

## 환경설정

LESS 컴파일은 `PlayJava`나 `PlayScala` 플러그인을 사용할 때 처럼 plugins.sbt 파일에 간단하게 플러그인을 추가해 활성화 할 수 있다.

```scala
addSbtPlugin("com.typesafe.sbt" % "sbt-less" % "1.0.0")
```

플러그인의 기본 환경설정은 이미 충분하게 되어있다. 그러나 설정에 대해서 정보가 필요하다면 [플러그인 문서](https://github.com/sbt/sbt-less#sbt-less)를 참조하기 바란다.
