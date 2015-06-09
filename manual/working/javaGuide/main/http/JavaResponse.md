<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# HTTP response 변경하기

## 기본 Content-Type 변경하기

body에 명시되어 있는 Java 값을 통해 결과 컨텐츠 타입을 자동으로 추론할 수 있다.

예를 들어:

@[text-content-type](code/javaguide/http/JavaResponse.java)

는 `Content-Type` 헤더가 `text/plain`으로 자동 설정이 된다. 반면에:

@[json-content-type](code/javaguide/http/JavaResponse.java)

는 `Content-Type` 헤더를 `application/json`로 설정하게 된다.

이 것은 상당히 유용하다. 왜냐하면 때때로 헤더를 변경하고 싶은 경우가 생기기 때문이다. `Content-Type` 헤더만 다른 비슷한 결과값을 생성하고 싶을 경우 `as(newContentType)` 메소드만 사용하면 된다:

@[custom-content-type](code/javaguide/http/JavaResponse.java)

HTTP response 컨텍스트에서 content type을 변경할 수도 있다:

@[context-content-type](code/javaguide/http/JavaResponse.java)

## HTTP response 헤더를 변경하기

@[response-headers](code/javaguide/http/JavaResponse.java)

HTTP 헤더를 설정하는 경우 이전 값을 자동으로 버리게 된다는 점을 유의하자.

## 쿠키 설정하기와 버리기

쿠키는 HTTP 헤더 중 특별한 형식을 가지고 있다. 하지만 Play는 쿠키를 쉽게 만들어 주는 헬퍼를 제공하고 있다. 

HTTP response로 쉽게 Cookie를 추가할 수 있다:

@[set-cookie](code/javaguide/http/JavaResponse.java)

세부사항을 설정하고 싶은 경우 (경로, 도메인, expiry, secure인지 아닌지나, HTTP only flag가 설정되어야 하는지 등), 메소드를 오버로딩하여 설정이 가능하다:

@[detailed-set-cookie](code/javaguide/http/JavaResponse.java)

웹 브라우저에서 이전에 저장된 쿠키를 버리기 위해선:

@[discard-cookie](code/javaguide/http/JavaResponse.java)

쿠키를 설정할 때 경로나, 도메일을 설정되어 이는 경우, 버리려는 쿠키가  같은 경로로 설정 되어 있는지 확인해야 한다. 브라우저는 경로와 도메인 쿠키 이름이 일치하는 경우에만 쿠키를 제거하기 때문이다.

## 텍스트 결과값에 대한 character 인코딩 설정하기

텍스트 기반의 HTTP response를 설정에서 character 인코딩을 올바르게 설정하는 것은 매우 중요하다. Play는 기본 값으로 `utf-8`을 사용하고 있다.

텍스트 response를 네트워크 소켓에 전송하기 위해 바이트로 변환하는 경우와, `Content-Type` 헤더에 `;charset=xxx` 확장자를 추가하는 경우에 인코딩이 사용된다.

`Result` 값을 생성할때 인코딩을 명시할 수 있다:

@[charset](code/javaguide/http/JavaResponse.java)
