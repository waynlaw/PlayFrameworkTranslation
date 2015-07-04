<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# JSON 기본

근대의 웹 애플리케이션은 종종 JSON(JavaScript Object Notation)형식으로 데이터를 읽고 생성해야 한다. 플레이는 이 기능을 [JSON library](api/scala/index.html#play.api.libs.json.package)를 통해서 제공한다.

JSON은 가벼운 데이터 교환을 위한 형식이며, 아래와 같은 모양을 가졌다.

```json
{
  "name" : "Watership Down",
  "location" : {
    "lat" : 51.235685,
    "long" : -1.309197
  },
  "residents" : [ {
    "name" : "Fiver",
    "age" : 4,
    "role" : null
  }, {
    "name" : "Bigwig",
    "age" : 6,
    "role" : "Owsla"
  } ]
}
```

> JSON에 대해 더 많은 정보를 얻으려면, [json.org](http://json.org/)를 확인하자.

## 플레이 JSON 라이브러리
[`play.api.libs.json`](api/scala/index.html#play.api.libs.json.package) 패키지는 JSON 데이터를 표현하는 자료구조를 담고 있으며, 이런 자료구조와 다른 표현 방식들 간의 전환을 위한 도구들도 가지고 있다. 흥미있을 만한 형식들은 아래와 같다:

### [`JsValue`](api/scala/index.html#play.api.libs.json.JsValue)

이것은 모든 JSON 값들을 표현하기 위한 트레잇이다. JSON 라이브러리는 각각의 JSON 타입을 표현하기 위해서 `JsValue`를 상속받은 케이스 클래스를 가지고 있다.

- [`JsString`](api/scala/index.html#play.api.libs.json.JsString)
- [`JsNumber`](api/scala/index.html#play.api.libs.json.JsNumber)
- [`JsBoolean`](api/scala/index.html#play.api.libs.json.JsBoolean)
- [`JsObject`](api/scala/index.html#play.api.libs.json.JsObject)
- [`JsArray`](api/scala/index.html#play.api.libs.json.JsArray)
- [`JsNull`](api/scala/index.html#play.api.libs.json.JsNull)

다양한 JSValue 타입들을 사용하면, 어떠한 JSON구조의 표현도 만들 수 있다.

### [`Json`](api/scala/index.html#play.api.libs.json.Json$)
Json 오브젝트는 JsValue 구조로 부터 변환하기 위한 유틸리티들을 기본적으로 제공한다.

### [`JsPath`](api/scala/index.html#play.api.libs.json.JsPath)
JSValue 구조의 경로를 표현하는 것은, XML에서의 XPath와 유사하다. 이것은 JsValue구조를 순회하기 위해서 사용하며, 암시적인 변환을 위한 방법이다.

## JsValue로 변환하기

### Using string parsing

@[convert-from-string](code/ScalaJsonSpec.scala)

### 클래스 생성 사용하기

@[convert-from-classes](code/ScalaJsonSpec.scala)

`Json.obj`와 `Json.arr`는 간단하게 생성할 수 있다. 팩토리 함수가 대부분의 값들에 대해 암시적 변환을 제공해주기 때문에, 명시적으로 JsValue클래스로 감쌀 필요가 없다. (아래에 더 자세한 내용이 있다.)

@[convert-from-factory](code/ScalaJsonSpec.scala)

### Writes 변환자 사용하기
Scala에서 JsValue 변환은 `Json.toJson[T](T)(implicit writes: Writes[T])`의 도움 함수를 이용해 수행한다. 이런 기능은 `T`를 `JsValue`로 변환할 수 있는 [`Writes[T]`](api/scala/index.html#play.api.libs.json.Writes) 형식의 변환자에 의존한다.

플레이 JSON API는 `Int`, `Double`, `String`, 그리고 `Boolean`와 같은 대부분의 기본 타입에 대해서 암시적으로 `Writes`를 제공한다. 또한 `Writes[T]`가 존재하는 `T`형식의 컬렉션을 위한 `Writes`도 지원한다.

@[convert-from-simple](code/ScalaJsonSpec.scala)

고유한 모델들을 JsValues로 변환하기 위해서는 반드시 암시적인 `Writes`변환자를 선언하고, 범위 내에 그것을 제공해야 한다.

@[sample-model](code/ScalaJsonSpec.scala)

@[convert-from-model](code/ScalaJsonSpec.scala)

다른 방법으로는, 결합자 패턴을 사용해서 고유한 `Writes`를 정의하는 것이다.

> 주의: 결합자 패턴은 [[JSON Reads/Writes/Formats Combinators|ScalaJsonCombinators]]에서 자세히 다루고 있다.

@[convert-from-model-prefwrites](code/ScalaJsonSpec.scala)

## JsValue 구조 순회하기

`JsValue` 구조를 순회할 수 있고, 특정 값을 추출할 수 있다. 문법과 기능은 Scala XML 처리와 유사하다.

> 주의: 아래의 예제는 이전의 예제에서 만든 JsValue 구조에 적용한 것이다.

### 간단한 경로 `\`
`\` 연산자를 `JsValue`에 적용하면 필드 전달인자에 대응되는 JsObject를 반환한다.

@[traverse-simple-path](code/ScalaJsonSpec.scala)

### 재귀 경로 `\\`
`\\` 연산자를 적용하면 현재 오브젝트와 모든 자손들의 필드를 살펴보게 된다.

@[traverse-recursive-path](code/ScalaJsonSpec.scala)

### Index 탐색하기 (JsArrays에서)
`JsArray`에서 인덱스 번호를 연산자에 적용하여 값을 가져올 수 있다.

@[traverse-array-index](code/ScalaJsonSpec.scala)

## JsValue에서 변환하기

### 문자열 도구들 사용하기
최소화는 다음과 같이 할 수 있다.

@[convert-to-string](code/ScalaJsonSpec.scala)

```
{"name":"Watership Down","location":{"lat":51.235685,"long":-1.309197},"residents":[{"name":"Fiver","age":4,"role":null},{"name":"Bigwig","age":6,"role":"Owsla"}]}
```
가독성을 위해서는 아래와 같이 할 수 있다.

@[convert-to-string-pretty](code/ScalaJsonSpec.scala)

```
{
  "name" : "Watership Down",
  "location" : {
    "lat" : 51.235685,
    "long" : -1.309197
  },
  "residents" : [ {
    "name" : "Fiver",
    "age" : 4,
    "role" : null
  }, {
    "name" : "Bigwig",
    "age" : 6,
    "role" : "Owsla"
  } ]
}
```

### JsValue.as/asOpt 사용하기

`JsValue`를 다른 형식으로 변경하는 가장 간단한 방법은 `JsValue.as[T](implicit fjs: Reads[T]): T`를 사용하는 것이다. 이것은 `JsValue`를 `T` (`Writes[T]`의 반대이다.)로 변환하기 위해서 [`Reads[T]`](api/scala/index.html#play.api.libs.json.Reads) 형식에 대한 암시적 변환자를 필요로 한다. `Writes`와 같이, JSON API는 기본 형식에 대한 `Reads`를 제공한다.

@[convert-to-type-as](code/ScalaJsonSpec.scala)

만일 경로를 찾을 수 없거나, 변환이 불가능한 경우에, `as` 함수는 `JsResultException`를 발생시킬 것이다. 보다 안전한 함수는 `JsValue.asOpt[T](implicit fjs: Reads[T]): Option[T]` 이다.

@[convert-to-type-as-opt](code/ScalaJsonSpec.scala)

`asOpt` 함수가 더 안전하지만, 일부 오류 정보가 사라질 것이다.

### 검증 사용하기
일반적으로 사용되는 `JsValue`를 다른 형식으로 변환하는 방법은 그것의 `validate` 함수(`Reads` 형식의 전달인자를 받는)를 사용하는 것이다. 이것은 검증과 변환을 모두 수행하며 [`JsResult`](api/scala/index.html#play.api.libs.json.JsResult)형식을 반환한다. `JsResult`는 두 클래스로 구현되어 있다.

- [`JsSuccess`](api/scala/index.html#play.api.libs.json.JsSuccess) - 성공적인 검증과 변환을 나타내며, 결과를 포함하고 있다.
- [`JsError`](api/scala/index.html#play.api.libs.json.JsError) - 검증 또는 반환의 실패를 나타내며, 검증 오류들의 리스트를 포함하고 있다.

검증 결과를 다루기 위해 다향한 방법을 적용할 수 있다.

@[convert-to-type-validate](code/ScalaJsonSpec.scala)

### JsValue의 모델화

JsValue를 모델로 변환하기 위해서는, `T`가 모델의 타입인 곳에 암시적인 `Reads[T]`를 정의해야 한다.

> 주의: `Reads`를 구현하기 위해 사용하는 방법과 고유한 검증에 대해서는 [[JSON Reads/Writes/Formats Combinators|ScalaJsonCombinators]]에서 보다 자세하게 다루고 있다.

@[sample-model](code/ScalaJsonSpec.scala)

@[convert-to-model](code/ScalaJsonSpec.scala)
