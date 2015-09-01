<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# Action 결합

이 항목에서는 일반적인 Action을 함수형으로 정의하는 몇가지 방법에 대해 알아보겠다.

## 맞춤 Action 생성기들

우리는 요청 인자나, 요청 인자 없이 또는 내용 분석기와 함께 Action을 선언하는 다양한 방법들을 [[이전 장|ScalaActions]]에서 보았다. 실제로는 그 이상의 것들이 있으며, [[비동기 프로그래밍|ScalaAsync]]에서 알아볼 것이다.

이러한 Action을 생성하는 방법들을 실제로는 [`ActionBuilder`](api/scala/index.html#play.api.mvc.ActionBuilder) 트레잇에 정의되어 있다. 그리고 우리가 우리만의 Action을 정의하기 위해서 사용하는 [`Action`](api/scala/index.html#play.api.mvc.Action$) 오브젝트는 이 트레잇을 실체화한 것이다. 재사용 가능한 Action 스택을 선언할 수 있는, 고유한 `ActionBuilder`를 만들어서 Action 생성에 사용할 수 있다.

Action이 호출될 때마다 로그를 기록하고자 할 때, 로그를 꾸미는 간단한 예를 시작해 보자.

`invokeBlock`함수에 이 기능을 구현하는 첫번째 방법은, `ActionBuilder`가 Action을 생성할 때마다 호출하는 것이다.

@[basic-logging](code/ScalaActionsComposition.scala)

이제 `Action`을 사용하는 것과 동일하게 사용할 수 있다.

@[basic-logging-index](code/ScalaActionsComposition.scala)
 
 `ActionBuilder`가 Action을 만드는 다른 모든 방법을 제공하는 방식으로, 임의의 몸체 분석기를 선언하는 것과 같은 것도 가능하다.

@[basic-logging-parse](code/ScalaActionsComposition.scala)


### Action들 조합하기

대부분의 애플리케이션에서, 여러개의 Action 생성자를 만들기를 원하며, 그 중의 일부는 서로 다른 형식의 인증을 사용하거나, 서로 다른 제네릭 형식을 지원하기도 한다. 이런 경우, 각 형식의 Action 생성자에서 로그를 기록하는 Action의 코드를 변경하기를 원하지는 않으며, 재사용 가능한 형식으로 정의하기를 원한다.

기존 Action을 감싸서, 재 사용가능한 Action 코드를 만들 수 있다.

@[actions-class-wrapping](code/ScalaActionsComposition.scala)

Action들을 고유의 클래스를 정의하지 않고 생성하기 위해서, `Action`을 Action 생성자로 사용할 수 있다.

@[actions-def-wrapping](code/ScalaActionsComposition.scala)

Action은 `composeAction` 함수를 이용하여 Action생성자에 섞어 넣을 수 있다.

@[actions-wrapping-builder](code/ScalaActionsComposition.scala)

이제 이전과 동일한 방법으로 생성자를 사용할 수 있다.

@[actions-wrapping-index](code/ScalaActionsComposition.scala)

또한 Action 생성자를 사용하지 않고 Action을 감싸 넣을 수 있다.

@[actions-wrapping-direct](code/ScalaActionsComposition.scala)

### 더 복잡한 Action들

지금까지는 요청에 대해 전혀 신경쓰지 않고 Action만을 사용했었다. 물런, 전달된 요청 오브젝트를 읽고 수정할 수도 있다.

@[modify-request](code/ScalaActionsComposition.scala)

> **주의:** 플레이는 이미 X-Forwarded-For 헤더에 대한 지원을 내장하고 있다.

요청을 막을 수도 있다.

@[block-request](code/ScalaActionsComposition.scala)

그리고 마지막으로 반환된 결과를 수정할 수도 있다.

@[modify-result](code/ScalaActionsComposition.scala)

## 다른 요청 형식들

Action을 합성하는 것은 HTTP 요청과 응답을 처리하는 단계에서 추가적인 작업을 수행할 수 있게 해준다.
이를 통해 종종 문맥을 추가하거나 요청을 자체적으로 검증하는 등의 데이터를 처리를 위한 파이프 라인을 만들고 싶어할 수 있다. `ActionFunction`는 요청에 함수가 될 수 있고 입력 형식과 다음 계층으로 전달할 출력 형식 모두에 파라메터화 될 수 있다. 각각의 Action 함수는 인증이나 객체를 위한 데이터베이스 접근, 권한 확인, 그 이외의 Action들을 조합하고 재사용 하기를 원하는 여러가지 모듈화된 처리등을 나타낸다.

`ActionFunction`을 몇가지 처리에 유용하도록 구현한, 미리 정의된 트레잇이 있다.

* [`ActionTransformer`](api/scala/index.html#play.api.mvc.ActionTransformer) 예를 들면 추가적인 정보를 더하는 것 처럼, 요청을 변경할 수 있다.
* [`ActionFilter`](api/scala/index.html#play.api.mvc.ActionFilter) 예를 들어 요청을 변경하지 않고 에러를 만들기 위해 요청을 선택하는 것 처럼, 선택적으로 요청을 가로챌 수 있다.
* [`ActionRefiner`](api/scala/index.html#play.api.mvc.ActionRefiner) 위의 두가지 기능을 모두 수행할 수 있다.
* [`ActionBuilder`](api/scala/index.html#play.api.mvc.ActionBuilder) 는 특별한 함수로, `Request`를 입력으로 받아 Action을 만들 수 있다.

또한 `invokeBlock` 함수를 정의하여, 고유한 임의의 `ActionFunction`을 정의할 수 있다. 종종 이 방법은 (`WrappedRequest`를 사용하는) `Request`의 입/출력 형식을 만들기 위해 편하게 사용될 수 있다. 하지만 이것이 반드시 필요한 것은 아니다.

### 인증

Action 함수의 일반적인 사용 용도 중 한가지는 인증이다. 쉽게 원래의 요청을 보낸 사용자인지를 결정하고, 새 `UserRequest`를 추가하는 고유한 인증 Action 변환자를 구현할 수 있다. 주의할 것은 그 또한 간단한 `Request`를 입력으로 받기 때문에, 일종의 `ActionBuilder`이라는 점이다.

@[authenticated-action-builder](code/ScalaActionsComposition.scala)

Play는 또한 내장된 인증 Action 생성자를 제공한다. 이에 대한 정보는 [여기](api/scala/index.html#play.api.mvc.Security$$AuthenticatedBuilder$)에서 더 찾을 수 있다.

> **주의:** 내장된 인증 Action 생성자는 단지 간단한 인증이 필요한 경우에 대해 코드 작성을 줄이기 위해 편리한 도움을 주는 것일 뿐이다. 그렇기 때문에 그 구현은 위의 예제와 거의 동일하다.

> 만일 기본으로 제공되는 인증 Action이 제공하는 인증 보다 보다 복잡한 인증이 필요하다면, 고유한 인증 Action을 만드는 것이 보다 간단할 뿐 아니라 권장되는 사항이다.

### 요청을 위한 정보 추가하기

`Item` 형식의 개체와 동작하는 REST API를 생각해 보자.  `/item/:itemId` 경로에 많은 라우트가 있고, 각각의 것은 item의 내용을 참조한다고 하자. 이러한 경우에, 이 로직을 Action함수에 넣는것이 더 유용할 수 있다.

우선, `Item`이 추가된 `UserRequest`를 만들자.

@[request-with-item](code/ScalaActionsComposition.scala)

이제는 항목을 확인하고 오류 (`Left`) 또는 `ItemRequest` (`Right`)을 가지는 `Either`를 반환하는 Action을 생성해보자. 이 Action은 항목의 id를 받는 함수 내에 정의되어야 한다는데 주의하자.

@[item-action-builder](code/ScalaActionsComposition.scala)

### 요청 검증하기

마지막으로, 요청이 계속되어야 하는지를 검증하는 Action함수를 알아보자. 예를 들면, `ItemAction`에 대한 권한을 `UserAction`에 가지고 있는 사용자인지를 판단하고, 그렇지 않는 경우에는 오류를 반환해야 하는 경우와 같다.

@[permission-check-action](code/ScalaActionsComposition.scala)

### 모든걸 함께 사용하기

이제 Action을 생성할 때 (`ActionBuilder` 부터 시작해서) `andThen`를 사용하여 이러한 Action 함수들을 함께 사용할 수 있다.

@[item-action-use](code/ScalaActionsComposition.scala)


플레이는 전역으로 사용되는 관심사를 위한 [[전역 필터 API | ScalaHttpFilters]]를 제공한다.
