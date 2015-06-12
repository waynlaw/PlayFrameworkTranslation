<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# Session 과 Flash 스코프

## Play에서는 어떻게 다른가

multiple HTTP 요청 전반에서 데이터를 가지고 있어야 하는경우, Session이나 Flash 스코프에 데이터를 저장할 수 있다. Session에 데이터를 저장하는 경우, 유저 session 동안에 사용이 가능하다. 그리고 flash 스코프에 저장된 경우에는 다음 요청까지만 사용이 가능하다.

Session과 Flash 데이터가 서버에 저장되어 있다는 것이 아니라, 이어지는 HTTP 요청에서 쿠키를 이용해 추가되는 것을 이해해야 한다. 이것은 즉 데이터 사이즈가 제한 되어 있다는 (4 KB까지) 것이며, 스트링 값만 저장 가능하다는 것을 의미한다.

쿠키는 secret key로 암호화 되어 이다. 그래서 클라이언트는 쿠키 정보를 수정할 수 없다(수정하는 경우 사용이 불가능해 질 것이다). Play 세션은 캐시를 사용하도록 설계되지 않았다. 특정 세션과 관계있는 데이터를 캐시해야 하는 경우, Play 내장 cache mechanism을 이용하거나, 타깃 유저의 (캐시 하고자 하는) 데이터와 매핑된 유일한 ID값을 저장하는 용도로 세션을 사용할 수 있다. 

> session에는 별다른 시간제한이 없다. 세션은 유저가 웹 브라우저를 껐을 때 만료된다. 애플리케이션에서 시간 제한을 추가하고 싶은 경우, 애플리케이션에서 필요할 때마다 유저 Session에서 timestamp를 저장하고, 이것을 사용하면 된다. (e.g. 최대 session 기간, 최소 비활성화 기간, 등)
> There is no technical timeout for the session, which expires when the user closes the web browser. If you need a functional timeout for a specific application, just store a timestamp into the user Session and use it however your application needs (e.g. for a maximum session duration, maximum inactivity duration, etc.).

## Session에 데이터 저장하기

Session은 사실 그냥 쿠키이다. 또다른 종류의 HTTP 헤더일 뿐이지만 Play는 session 값을 저장할 수 있도록 헬퍼 메소드를 제공한다:

@[store-session](code/javaguide/http/JavaSessionFlash.java)

같은 방법으로 incoming session에서 어떤 값이라도 제거할 수 있다:

@[remove-from-session](code/javaguide/http/JavaSessionFlash.java)

## Session 값 읽기

HTTP 요청에서 incoming Session세션을 읽어올 수 있다:

@[read-session](code/javaguide/http/JavaSessionFlash.java)

## session을 통째로 버리기

session을 통째로 버리고 싶다면, 특별한 방법이 있다:

@[discard-whole-session](code/javaguide/http/JavaSessionFlash.java)

## Flash 스코프

Flash 스코프는 Session과 비슷하게 동작하지만, 2가지 다른 점이 있다:

- 데이터가 단일 요청까지만 유지된다
- Flash 쿠키는 인증된 상태가 아니기 (not signed) 때문에 유저가 수정하는 것이 가능하다.

> **중요:** flash 스코프는 간단한 비 Ajax 애플리케이션에서 성공/에러 메시지를 전
> **Important:** The flash scope should only be used to transport success/error messages on simple non-Ajax applications. As the data are just kept for the next request and because there are no guarantees to ensure the request order in a complex Web application, the Flash scope is subject to race conditions.

So for example, after saving an item, you might want to redirect the user back to the index page, and you might want to display an error on the index page saying that the save was successful.  In the save action, you would add the success message to the flash scope:

@[store-flash](code/javaguide/http/JavaSessionFlash.java)

Then in the index action, you could check if the success message exists in the flash scope, and if so, render it:

@[read-flash](code/javaguide/http/JavaSessionFlash.java)

A flash value is also automatically available in Twirl templates. For example:

@[flash-template](code/javaguide/http/views/index.scala.html)
