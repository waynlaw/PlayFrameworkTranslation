<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 메시지들과 국제화

## 애플리케이션이 지원하는 언어 지정

`fr`나 `en-US`와 같은 올바른 언어 코드는 **ISO 639-2 language code**에 명시되어 있으며, 추가적으로 **ISO 3166-1 alpha-2 country code**를 덧붙일 수 있다.

어플리케이션이 지원하는 언어들을 명시하려면 `conf/application.conf`파일 에서 시작하면 된다.:

```
play.i18n.langs = [ "en", "en-US", "fr" ]
```

## 메시지들 외부에 두기

`conf/messages.xxx` 파일들에서 메시지들을 외부에 둘 수 있다.

기본 `conf/messages`파일은 모든 언어를 수용한다. 추가적으로 `conf/messages.fr`나 `conf/messages.en-US`와 같이 특정 언어에 대한 메시지 파일을 명시할 수 있다.

그리고 메시지들을 `play.api.i18n.Messages` 오브젝트를 통해서 가져올 수 있다.:

```scala
val title = Messages("home.title")
```

모든 국제화 API 호출은 현재 구역에서 사용되는 `play.api.i18n.Messages`를 암시적으로 전달받는다. 이 암시적은 값은 사용할 언어와 (본질적인) 국제화된 메시지들을 다 포함하고 있다.

암시적인 값을 받을 수 있는 가장 간단한 방법은 `I18nSupport` 트레잇을 사용하는 것이다. 예를 들면 그것은 컨트롤러들에서 사용할 수도 있다.:

@[i18n-support](code/ScalaI18N.scala)

`I18nSupport` 트레잇은 암시적인 범위 내에 `Lang`나 `RequestHeader`가 있는 동안, 암시적인 `Messages` 값을 제공해 준다.

> **주의:** 만일 `RequestHeader`를 암시적인 범위내에 가지고 있다면, `Accept-Language`헤더에서 추출된 선호하는 언어를 사용하고, `MessagesApi`가 지원하는 언어중의 하나를 찾을 것이다. `Messages`의 암시적인 파라메터를 템플릿에 다음과 같이 추가해야 할 것이다: `@()(implicit messages: Messages)`.

> **주의:** 또한 플레이는 외부에서 `MessagesApi`값(`DefaultMessagesApi`의 구현을 사용하는)을 어떻게 추가해야될지를 "알고"있다. 그렇기 때문에 `@javax.inject.Inject` 어노테이션을 컨트롤러에 넣기만 하면, 플레이가 구성요소들을 연결해 줄것이다.

## 메시지 형식

메시지들은 `java.text.MessageFormat` 라이브러리를 통해서 형식을 맞춘다. 예를 들면 메시지를 아래와 같이 정의했다고 가정하자.:

```
files.summary=The disk {1} contains {0} file(s).
```

파라메터를 아래와 같이 정의할 수 있다.:

```scala
Messages("files.summary", d.files.length, d.name)
```

## 아포스트로피에 주의하기

메시지들은 `java.text.MessageFormat`를 사용하기 때문에, 따옴표는 파라메터 치환을 빠져나가는 메타 문자로 사용되고 있는지를 주의해야 한다.

예를 들어, 아래와 같은 메시지를 정의해보자.:

@[apostrophe-messages](code/scalaguide/i18n/messages)
@[parameter-escaping](code/scalaguide/i18n/messages)

아래와 같은 결과가 나올 것이다.:

@[apostrophe-messages](code/ScalaI18N.scala)
@[parameter-escaping](code/ScalaI18N.scala)

## HTTP 요청에서 지원하는 언어 가져오기

특정한 HTTP 요청에서 지원하는 언어를 가져올 수 있다.:

```scala
def index = Action { request =>
  Ok("Languages: " + request.acceptLanguages.map(_.code).mkString(", "))
}
```
