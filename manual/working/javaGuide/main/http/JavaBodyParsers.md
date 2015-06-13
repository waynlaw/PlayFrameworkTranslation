<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# Body 파서

## body 파서는 무엇인가?

HTTP 요청은 (최소한 POST, PUT 요청을 해야 하는 경우) body를 포함하고 있다. 이 body는  Content-Type 헤더에서 특정 포멧으로 설정이 가능하다. **body 파서**는 이 요청의 body를 Java 값으로 변환해준다.

> **주의:** `BodyParser`구현체를 Java를 사용해서 재정의 할 수 없다. 왜냐하면 Play의 `BodyParser`는 `Iteratee[Array[Byte], A]`를 사용하여 body content를 추가하는 과정을 제어하는데, 반드시 Scala로 구현이 되어야 하기 때문이다.

> 하지만 Play의 `BodyParser`는 대부분 사용 케이스에 적합한 기능을 제공한다(Json, Xml, Text 파싱, 파일 업로드). Java에서 확장해 쓰고 싶다면 기본 파서를 재 사용하는 것도 가능하다. 예를 들어 텍스트 파서를 기본으로 RDF 파서를 만들 수도 있을 것이다.

## `BodyParser` Java API

요청 body 작업을 해야 할 때, 컨트롤러에서 아래의 import를 확인해야 한다:

@[imports](code/javaguide/http/JavaBodyParsers.java)

Java API에서 모든 body 파서는 `play.mvc.Http.RequestBody`값을 반드시 생성해야 한다. body 파서가 생성한 이 값은 `request().body()`를 통해 불러오는 게 가능하다:

@[request-body](code/javaguide/http/JavaBodyParsers.java)

`@BodyParser.Of` 애노테이션을 사용하는 특별한 action을 이용하기 위해 `BodyParser`를 명시할 수 있다:

@[particular-body-parser](code/javaguide/http/JavaBodyParsers.java)

## `Http.RequestBody` API

Java API에서 제공하는 모든 body 파서들은 `play.mvc.Http.RequestBody` 값을 생성한다고 할 수 있다. 이 body 객체로부터 적절한 Java 타입으로 변환된 요청 body content를 불러올 수 있다.

> **주의:** `asText()` 나 `asJson()` 같은 `RequestBody` 메소드는, 파서가 지원하지 않는 요청 body를 받는 경우 null을 리턴하게 된다. 예를 들어 `@BodyParser.Of(BodyParser.Json.class)` 애노테이션이 적용된 action 메소드가 생성한 body에서 `asXml()를 호출하는 경우 `null`을 리턴하게 될 것이다.

## 기본 body 파서: AnyContent

특별한 body 파서를 지정하지 않는 이상, Play는 `Content-Type` 헤더에서 적절한 content 타입을 추정하여 기본값으로 지정해 사용하게 된다:

- **text/plain**: `String`, `asText()`를 통해 접근 가능
- **application/json**: `JsonNode`, `asJson()`를 통해 접근 가능
- **application/xml**, **text/xml** or **application/XXX+xml**: `org.w3c.Document`, `asXml()`를 통해 접근 가능
- **application/form-url-encoded**: `Map<String, String[]>`, `asFormUrlEncoded()`를 통해 접근 가능
- **multipart/form-data**: `Http.MultipartFormData`, `asMultipartFormData()`를 통해 접근 가능
- 그 외 다른 content type: `Http.RawBuffer`, `asRaw()`를 통해 접근 가능

예를 들어:

@[default-parser](code/javaguide/http/JavaBodyParsers.java)

## 최대 content 길이

텍스트 기반 body 파서 (예를 들어 **text**, **json**, **xml** 나 **formUrlEncoded**)는 최대 content 길이를 사용한다. 왜냐하면 모든 content를 메모리에 불러와야 하기 때문이다. 기본값으로 최대로 파싱할 수 있는 content 길이는 100KB이다. 이 값은 `application.conf` 파일에서 `play.http.parser.maxMemoryBuffer` 프로퍼티 값을 지정하여 재정의가 가능하다:

    play.http.parser.maxMemoryBuffer=128K

row (데이터)나 `multipart/form-data` 파서와 같이 디스크에 content를 버퍼링 처리해야하는 파서를 위해, `play.http.parser.maxDiskBuffer` 프로퍼티에 최대 content 길이를 명시할 수 있다. 기본 값은 10MB이다. `multipart/form-data` 파서는 데이터 필드의 조합을 위해 텍스트 최대 길이 프로퍼티 값을 강제로 적용하게 만들 수도 있다.

또한 `@BodyParser.Of` 애노테이션을 이용하면 주어진 action에 기본 최대 content 길이를 재정의 할 수도 있다: 

@[max-length](code/javaguide/http/JavaBodyParsers.java)
