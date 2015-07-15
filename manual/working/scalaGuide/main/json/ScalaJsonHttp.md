<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# HTTP와 JSON

플레이는 JSON 라이브러리와 결합된 HTTP API를 이용한 JSON형식의 HTTP 요청과 응답을 지원한다.

> 컨트롤러, 액션, 라우팅에 대해 더 자세히 알아보려면 [[HTTP 프로그래밍 | ScalaActions]]을 참고하자.

GET으로 항목의 목록을 받고, 새로운 항목을 생성하는 POST를 받는 간단한 RESTful 웹서비스를 디자인 하는데 필요한 컨셉에 대해 알아보겠다. 서비스의 모든 데이터의 타입은 JSON을 사용할 것이다.

서비스를 위해 사용할 모델은 아래와 같다.

@[model](code/ScalaJsonHttpSpec.scala)

## 항목의 목록을 JSON으로 제공하기

컨트롤러에 필요한 것을 추가하는 것 부터 시작해보자.

@[controller](code/ScalaJsonHttpSpec.scala)

`Action`을 작성하기 전에, 모델을 `JsValue`로 변환하기 위한 연결이 필요하다. 암시적인 `Writes[Place]`를 선언하여 해결할 수 있다.

@[serve-json-implicits](code/ScalaJsonHttpSpec.scala)

`Action`을 작성해야 한다.

@[serve-json](code/ScalaJsonHttpSpec.scala)

`Action`은 `Place` 객체의 리스트를 가져와서 암시적인 `Writes[Place]`와 `Json.toJson`을 이용해서 `JsValue`로 변환한다. 그 후에 이를 반환하여 응답의 내용으로 사용한다. 플레이는 JSON으로 결과를 인지하고, 응답에 적절한 `Content-Type` 헤더와 내용을 설정한다.

마지막 절차는 `conf/routes`내의 라우트에 `Action`을 추가하는 것이다.

```
GET   /places               controllers.Application.listPlaces
```

우리는 브라우져나 HTTP도구를 사용해서 요청을 만들어 액션을 테스트할 수 있다. 아래에는 unix의 명령창 도구인 [cURL](http://curl.haxx.se/)를 이용한 예제이다.

```
curl --include http://localhost:9000/places
```

응답:

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Content-Length: 141

[{"name":"Sandleford","location":{"lat":51.377797,"long":-1.318965}},{"name":"Watership Down","location":{"lat":51.235685,"long":-1.309197}}]
```

## JSON에서 새 항목 생성하기

이 `Action`을 위해서는 `JsValue`를 모델로 변환해 주는 암시적인 `Reads[Place]`를 필요로 한다.

@[handle-json-implicits](code/ScalaJsonHttpSpec.scala)

이 다음은 `Action`을 정의해야 한다.

@[handle-json-bodyparser](code/ScalaJsonHttpSpec.scala)

이 `Action`은 목록을 만드는 경우보다 좀더 복잡하다. 주의해야 할 점은 아래와 같다.

- 이 `Action`은 헤더의 `Content-Type`이 `text/json`이거나 `application/json`인 요청을 필요로 하며, 내용에는 생성할 항목의 내용을 담고 있어야 한다.
- 이는 요청을 처리하기 위해 JSON을 위한 특정 `BodyParser`을 사용하며, `JsValue`로 `request.body`를 제공한다.
- 암시적인 `Reads[Place]`에 의존적인 변환을 하기위해 `validate`함수를 사용한다.
- 검증 결과를 처리하기 위해, 실패와 성공 절차와 `fold`를 사용한다. 이 방식은 폼 제출에서 사용한 방식과 유사한 방식이다.
- 이 `Action` 또한 JSON 응답을 보낸다.

마지막으로 `conf/routes`에 라우트를 추가한다.

```
POST  /places               controllers.Application.savePlace
```

이 액션에서 성공과 실패 흐름을 테스트하기위해서 올바른 요청이나, 잘못된 요청을 사용할 수 있다.

올바른 데이터를 이용해서 액션을 테스트 해보자.

```
curl --include
  --request POST
  --header "Content-type: application/json" 
  --data '{"name":"Nuthanger Farm","location":{"lat" : 51.244031,"long" : -1.263224}}' 
  http://localhost:9000/places
```

응답:

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Content-Length: 57

{"status":"OK","message":"Place 'Nuthanger Farm' saved."}
```

"name" 항목이 빠져있는 잘못된 데이터를 이용하여, 액션을 테스트 해보자.

```
curl --include
  --request POST
  --header "Content-type: application/json"
  --data '{"location":{"lat" : 51.244031,"long" : -1.263224}}' 
  http://localhost:9000/places
```

응답:

```
HTTP/1.1 400 Bad Request
Content-Type: application/json; charset=utf-8
Content-Length: 79

{"status":"KO","message":{"obj.name":[{"msg":"error.path.missing","args":[]}]}}
```
"lat" 항목의 타입이 잘못된 데이터로 액션을 테스트 해보자.

```
curl --include
  --request POST
  --header "Content-type: application/json" 
  --data '{"name":"Nuthanger Farm","location":{"lat" : "xxx","long" : -1.263224}}' 
  http://localhost:9000/places
```
응답:

```
HTTP/1.1 400 Bad Request
Content-Type: application/json; charset=utf-8
Content-Length: 92

{"status":"KO","message":{"obj.location.lat":[{"msg":"error.expected.jsnumber","args":[]}]}}
```

## 요약
플레이는 JSON을 이용한 REST를 지원하도록 설계되었고, 이런 서비스를 간단하게 개발할 수 있도록 되어있다. 모델을 위해 `Reads` 과 `Writes` 를 다량으로 작성하는 것은, 다음 장에서 다룬다.
