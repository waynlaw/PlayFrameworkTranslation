<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# Guice를 사용하여 테스트 하기

만일 [[의존성 주입|ScalaDependencyInjection]]을 위해 Guice를 사용한다면, 테스트를 위해 어떻게 컴포넌트와 애플리케이션을 생성할지에 대한 설정을 바로 할 수 있다. 이 기능은 존재하는 바인딩을 덮어쓰거나 바인딩을 추가하는것을 포함한다.

## GuiceApplicationBuilder

[GuiceApplicationBuilder](api/scala/index.html#play.api.inject.guice.GuiceApplicationBuilder)는 의존성 주입을 설정하고 [애플리케이션](api/scala/index.html#play.api.Application)을 생성하기 위한 빌더 API를 제공다.

### 환경

[환경](api/scala/index.html#play.api.Environment), 또는 루트 경로, 모드, 애플리케이션의 클래스 로더와 같은 환경의 일부분들이 명시될 수 있다. 설정된 환경은 애플리케이션 설정을 불러들이는데 사용된다. 이것은 모듈들을 불러들이거나, 플레이 모듈에서 바인딩을 얻거나, 다른 컴포넌트에 주입할 수 있다.

@[builder-imports](code/tests/guice/ScalaGuiceApplicationBuilderSpec.scala)
@[set-environment](code/tests/guice/ScalaGuiceApplicationBuilderSpec.scala)
@[set-environment-values](code/tests/guice/ScalaGuiceApplicationBuilderSpec.scala)

### 설정

추가적인 설정을 할 수 있다. 애플리케이션을 위해 설정이 자동으로 로딩될 때, 이 설정은 언제나 적용될 수 있다. 만일 이미 존재하는 키들이 있는 경우에는 새로운 설정이 더 선호된다.

@[add-configuration](code/tests/guice/ScalaGuiceApplicationBuilderSpec.scala)

애플리케이션 환경에서 설정을 자동으로 읽어들이는 것은 재정의될 수 있다. 이런 경우에는 애플리케이션 설정이 완전히 대체된다. 예를 들면:

@[override-configuration](code/tests/guice/ScalaGuiceApplicationBuilderSpec.scala)

### 바인딩들과 모듈들

의존성 주입을 위한 바인딩들은 모든 설정이 가능하다. 빌더 함수들은 [[Play Modules and Bindings|ScalaDependencyInjection]]와 Guice 모듈들을 지원한다.

#### 추가적인 바인딩들

플레이 모듈들이나, 플레이 바인딩들 또는 Guice 모듈들을 통한 추가적인 바인딩들을 넣을 수 있다.:

@[bind-imports](code/tests/guice/ScalaGuiceApplicationBuilderSpec.scala)
@[add-bindings](code/tests/guice/ScalaGuiceApplicationBuilderSpec.scala)

#### 바인딩들 덮어쓰기

플레이 바인딩들이나 바인딩을 제공하는 모듈들을 사용하여 바인딩들을 덮어쓸 수 있다. 예:

@[override-bindings](code/tests/guice/ScalaGuiceApplicationBuilderSpec.scala)

#### 모듈들 비활성화 하기

읽어진 어떤 모듈이라도 클래스 이름을 통해서 비활성화 할 수 있다:

@[disable-modules](code/tests/guice/ScalaGuiceApplicationBuilderSpec.scala)

#### 읽어진 모듈들

모듈들은  `play.modules.enabled`설정의 클래스패스에서 자동적으로 읽어진다. 모듈들을 읽어들이는 기본 기능은 덮어써질 수 있다. 예:

@[load-modules](code/tests/guice/ScalaGuiceApplicationBuilderSpec.scala)


## GuiceInjectorBuilder

[GuiceInjectorBuilder](api/scala/index.html#play.api.inject.guice.GuiceInjectorBuilder)는 Guice 의존성 주입을 보다 일반적으로 설정할 수 있는 빌더 API를 제공한다. 이 빌더는 `GuiceApplicationBuilder`처럼 설정이나 모듈을 환경에서 자동으로 로드하지는 않는다. 하지만 설정과 바인딩들을 추가하기 위해서 완전이 깨끗한 상태를 제공해 준다. 두 빌더들을 위한 일반적인 인터페이스는 [GuiceBuilder](api/scala/index.html#play.api.inject.guice.GuiceBuilder)에서 찾을 수 있다. 플레이 [Injector](api/scala/index.html#play.api.inject.Injector) 도 만들어졌다. 여기에 주입 빌더를 사용해서 컴포넌트를 초기화 하는 예제가 있다.:

@[injector-imports](code/tests/guice/ScalaGuiceApplicationBuilderSpec.scala)
@[bind-imports](code/tests/guice/ScalaGuiceApplicationBuilderSpec.scala)
@[injector-builder](code/tests/guice/ScalaGuiceApplicationBuilderSpec.scala)


## 함수형 테스트 내에서 바인딩들을 덮어쓰기

테스트를 위한 가짜컴포넌트들로 기존의 컴포넌트를 변경한는 완전한 예제가 여기에 있다. 기본적인 구현을 컴포넌와 테스트를 위한 가짜 객체를 만드는 것에서 시작해 보자:

@[component](code/tests/guice/Component.scala)

모듈을 사용하면 이 컴포넌트는 자동으로 로드된다.:

@[component-module](code/tests/guice/Component.scala)

그리고 그 컴포넌트는 컨트롤러에 의해 사용된다.:

@[controller](code/tests/guice/controllers/Application.scala)

함수형 테스트를 위한 `애플리케이션을` 만들기 위해서는 컴포넌트의 바인딩을 간단하게 덮어쓸 수 있다.:

@[builder-imports](code/tests/guice/ScalaGuiceApplicationBuilderSpec.scala)
@[bind-imports](code/tests/guice/ScalaGuiceApplicationBuilderSpec.scala)
@[override-bindings](code/tests/guice/ScalaGuiceApplicationBuilderSpec.scala)

만들어진 애플리케이션은 [[Specs2|ScalaFunctionalTestingWithSpecs2]]와 [[ScalaTest|ScalaFunctionalTestingWithScalaTest]]를 위한 함수형 테스트를 도와주는 용도로 사용할 수 있다.
