<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# Guice를 이용하여 테스트 하기

[[의존성 주입|JavaDependencyInjection]]을 위해 Guice를 이용한다면, 테스트를 위해 어떻게 컴포넌트와 애플리케이션이 생성될 것인지 직접 설정을 해야 할 것이다. 이 작업은 추가 바인딩 혹은 기존 바인딩을 오버라이딩 하는 작업등이 포함한다. 

## GuiceApplicationBuilder

[GuiceApplicationBuilder](api/java/play/inject/guice/GuiceApplicationBuilder.html)는 의존성 주입과,  [Application](api/java/play/Application.html) 생성을 설정하기 위해 builder API를 제공하고 있다.

### Imports

Guice를 이용해 애플리케이션을 구성하기 위해 main에 임포트 해야 할 것은 아래와 같다:

@[builder-imports](code/tests/guice/JavaGuiceApplicationBuilderTest.java)

### Environment

[Environment](api/java/play/Environment.html), 혹은 루트 경로와 같은 환경 요소들, 혹은 애플리케이션 클래스로더는 따로 명시하는 것이 가능하다. 설정한 환경은 애플리케이션 설정을 불러오는데 사용된다. 또한 모듈을 불러올 때나 사용되거나, 플레이 모듈 바인딩 과정을 거쳐 다른 컴포넌트에 주입가능한 상태가 될 것이다.

@[set-environment](code/tests/guice/JavaGuiceApplicationBuilderTest.java)
@[set-environment-values](code/tests/guice/JavaGuiceApplicationBuilderTest.java)

### 설정

설정은 추가가 가능하다. 설정은 언제나 애플리케이션에서 자동으로 로드된 설정에 추가되는 형태이다. 기존에 사용되는 키값이 있는 경우, 새로운 설정이 우선 순위를 가진다.

@[add-configuration](code/tests/guice/JavaGuiceApplicationBuilderTest.java)

애플리케이션 환경에서 자동으로 불러온 설정은 오버라이딩이 가능하다. 이 과정을 통해 애플리케이션 설정을 완전히 바꿔치기 하게 된다. 예를 들어:

@[override-configuration](code/tests/guice/JavaGuiceApplicationBuilderTest.java)

### 바인딩과 모듈

의존성 주입을 위한 바인딩 과정은 설정이 가능하다. 빌더 메소드를 통해 [[플레이 바인딩과 모듈|JavaDependencyInjection]]과 Guice 모듈 설정을 지원하고 있다. 

#### 추가적인 바인딩

플레이 모듈, 플레이 바인딩, 혹은 Guice을 통해 추가적인 바인딩이 가능하다:

@[bind-imports](code/tests/guice/JavaGuiceApplicationBuilderTest.java)
@[add-bindings](code/tests/guice/JavaGuiceApplicationBuilderTest.java)

#### 오버라이딩 바인딩

플레이 바인딩이나, 바인딩을 제공하는 모듈을 사용하여 바인딩을 오버라이드 할 수 있다. 예를 들어: 

@[override-bindings](code/tests/guice/JavaGuiceApplicationBuilderTest.java)

#### 모듈 무효화 시키기

클래스 이름을 이용하면 어떤 불러온 모듈이든지 무효화가 가능하다:

@[disable-modules](code/tests/guice/JavaGuiceApplicationBuilderTest.java)

#### 모듈 불러오기

`play.modules.enabled` 설정을 기반하여, 클래스패스에서 자동으로 모듈을 불러오게 된다. 기본 모듈 불러오기 설정은 오버라이딩이 가능하다. 예를 들어:

@[guiceable-imports](code/tests/guice/JavaGuiceApplicationBuilderTest.java)
@[load-modules](code/tests/guice/JavaGuiceApplicationBuilderTest.java)


## GuiceInjectorBuilder

[GuiceInjectorBuilder](api/java/play/inject/guice/GuiceInjectorBuilder.html)은 Guice 의존성 주입을 좀 더 보편적으로 설정할 수 있게하기 위해 빌더 API를 제공한다. 이 빌더는 `GuiceApplicationBuilder`와 달리 환경에서 자동으로 모듈을 불러오거나, 설정을 가져오지 않는다. 대신 추가 설정이나 바인딩을 위해 완벽히 깨끗한 상태를 제공한다. 위 두가지 빌더를 위한 공통 인터페이스는 [GuiceBuilder](api/java/play/inject/guice/GuiceBuilder.html)에서 찾아볼 수 있다. 플레이 [Injector](api/java/play/inject/Injector.html)는 만들어져 있다. 아래는 인젝터 빌더를 사용하여 컴포넌트를 기술한 예시이다:

@[injector-imports](code/tests/guice/JavaGuiceApplicationBuilderTest.java)
@[bind-imports](code/tests/guice/JavaGuiceApplicationBuilderTest.java)
@[injector-builder](code/tests/guice/JavaGuiceApplicationBuilderTest.java)


## 함수형 테스트에서 바인딩 오버라이딩하기

아래는 테스트를 위해 목 컴포넌트로 대체 된 컴포넌트를 보여주고 있다. 컴포넌트부터 시작해보자. 컴포넌트는 기본 구현체를 가지고 있고, 테스트를 위해 목 구현을 추가적으로 가지고 있다:

@[component](code/tests/guice/Component.java)

@[default-component](code/tests/guice/DefaultComponent.java)

@[mock-component](code/tests/guice/MockComponent.java)

컴포넌트는 모듈을 사용할 때 자동으로 불러오게 된다:

@[component-module](code/tests/guice/ComponentModule.java)

그리고 컴포넌트는 컨트롤러에서 사용된다:

@[controller](code/tests/guice/controllers/Application.java)

[[함수형 테스트|JavaFunctionalTest]]에서 `Application`를 빌드하여 사용하기 위해 간단히 해당 컴포넌트에 대한 바인딩을 오버라이딩 할 수 있다:

@[builder-imports](code/tests/guice/JavaGuiceApplicationBuilderTest.java)
@[bind-imports](code/tests/guice/JavaGuiceApplicationBuilderTest.java)
@[override-bindings](code/tests/guice/JavaGuiceApplicationBuilderTest.java)

`running` 과 `WithApplication`같은 테스트 헬퍼를 이용하여 이 애플리케이션을 사용할 수 있다.