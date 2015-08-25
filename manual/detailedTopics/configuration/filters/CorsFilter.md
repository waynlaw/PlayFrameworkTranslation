# Cross-Origin Resource Sharing

플레이는 CORS로 구현한 필터를 제공한다.

CORS is a protocol that allows web applications to make requests from the browser across different domains.  A full specification can be found [here|http://www.w3.org/TR/cors/].

CORS는 브라우저의 다른 도메인으로 요청을 만들 수 있는 웹 애플리케이션 프로토콜이다. 전체 명세는 [여기서|http://www.w3.org/TR/cors/] 찾을 수 있다.

## CORS 필터 활성화 하기

CORS 필터를 활성화 하기 위해 `build.sbt`에 `libraryDependencies`에 플레이 필터 프로젝트를 추가하자.

@[content](code/filters.sbt)

이제 여러분의 필터에 실제로 여러분의 프로젝트의 최상단에 `Filters` 클래스를 생성할 CORS 필터를 추가했다.

Scala
: @[filters](code/CorsFilter.scala)

Java
: @[filters](code/detailedtopics/configuration/cors/Filters.java)

## CORS 필터 환경설정하기

필터는 `application.conf`에서 환경설정 할 수 있다. 환경설정 옵션을 위한 전체 리스트는 플레이 필터 [`reference.conf`](resources/confs/filters-helpers/reference.conf)를 살펴보자.

가능한 옵션은 다음과 같다.

* `play.filters.cors.pathPrefixes` - filter paths by a whitelist of path prefixes
* `play.filters.cors.allowOrigins` - allow only requests with origins from a whitelist (by default all origins are allowed)
* `play.filters.cors.allowHttpMethods` - allow only HTTP methods from a whitelist for preflight requests (by default all methods are allowed)
* `play.filters.cors.allowHttpHeaders` - allow only HTTP headers from a whitelist for preflight requests (by default all headers are allowed)
* `play.filters.cors.exposeHeaders` - set custom HTTP headers to be exposed in the response (by default no headers are exposed)
* `play.filters.cors.supportsCredentials` - disable/enable support for credentials (by default credentials support is enabled)
* `play.filters.cors.preflightMaxAge` - set how long the results of a preflight request can be cached in a preflight result cache (by default 1 hour)

예를 들어 다음과 같이 할 수 있다.

```
play.filters.cors {
  pathPrefixes = ["/some/path", ...]
  allowOrigins = ["http://www.example.com", ...]
  allowHttpMethods = ["GET", "POST"]
  allowHttpHeaders = ["Accept"]
  preflightMaxAge = 3 days
}
```
