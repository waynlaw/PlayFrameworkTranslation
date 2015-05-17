<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 플레이 애플리케이션의 구조

## 플레이 애플리케이션의 레이아웃

플레이 애플리케이션의 레이아웃은 최대한 간단하게 유지하고자 표준화 되어있다. 첫 성공적인 컴파일 이후에 플레이 애플리케이션의 구조는 아래와 같다.

```
app                      → 애플리케이션 소스
 └ assets                → 컴파일된 자산 소스
    └ stylesheets        → 대표적으로 LESS와 CSS 소스
    └ javascripts        → 대표적으로 커피스크립트 소스
 └ controllers           → 애플리케이션 컨트롤러
 └ models                → 애플리케이션 비즈니스 계층
 └ views                 → 뷰 템플릿
build.sbt                → 애플리케이션 빌드 스크립트
conf                     → 환경설정 파일과 다른 컴파일이 필요없는 자원 (on classpath)
 └ application.conf      → 메인 환경설정 파일
 └ routes                → 라우트 정의
public                   → 공용 자산
 └ stylesheets           → CSS 파일
 └ javascripts           → 자바스크립트 파일
 └ images                → 이미지 파일
project                  → sbt 환경설정 파일
 └ build.properties      → sbt 프로젝트를 위한 생성자
 └ plugins.sbt           → 플레이를 위한 정의를 포함한 sbt 플러그인
lib                      → 관리되지 않는 라이브러리 의존성
logs                     → 로그 폴더
 └ application.log       → 기본 로그 파일 
target                   → 생성된 것
 └ scala-2.10.0            
    └ cache              
    └ classes            → 컴파일된 클래스 파일
    └ classes_managed    → 관리되는 클래스 파일 (템플릿, ...)
    └ resource_managed   → 관리되는 자원 (less, ...)
    └ src_managed        → 생성된 소스 (템플릿, ...)
test                     → 유닛 테스트 및 기능 테스트를 위한 소스 폴더
```

## app/ 디렉터리

`app` 디렉터리는 자바나 스칼라 소스 코드, 템플릿 그리고 컴파일된 자산 소스처럼 모든 실행할 수 있는 것을 포함한다. 

`app` 디렉터리에는 3개의 패키지가 있고, 각각은 MVC 구조 패턴의 구성요소이다.

- `app/controllers`
- `app/models`
- `app/views`

물론 원한다면 `app/utils` 패키지처럼 필요한 것을 추가할 수 있다.

> 플레이에서는 controllers, models 그리고 views 패키지의 이름은 컨벤션일 뿐이다. 필요하면 `com.yourcompany`처럼 변경해서 사용할 수 있다.

또한 [LESS sources](http://lesscss.org/)나 [CoffeeScript sources](http://coffeescript.org/)처럼 컴파일된 자산을 위한 `app/assets`처럼 추가적인 디렉터리도 올 수 있다.

## public/ 디렉터리

자원은 `public` 디렉터리에 저장되며 웹 서버에 의해 직접 제공될 정적 자산이다.

이 디렉터리는 이미지, CSS 스타일시트와 자바스크립트 파일을 위해 3개의 하위 디렉터리로 분할된다. 모든 플레이 애플리케이션을 안정성을 유지하기 위해 정적 자산을 이처럼 구성할 필요가 있다.

> 최근에 생성된 애플리케이션에서 `/public` 디렉터리는 `/assets` URL 경로와 맵핑된다. 그러나 이를 쉽게 변경할 수 있으며, 심지어 정적 자산을 위해서 각각의 디렉터리를 사용할 수 있다.

## conf/ 디렉터리

`conf` 디렉터리에는 애플리케이션의 환경설정 파일이 들어있다. 메인 환경설정 파일은 2가지가 있다.

- `application.conf` : 애플리케이션을 위한 메인 환경설정 파일이며, 환경설정 파라미터를 주로 여기에 저장한다.

- `routes` : 라우트를 정의한 파일이다.

애플리케이션을 위한 구체적인 옵션을 환경설정을 추가하려면 `application.conf` 파일에 옵션을 더 추가하는 것이 아주 좋은 생각이다.

구체적인 환경설정 파일이 필요한 라이브러리가 있다면, `conf` 디렉터리에 파일을 넣는 것이 좋다.

## lib/ 디렉터리

`lib`는 부가적인 디렉터리이며, 관리되지 않는 라이브러리 의존성 즉, 빌드 시스템 외적으로 직접 관리하는 모든 JAR 파일을 여기에 저장한다. 이 디렉터리에 JAR파일을 넣으면 애플리케이션 클래스패스에 추가될 것이다.

## build.sbt 파일

일반적으로 프로젝트의 주요 빌드 선언은 프로젝트의 최 상단에 위치한 `build.sbt`에서 찾을 수 있다. `project/` 디렉터리의 `.scala` 파일들도 또한 프로젝트의 빌드에 대해 선언할 수 있다.

## project/ 디렉터리

`project` 디렉터리에는 sbt 빌드 정의를 담고 있다.

- `plugins.sbt`는 이 프로젝트에서 사용하는 sbt 플러그인들을 정의한다.
- `build.properties`는 앱에서 빌드를 위한 sbt 버전을 저장한다.

## target/ 디렉터리

`target` 디렉터리는 빌드 시스템에 의해 생성된 모든 것을 담고 있다. 아래에 무엇이 생성되었는지에 대한 유용한 정보들을 나타냈다.

- `classes/` 디렉터리는 모든 컴파일된 클래스를 담고 있다. (자바 또는 스칼라 소스).
- `classes_managed/` 디렉터리는 프레임워크에 의해 관리되는 클래스들만 담고 있다. (라우터나 템플릿 시스템에서 생성된 것과 같은). IDE 프로젝트에 외부 클래스 폴더로 이 폴더를 추가하는 것이 유용할 수 있다.
- `resource_managed/` 디렉터리는 대표적으로 LESS CSS나 커피스크립트 컴파일된 결과처럼 컴파일된 자산과 같이 생성된 자원을 담고 있다. 
- `src_managed/` 디렉터리는 템플릿 시스템에서 생성된 스칼라 소스처럼 생성된 자원을 담고 있다.
- `web/` 디렉터리는 `app/asserts` 또는 `public` 폴더에서 [sbt-web](https://github.com/sbt/sbt-web#sbt-web)에 의해 처리된 자산을 담고 있다.

## 전형적인 .gitignore 파일

생성된 폴더는 버전 컨트롤 시스템에서는 무시되어야 한다. 여기 플레이 애플리케이션을 위한 전형적인 `.gitignore` 파일이 있다.

```txt
logs
project/project
project/target
target
tmp
dist
.cache
```

## 기본 SBT 레이아웃

SBT나 메이븐을 이용하도록 기본적인 레이아웃을 사용할 수 있다. 이 레이아웃은 실험적인 것이며 이슈가 있을 수 있다. 이 레이아웃을 사용하기 위해 `disablePlugins(PlayLayoutPlugin)`을 사용하자.

```
build.sbt                  → 애플리케이션 빌드 스크립트
src                        → 애플리케이션 소스
 └ main                    → 컴파일된 자산 소스
    └ java                 → 자바 소스
       └ controllers       → 자바 컨트롤러
       └ models            → 자바 비즈니스 레이어
    └ scala                → 스칼라 소스
       └ controllers       → 스칼라 컨트롤러
       └ models            → 스칼라 비즈니스 레이어
    └ resources            → 환경설정 파일과 다른 컴파일이 필요없는 자원 (on classpath)
       └ application.conf  → 메인 환경설정 파일
       └ routes            → 라우트 정의
    └ twirl                → 템플릿
    └ assets               → 컴파일된 자산 소스
       └ css               → 대표적으로 LESS와 CSS 소스
       └ js                → 대표적으로 커피스크립트 소스
    └ public               → 공용 자산
       └ css               → CSS 파일
       └ js                → 자바스크립트 파일
       └ images            → 이미지 파일
 └ test                    → 유닛 테스트 및 기능 테스트
    └ java                 → 유닛 및 기능 테스트를 위한 자바 소스 폴더
    └ scala                → 유닛 및 기능 테스트를 위한 스칼라 소스 폴더
    └ resources            → 유닛 및 기능 테스트를 위한 자원 폴더
project                    → sbt 환경설정 파일
 └ build.properties        → sbt 프로젝트를 위한 생성자
 └ plugins.sbt             → 플레이를 위한 정의를 포함한 sbt 플러그인 
lib                        → 관리되지 않는 라이브러리 의존성
logs                       → 로그 폴더
 └ application.log         → 기본 로그 파일 
target                     → 생성된 것
 └ scala-2.10.0            
    └ cache              
    └ classes              → 컴파일된 클래스 파일
    └ classes_managed      → 관리되는 클래스 파일 (템플릿, ...)
    └ resource_managed     → 관리되는 자원 (less, ...)
    └ src_managed          → 생성된 소스 (템플릿, ...)
```
