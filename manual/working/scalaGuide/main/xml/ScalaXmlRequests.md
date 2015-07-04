<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# XML 요청을 제공하고 처리하기

## XML 요청 처리하기

XML 요청은 올바른 XML내용을 요첨 내용으로 사용하는 HTTP 요청이다. 이것은 반드시 `Content-Type` 헤더안의 MIME 타입을 `application/xml` 또는 `text/xml`로 명시해야 한다.

기본적으로 `Action`은 내용을 XML 형식(실제로는 `NodeSeq`형식)으로 받을 수 있게 해주는 **any content** 내용 파서를 사용한다.

@[xml-request-body-asXml](code/ScalaXmlRequests.scala)

내용을 바로 XML로 파싱하도록 플레이에게 요청하기 위해서는, 고유한 `BodyParser`를 지정하는 것이 더 나은(더 간단한) 방법이다.

@[xml-request-body-parser](code/ScalaXmlRequests.scala)

> **주의:** XML 내용 파서를 사용할 때에는, `request.body` 값이 바로 올바른 `NodeSeq`가 된다. 

이것을 명령창에서 **cURL**을 통해서 테스트 할 수 있다.

```
curl 
  --header "Content-type: application/xml" 
  --request POST 
  --data '<name>Guillaume</name>' 
  http://localhost:9000/sayHello
```

이 요청은 아래의 응답으로 변환된다.

```
HTTP/1.1 200 OK
Content-Type: text/plain; charset=utf-8
Content-Length: 15

Hello Guillaume
```

## XML 응답 제공하기

이전의 예제에서는 XML 요청에 대해서 살펴봤지만, `text/plain` 응답을 제공하였다. 이것을 올바른 XML HTTP 응답을 돌려주는 것으로 변경해보자.

@[xml-request-body-parser-xml-response](code/ScalaXmlRequests.scala)

이제 아래와 같이 응답할 것이다.

```
HTTP/1.1 200 OK
Content-Type: application/xml; charset=utf-8
Content-Length: 46

<message status="OK">Hello Guillaume</message>
```
