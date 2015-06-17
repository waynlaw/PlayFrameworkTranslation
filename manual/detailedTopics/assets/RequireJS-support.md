<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# RequireJS

[RequireJS](http://requirejs.org/)의 웹 페이지의 설명을 보면

> RequireJS는 자바스크립 파일과 모듈 로더이며 브라우저에서 사용될 때 최적화를 해준다. 그러나 다른 자바스크립트 환경에서도 사용할 수 있다. RequireJS 같은 모듈 스크립트 로더를 사용하면 코드의 질을 높이고 빠르게 향상시킬 수 있다.

이것이 실제로 의미하는 것은 자바스크립트를 모듈화하기 위해 [RequireJS](http://requirejs.org/) 사용할 수 있다는 것이다. RequireJS는 [비동기적 모듈 정의](http://wiki.commonjs.org/wiki/Modules/AsynchronousDefinition) (비슷한 아이디어가[CommonJS](http://www.commonjs.org/)에도 적용됨 )라고 부르는 준-표준(semi-standard) API를 구현하여 모듈화를 가능하게 한다. AMD를 사용하여 개발하면 서버측 _optimization_을 허용하면서 클라이언트측의 자바스크립트 모듈을 로드하고 해결할 수 있다. 서버측 모듈 최적화는 [UglifyJS 2](https://github.com/mishoo/UglifyJS2#uglifyjs-2)를 사용하여 결합되고 경량화(minified)된다.

관례적으로 RequireJS는 모듈 로더로 부트스트랩을 위한 main.js 파일을 추가해주어야 한다.

## 개발하기

RequireJS 옵티마이저는 일반적으로 `start`, `stage`나 `dist` 태스크처럼 배포가 수행되기 전까지 효과가 나타날 필요가 없다.

빌드에 WebJars을 사용하면 RequireJS 옵티마이저 플러그인은 WebJar에서 참조되는 자바스크립트 리소스가 자동으로 [jsdelivr (http://www.jsdelivr.com)CND에서 참조되어 있는지 확인한다. 또한 `.min.js`파일이 발견되면 `.js` 파일 대신 `.min.js` 파일이 사용된다. 추가적인 보너스는 HTML이 변경될 필요가 없다는 것이다!

## 환경설정

RequireJS 옵티마이저는 `PlayJava`나 `PlayScala` 플러그인을 사용할 때 처럼 plugins.sbt 파일에 간단하게 플러그인을 추가해 활성화 할 수 있다.

```scala
addSbtPlugin("com.typesafe.sbt" % "sbt-rjs" % "1.0.1")
```

다음과 같이 자산 파이프라인을 위한 플러그인을 추가해야한다.(파이프라인을 위한 플러그인이 하나라고 가정 했을 때이며, 필요에 따라 digest나 gzip처럼 다른것도 시퀀스에 추가할 수 있다)

```scala
pipelineStages := Seq(rjs)
```

RequireJS 옵티마이저를 위한 표준 빌드 프로파일은 기본적으로 제공되며 대부분의 프로젝트에서 사용하기에 충분할 것이다. 그러나 설정에 대해서 정보가 필요하다면 [플러그인 문서 ](https://github.com/sbt/sbt-rjs#sbt-rjs)를 참조하기 바란다.

RequireJS는 많은 동작을 수행하며 Trireme 아래 JVM에서 실행되는 동안 잘 동작한다. 성능 측면에서 보면 js-engine으로 Node.js를 사용하는 것이 가장 좋다. 편의를 위해  `SBT_OPTS`에 `sbt.jse.engineType` 속성을 설정할 수 있다. 예를 들어 유닉스에서는 다음과 같이 설정할 수 있다.


```bash
export SBT_OPTS="$SBT_OPTS -Dsbt.jse.engineType=Node"
```

설정에 대해서 정보가 필요하다면 [플러그인 문서 ](https://github.com/sbt/sbt-rjs#sbt-rjs)를 참조하기 바란다.