<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 로그를 위한 API

어플리케이션 안에서 로그를 남기는 것은 모니터링, 디버깅, 에러 추적, 그리고 비지니스 인텔리전스에 유용하다. Play는 [`Logger`](api/scala/index.html#play.api.Logger$)를 통해 접근할 수 있고, [Logback](http://logback.qos.ch/) 을 로그 엔진으로 사용하는 API를 제공한다.

## 로그를 위한 아키텍쳐

로그를 위한 API는 효과적인 로그 방법을 구현하는 것을 돕는 구성 요소들을 사용한다.

#### Logger
어플리케이션에서는 로그 메시지 요청들을 보낼 수 있는 [`Logger`](api/scala/index.html#play.api.Logger) 인스턴스를 설정할 수 있다. 각각의 `Logger`는 로그메시지에 표시되는 이름을 가지며, 설정을 위해 이름을 사용할 수 있다.

Logger들은 자신의 이름을 기반으로한 상속 계층 구조를 따르며, "후손 로거"의 이름 중 앞의 일부를 자신의 이름으로 사용하는 경우에는 "조상 로거"라고 부른다. 예를 들면 이름이 "com.foo"인 로거는 "com.foo.bar.Baz."로거의 조상이다. 모든 로거들의 조상은 루트 로거이다. 로거의 상속관계를 이용하면 공통의 조상 로거의 설정을 통해 그 후손 로거들을 설정할 수 있다.

Play 어플리케이션은 기본 로거 이름으로 "application"을 사용하지만 원하는 이름으로 생성할 수 있다. Play 라이브러리들은 로거의 이름으로 "play"를 사용하며, 몇몇의 써드파티 라이브러리들은 고유한 이름을 가질수도 있다.

#### 로그 레벨들
로그 메시지들의 중요도들 분류하는데 로그 레벨을 사용한다. 로그 메시지를 사용할 때 특정한 중요도를 설정할 수 있으며, 이 중요도가 만들어진 로그 메시지에 나타난다.

아래에는 중요도가 높은 순서부터 낮은 순서로 사용 가능한 로그 레벨들이 나와있다.

- `OFF` - 로그를 표시 하지 않는다. 로그 메시지의 분류 기준은 아님.
- `ERROR` - 실행중 오류이거나, 예기치 못한 조건들.
- `WARN` - 오래된 API를 사용했거나, API를 잘못된 사용, 거의 에러이거나 예상하지 못했지만 반드시 잘못된 것이라고만은 할 수 없는 상황.
- `INFO` - 실행중에 발생하는 어플리케이션 시작이나 종료와 같은 관심있을 만한 이벤트들.
- `DEBUG` - 시스템 전반의 흐름에 대한 상세한 정보들.
- `TRACE` - 가장 자세한 정보들.

게다가 메시지들을 분류할 때의 로그레벨들은 로그를 남길때, 얼마나 상세하게 남길지를 설정하는데 사용할 수도 있다. 예를 들면, 로거의 레벨을 `INFO`로 설정한 경우에 로거는 `INFO`보다 같거나 높은 모든 요청(`INFO`, `WARN`, `ERROR`)들을 기록할 것이다. 하지만 그보다 중요도가 낮은 로그들(`DEBUG`, `TRACE`)은 무시할 것이다. 만일 `OFF`로 설정하게 되면 모든 로그 요청들이 무시될 것이다.

#### Appenders 
로그 API를 통해 하나나 그 이상의 "Appenders"라고 불리는 출력 장소를 지정할 수 있다. Appender들은 설정에 정의되어 있으며, 콘솔이나 파일, 데이터베이스, 다른 여러 출력장소들을 위한 옵션들이 존재한다.

로거들과 연결되있는 Appender들을 사용해서 다른 출력할 방법을 지정하거나 필요한 로그 메시지들만 선별할 수 있다. 예를 들면, 하나의 Appender를 사용해서 분석에 유용한 데이터만을 모을 수 있고, 동시에 다른 Appender에서는 관리 팀에서 사용할 에러 데이터만을 수집할 수 있다.

> 메모: 구조에 대한 더 많은 정보를 얻으려면, [Logback documentation](http://logback.qos.ch/manual/architecture.html)을 보십시오.

## 로거들을 사용하기
우선 `Logger` 클래스와 짝 오브젝트를 포함해야 한다.:

@[logging-import](code/ScalaLoggingSpec.scala)

#### 기본 로거
`Logger` 오브젝트는 기본 로거이며 "application"을 이름으로 사용한다. 로그문장들을 기록하기 위해서 이것을 사용할 수 있다.:

@[logging-default-logger](code/ScalaLoggingSpec.scala)

Play의 기본 로그 설정을 사용하면, 아래와 유사한 결과가 콘솔에 표시될 것이다.:

```
[debug] application - Attempting risky calculation.
[error] application - Exception with riskyCalculation
java.lang.ArithmeticException: / by zero
    at controllers.Application$.controllers$Application$$riskyCalculation(Application.scala:32) ~[classes/:na]
    at controllers.Application$$anonfun$test$1.apply(Application.scala:18) [classes/:na]
    at controllers.Application$$anonfun$test$1.apply(Application.scala:12) [classes/:na]
    at play.api.mvc.ActionBuilder$$anonfun$apply$17.apply(Action.scala:390) [play_2.10-2.3-M1.jar:2.3-M1]
    at play.api.mvc.ActionBuilder$$anonfun$apply$17.apply(Action.scala:390) [play_2.10-2.3-M1.jar:2.3-M1]
```

이 메시지들은 로그 레벨, 로그 이름, 내용을 가지고 있으며, Throwable이 로그 요청에 사용된 경우에는 호출 스텍까지도 포함하고 있다.

#### 고유한 로거 만들기
기본으로 제공되는 로거를 모든 곳에 쓰는 것도 매력적이긴 하지만, 보통 잘못된 디자인 방법이다. 별개의 이름을 가진 로거를 만들어 사용하면 더 유연한 설정을 할 수 있고, 로그 결과물을 선별적으로 볼 수 있도록 도와준다. 또한 로그메시지의 정확한 출처를 쉽게 찾을 수 있다.

`Logger.apply` 펙토리 메소드에 이름을 전달하여 고유한 로거를 만들 수 있다.:

@[logging-create-logger-name](code/ScalaLoggingSpec.scala)

클래스 이름을 사용하여 클래스마다 개별적인 로거를 사용하는 것이, 일반적인 어플리케이션의 이벤트 로그를 남기는 방법이다. 로그를 남기는 API는 클래스를 전달 인자로 받는 펙토리 메소드를 지원한다.:

@[logging-create-logger-class](code/ScalaLoggingSpec.scala)

#### 로그를 남기는 패턴들
효과적으로 로그를 남기면 동일한 도구를 이용해서 여러가지 목표를 이룰 수 있다.:

@[logging-pattern-mix](code/ScalaLoggingSpec.scala)

이 예제는 "acess"라는 이름의 로거에 로그를 요청하는 `AccessLoggingAction`을 정의하기 위해서 [[액션 합성|ScalaActionsComposition]] 사용하는 예제이다. `Application` 컨트롤러는 이 액션을 사용하며, 어플리케이션 이벤트를 기록하기위해 자기 자신의 로거 또한 사용한다. 예를 들면 접근 로그나 어플리케이션 로그와 같은 로그들을 설정에서는 다른 Appender로 연결할 수 있다.

위와 같은 디자인은 특정한 액션에 대해서 로그를 전달하기를 원할때에만 잘 작동한다. 모든 요청을 기록하기 위해서는 [[필터|ScalaHttpFilters]] 를 사용하는 것이 더 효과적이다.:

@[logging-pattern-filter](code/ScalaLoggingSpec.scala)

필터 버전에서는 `Future[Result]`가 완료될 때, 응답 상태를 로그에 기록하도록 하였다.

## 설정
[[로그 설정|SettingsLogger]]에서 설정에 대한 정보를 얻을 수 있다. 
