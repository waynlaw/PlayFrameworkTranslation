<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 보안 헤더 환경설정하기

Play provides a security headers filter that can be used to configure some default headers in the HTTP response to mitigate security issues and provide an extra level of defense for new applications.

## 보안 헤더 필터 활성화 하기

보안 헤더 필터를 활성화 하기 위해 `build.sbt`에 `libraryDependencies`에 플레이 필터 프로젝트를 추가하자.

@[content](code/filters.sbt)

이제 여러분의 필터에 실제로 여러분의 프로젝트의 최상단에 `Filters` 클래스를 생성할 보안 헤더 필터를 추가했다.

Scala
: @[filters](code/SecurityHeaders.scala)

Java
: @[filters](code/detailedtopics/configuration/headers/Filters.java)

## 보안 헤더 환경설정하기

스칼라 문서는 [play.filters.headers](api/scala/index.html#play.filters.headers.package) 패키지에서 사용할 수 있다.

필터는 HTTP 응답에서 자동으로 헤더를 설정할 것이다. `application.conf`에서 다음 설정을 통해 환경설정을 수행할 수 있다.

* `play.filters.headers.frameOptions` - sets <a href="https://developer.mozilla.org/en-US/docs/HTTP/X-Frame-Options">X-Frame-Options</a>, "DENY" by default.
* `play.filters.headers.xssProtection` - sets <a href="http://blogs.msdn.com/b/ie/archive/2008/07/02/ie8-security-part-iv-the-xss-filter.aspx">X-XSS-Protection</a>, "1; mode=block" by default.
* `play.filters.headers.contentTypeOptions` - sets <a href="http://blogs.msdn.com/b/ie/archive/2008/09/02/ie8-security-part-vi-beta-2-update.aspx">X-Content-Type-Options</a>, "nosniff" by default.
* `play.filters.headers.permittedCrossDomainPolicies` - sets <a href="http://www.adobe.com/devnet/articles/crossdomain_policy_file_spec.html">X-Permitted-Cross-Domain-Policies</a>, "master-only" by default.
* `play.filters.headers.contentSecurityPolicy` - sets <a href="http://www.html5rocks.com/en/tutorials/security/content-security-policy/">Content-Security-Policy</a>, "default-src 'self'" by default.

예를들어 `null` 값으로 환경설정하여 어떤 헤더를 비활성화 할 수 있다.

    play.filters.headers.frameOptions = null

모든 환경설정 옵션을 살펴보기 위해, 플레이 필터 [`reference.conf`](resources/confs/filters-helpers/reference.conf)를 살펴보자.