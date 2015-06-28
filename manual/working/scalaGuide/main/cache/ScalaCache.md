<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# Play 캐시 API

데이터를 캐시에 보관하는 것은 오늘날 애플리케이션 개발의 일반적인 최적화기법이므로, 플레이는 전역 캐시를 제공한다. 캐시의 중요한 점은 단지 바로 보관하고 바로 잊을 수 있는 것 처럼 동작해야 한다는 것이다. 

캐시에 보관된 데이터는 데이터가 사라진 경우에 다시 채우기 위해서, 데이터를 생성하는 전략이 필요하다. 이 이론은 플레이의 근본적인 이념 중 한가지이며, 세션의 생명주기 동안 값을 유지하길 원하는 Java EE와의 차이점이다. 

캐시 API 의 기본적인 구현은 [EHCache](http://ehcache.org/)이다.

## 캐시 API 포함하기

`캐시`를 의존성 리스트에 추가해야 한다. 예를 들면 `build.sbt`에서는 아래와 같다.

```scala
libraryDependencies ++= Seq(
  cache,
  ...
)
```

## 캐시 API 접근하기

[CacheApi](api/scala/index.html#play.api.cache.CacheApi)가 캐시 API를 제공한다. 그리고 다른 의존성을 가지는 컴포넌트와 같이 의존성을 주입할 수 있다. 예를 들면:

@[inject](code/ScalaCache.scala)

> **주의:** API는 추가적인 플러그인을 허용하기 위해 최소한으로 만들어져 있다. 만일 보다 상세한 API가 필요하다면, 캐시 플러그인을 사용해야 할것이다.

단순한 API를 사용하면, 캐시에 데이터를 보관할 수 있다.

@[set-value](code/ScalaCache.scala)

그리고 이후에 그 데이터를 가져올 수 있다.

@[get-value](code/ScalaCache.scala)

또한 캐시에서 데이터를 가져오거나, 데이터가 사라졌을때 데이터를 설정할 수 있는 편리한 도우미를 제공한다.

@[retrieve-missing](code/ScalaCache.scala)

기간을 전달해서 만료 기간을 설정할 수 있다. 기본적으로는 영원히 만료되지 않는다.

@[set-value-expiration](code/ScalaCache.scala)

캐시에서 항목을 제거하려면 `remove`함수를 사용해야 한다.

@[remove-value](code/ScalaCache.scala)

## 다른 캐시에 접근하기

다른 캐시에 접근하는 것 또한 가능하다. 기본 캐시는 `play`라고 불리며, `ehcache.xml` 파일을 생성하여 설정할 수 있다. 추가적인 캐시는 다른 설정이나 구현을 통해서 설정할 수 있다.

만일 여러개의 다른 ehcache 캐시에 접근하고자 한다면, 다음과 같이 `application.conf` 파일에서, 그 정보들을 바인드해야한다고 알려주어야 한다.

    play.cache.bindCaches = ["db-cache", "user-cache", "session-cache"]

이제 그것들을 주입했을 때, 이런 서로다른 캐시에 접근하기 위해서는, 의존성에 [NamedCache](api/java/play/cache/NamedCache.html) 적임자를 명시해야 한다. 예를 들면:

@[qualified](code/ScalaCache.scala)

## HTTP 응답 캐시하기

기본적인 Action 병합을 통해서 간단하게 스마트 캐시를 만들 수 있다.

> **주의:** 플레이 HTTP `Result`는 캐시에 안전하게 보관할 수 있고, 나중에 사용할 수 있다.

[Cached](api/scala/index.html#play.api.cache.Cached) 클래스가 캐시된 Action을 만드는 것을 도와줄 것이다.

@[cached-action-app](code/ScalaCache.scala)

`"homePage"`처럼 고정된 키를 이용해서, Action의 결과를 캐시할 수 있다.

@[cached-action](code/ScalaCache.scala)

만일 결과가 다양하면, 그 결과를 서로 다른 키들을 통해서 캐시할 수 있다. 예를 들어 각각의 사용자가 서로 다른 캐시 결과를 가지고 있다고 하자.

@[composition-cached-action](code/ScalaCache.scala)

### 캐시 조절하기

캐시하고자 하는 것과 캐시에서 제외하려고 하는 것을 쉽게 조절할 수 있다.

만일 오직 200 OK 결과만을 원할 수 있을 것이다.

@[cached-action-control](code/ScalaCache.scala)

또는 몇 분 동안 404 Not Found 를 캐시하기를 원할 수 있을 것이다.

@[cached-action-control-404](code/ScalaCache.scala)

## 직접 구현하기

[CacheApi](api/scala/index.html#play.api.cache.CacheApi)를 직접 구현하여 제공하는 것도 가능하다. 이때에는 기본 구현을 대체하거나 그대로 둘 수 있다.

기본 구현을 대체하기 위해서는, `application.conf`를 설정하여 기본 구현을 비활성화 해야한다.

```
play.modules.disabled += "play.api.cache.EhCacheModule"
```

그리고 간단히 CacheAPi를 구현하고 DI container에 바인드 해야 한다.

기본 구현에 추가로 Cache API의 구현을 제공하기 위해서는, 검증자를 새로 만들거나 `NamedCache` 검증자를 재사용할 수 있다.
