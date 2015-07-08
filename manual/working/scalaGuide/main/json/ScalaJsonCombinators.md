<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# JSON 읽기/쓰기/형식 결합자들

[[JSON 기본|ScalaJson]] 에서는 [`JsValue`](api/scala/index.html#play.api.libs.json.JsValue) 구조체와 다른 데이터 형식들간의 변환에 사용되는 [`Reads`](api/scala/index.html#play.api.libs.json.Reads) 와 [`Writes`](api/scala/index.html#play.api.libs.json.Writes) 변환자들에 대해 소개했다. 이 페이지는 이 변환자들을 어떻게 만들고 어떻게 변환의 검증에 사용하는지 보다 자세하게 알아볼 것이다.

이 페이지의 예제는 `JsValue` 구조체와, 이에 대응되는 모델을 사용할 것이다.

@[sample-json](code/ScalaJsonCombinatorsSpec.scala)

@[sample-model](code/ScalaJsonCombinatorsSpec.scala)

## JsPath

`Reads`/`Writes`를 생성하기 위한 [`JsPath`](api/scala/index.html#play.api.libs.json.JsPath) 는 핵심 구성 요소이다. `JsPath`는 `JsValue` 구조체에서 데이터의 위치를 나타낸다. `JsPath` 오브젝트 (루트 경로)에서는 `JsValue`를 순회하는 것과 유사한 문법을 사용하여 `JsPath` 자식 객체를 정의할 수 있다.

@[jspath-define](code/ScalaJsonCombinatorsSpec.scala)

[`play.api.libs.json`](api/scala/index.html#play.api.libs.json.package) 는 `JsPath`: `__` (이중 밑줄)를 위한 별칭을 정의하고 있다. 이것을 더 선호한다면 사용해도 된다.

@[jspath-define-alias](code/ScalaJsonCombinatorsSpec.scala)

## Reads
[`Reads`](api/scala/index.html#play.api.libs.json.Reads) 변환자는 `JsValue` 에서 다른 형식으로 변환할 때 사용한다. 더 복잡한 `Reads`를 만들기 위해서 이를 결합하거나 `Reads` 내부에 감싸진 `Reads`를 사용할 수 있다.

`Reads`를 사용하기 위해 아래 것들을 포함해야 한다.

@[reads-imports](code/ScalaJsonCombinatorsSpec.scala)

### Path Reads
`JsPath`는 다른 `Reads`를 특정 경로의 `JsValue`에 적용해주는 특별한 `Reads`를 생성하기 위한 함수를 가지고 있다.

- `JsPath.read[T](implicit r: Reads[T]): Reads[T]` - 암시적인 인자 `r`을 이 경로의 `JsValue`에 적용해주는 `Reads[T]`를 생성한다.
- `JsPath.readNullable[T](implicit r: Reads[T]): Reads[Option[T]]readNullable`
- 없거나 null 값을 가질 수 있는 경로를 위해 사용한다.

> 주의: JSON 라이브러리는 String, Int, Double 등의 기본 형식을 위한 암시적인 `Reads`를 제공한다.

각각의 경로 `Reads`를 정의하는 것은 아래와 같다.

@[reads-simple](code/ScalaJsonCombinatorsSpec.scala)

### 복잡한 Reads
복잡한 모델의 변환을 위한 더 복잡한 `Reads`를 만들기 위해서, 각각의 경로의 `Reads`를 결합할 수 있다.

이해를 돕기 위해서 결합 기능을 둘로 분할할 것이다. 첫번째의 `Reads` 오브젝트는 `and` 결합자를 사용한다.

@[reads-complex-builder](code/ScalaJsonCombinatorsSpec.scala)

이는 `FunctionalBuilder[Reads]#CanBuild2[Double, Double]` 형식을 미룰 것이다. 이는 중간의 오브젝트로, 이에 대해 너무 걱정할 필요는 없다. 단지 복잡한 `Reads`를 만드는데 사용한다는 것만 알면 된다.

두번째로 각각의 값들을 모델로 변환해주는 함수와 함께 `CanBuildX`의 `apply` 함수를 불러야 한다. 이것은 복잡한 `Reads`를 반환할 것이다. 만일 맞춰진 생성자 형식을 가진 케이스 클래스를 가지고 있다면, 단지 `apply`함수만 호출하면 된다.

@[reads-complex-buildertoreads](code/ScalaJsonCombinatorsSpec.scala)

아래는 한 문장으로 동일한 기능을 수행하는 예제이다.

@[reads-complex-statement](code/ScalaJsonCombinatorsSpec.scala)

### Reads로 검증하기

`JsValue.validate` 함수는 [[JSON 기본|ScalaJson]]에서 `JsValue`를 다른 형식으로 변환하거나 검증을 하는데 선호되는 방법으로 소개되었다. 여기에 기본적인 예제가 있다.

@[reads-validation-simple](code/ScalaJsonCombinatorsSpec.scala)

`Reads`에 대한 기본 검증은 형식 변환 오류와 같은 것을 검증할 수 있는 정도만 최소로 있다. `Reads` 검증 도우미를 이용하여 몇가지 검증하는 규칙을 정의할 수 있다. 일반적으로 사용되는 예는 아래와 같다.

- `Reads.email` - 문자열이 메일 형식인지 검증한다.
- `Reads.minLength(nb)` - 문자열의 최소 길이를 검증한다.
- `Reads.min` - 숫자 값의 최소값을 검증한다.
- `Reads.max` - 숫자 값의 최대값을 검증한다.
- `Reads[A] keepAnd Reads[B] => Reads[A]` - `Reads[A]`와 `Reads[B]`를 시도하지만, `Reads[A]`의 결과만을 갖는다. (Scala 파서 조합기의 `keepAnd == <~`)
- `Reads[A] andKeep Reads[B] => Reads[B]` - `Reads[A]`와 `Reads[B]`를 시도하지만, `Reads[B]`의 결과만을 갖는다. (Scala 파서 조합기의 `keepAnd == ~>`)
- `Reads[A] or Reads[B] => Reads` - 논리적 OR를 수행하고 마지막으로 확인한 결과를 갖는다.

검증을 추가하기 위해서는 도우미를 `JsPath.read` 함수의 인자로 적용해야 한다.

@[reads-validation-custom](code/ScalaJsonCombinatorsSpec.scala)

### 모든 것을 함께 사용하기

복잡한 `Reads`와 고유의 검증을 사용하게되면, 예제의 모델을 위한 효과적인 `Reads`들을 정의하고 적용할 수 있다.

@[reads-model](code/ScalaJsonCombinatorsSpec.scala)

복잡한 `Reads`는 감싸져있다는 것에 주의해야 한다. 이 경우의 `placeReads`는 이전에 정의된 구조체의 특정 경로의 암시적 `locationReads`과 `residentReads`를 사용한다.

## Writes
[`Writes`](api/scala/index.html#play.api.libs.json.Writes) 변환자들은 몇가지 형식을 `JsValue`로 변환할 때 사용한다.

`JsPath`와 `Reads`와 유사한 조합자들을 사용하여, 복잡한 `Writes`를 만들 수 있다. 예제의 모델을 위한 `Writes`는 아래와 같다.

@[writes-model](code/ScalaJsonCombinatorsSpec.scala)

복잡한 `Writes`와 `Reads`는 몇가지 차이점이 있다.

- `Writes`의 개별적인 경로는 `JsPath.write` 함수를 이용하여 생성한다.
- 구조를 간단하게 하기 위해 `JsValue`로의 변환에는 검증이 없다. 그래서 검증 도우미가 필요없을 것이다.
- 중간의 `FunctionalBuilder#CanBuildX` (`and`결합자에서 생성된)는 복잡한 형식 `T`를 각각의 경로와 `Writes` 튜플로 변환하는 함수를 받는다. 비록 이런 것이 `Reads`의 경우와 동일하더라도, 케이스 클래스의 `unapply` 함수는 속성들의 튜플의 `Option`을 반환할 것이다. 그리고 튜플을 추출하기 위해서 `unlift`과 함께 사용해야 한다.

## 재귀 형식
예제의 모델의 특별한 점은, 재귀 형식의 `Reads`와 `Writes`에 대해 어떻게 다루어야 하는지를 보여주지 않았다는 것이다. 이런 경우를 위해서 `JsPath`는 이름으로 값을 전달받는 `lazyRead`와 `lazyWrite` 함수를 제공한다.

@[reads-writes-recursive](code/ScalaJsonCombinatorsSpec.scala)

## Format
[`Format[T]`](api/scala/index.html#play.api.libs.json.Format)은 `Reads`과 `Writes` 트레잇의 결합이다. 그 구성물의 암시적 변환에 사용된다.

### Reads와 Writes에서 Format 만들기
동일 형식의 `Reads` and `Writes`를 이용해서 `Format`를 정의할 수 있다.

@[format-components](code/ScalaJsonCombinatorsSpec.scala)

### 결합자를 이용해서 Format 생성하기
`Reads`와 `Writes`가 대칭적인 경우는 (실제 애플리케이션이 아닌 경우일 수 있다.), `Format`를 결합자에서 바로 정의할 수 있다.

@[format-combinators](code/ScalaJsonCombinatorsSpec.scala)
