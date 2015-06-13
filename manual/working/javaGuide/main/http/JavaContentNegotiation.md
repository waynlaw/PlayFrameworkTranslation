<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# Content 전략

Content 전략은 같은 자원(URI)을 다른 방법으로 표현할 수 있게 도와준다. 이것은  *예를 들어* 웹서비스에서 특정 아웃풋 형식을 지원(XML, JSON 등)할 때 유용하게 쓰인다. `Accept*` 요청 헤서를 이용하여 서버 기반 전략이 수행되게 된다.  [HTTP specification](http://www.w3.org/Protocols/rfc2616/rfc2616-sec12.html)에서 관련 content 전략 정보를 더 찾아볼 수 있다.


## 언어

`play.mvc.Http.RequestHeader#acceptLanguages` 메서드를 사용하여 지원할 언어 리스트를 가져올 수 있다. 이 메소드는 `Accept-Language` 헤더에서 해당 정보를 가져와 quality 값에 따라 정렬을 해준다. Play는 HTTP 컨텍스트 요청의 `lang`값을 설정하기 위해 이 리스트를 사용한다. 따라서 자동으로 최적화된 언어를 사용할 수 있게 된다. (사용자 애플리케이션에서 지원할 수 있거나, 애플리케이션이 기본으로 사용하는 언어를 사용하게 된다.)


## Content

비슷하게 `play.mvc.Http.RequestHeader#acceptedTypes` 메서드는 사용자 요청에서 MIME 타입의 허용가능한 결과값 리스트를 제공해 준다. `Accept` 헤더에서 해당 정보를 가져와 quality 요소에 따라 정렬을 해준다.

현재 요청에 `play.mvc.Http.RequestHeader#accepts` 메서드를 사용하여 주어진 MIME 타입이 허용 가능한지 테스트 해 볼 수 있다: hod:

@[negotiate-content](code/javaguide/http/JavaContentNegotiation.java)
