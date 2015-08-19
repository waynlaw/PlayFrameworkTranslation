<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# Artifact 저장소

## 타입세이프 리포지토리

모든 플레이의 artifacts는 타입세이프 저장소인 <https://repo.typesafe.com/typesafe/releases/>에 발행되고 있다.

> **주의:** 이것은 Maven2에 적합한 저장소이다.

sbt 빌드에 이를 사용하기 위해, 적절한 리졸버를 추가해야한다.(일반적으로는 `plugins.sbt` 파일에 넣는다.)

```
// The Typesafe repository
resolvers += "Typesafe Releases" at "https://repo.typesafe.com/typesafe/releases/"
```

## 스냅샷에 접근하기

스냅샷은 매일 우리의 [[Continuous Integration Server|ThirdPartyTools]]으로부터 타입세이프 스냅샷 저장소 인 <https://repo.typesafe.com/typesafe/snapshots/>에 발행되고 있다.

> **주의:** 이것은 ivy 형식의 저장소이다.

```
// The Typesafe snapshots repository
resolvers += Resolver.url("Typesafe Ivy Snapshots Repository", url("https://repo.typesafe.com/typesafe/ivy-snapshots"))(Resolver.ivyStylePatterns)
```

