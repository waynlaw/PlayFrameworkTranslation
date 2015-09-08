<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 파일 문법과 기능 환경설정하기

플레이에서 사용하는 환경설정 파일의 기초는 [Typesafe config library](https://github.com/typesafehub/config)이다.

The configuration file of a Play application must be defined in `conf/application.conf`. It uses the [HOCON format](https://github.com/typesafehub/config/blob/master/HOCON.md).

플레이 애플리케이션의 환경설정 파일은 `conf/application.conf`에 정의되어야 한다. [HOCON format](https://github.com/typesafehub/config/blob/master/HOCON.md).을 사용한다.

`application.conf`파일 뿐만 아니라 환경설정은 다른 장소에도 올 수 있다.

* 기본 설정은 클래스패스에서 발견될 수 있는 어떤 `reference.conf` 파일에서 로드된다. 대부분의 Play JARs는 `reference.conf` 파일을 기본 설정으로 포함한다. `application.conf`의 설정은 `reference.conf` 파일의 설정을 오버라이드 할 것이다.
* 시스템 프로퍼티를 사용하여 환경설정을 하는 것도 가능하다. 시스템 프로퍼티는 `application.conf` 설정을 오버라이드 한다.

## 다른 환경설정 파일 명시하기

실환경에서 기본 `application.conf`는 클래스패스로 부터 로드된다. 시스템 프로퍼티로 다른 config 소스를 강제 할 수 있다.

* `config.resource`는 확장자을 포함하는 리소스 이름을 명시한다. 예를들어 `application.conf`이지 `application`이 아니다.
* `config.file`는 파일 시스템 경로를 명시하며, 마찬가지로 확장자를 포함해야 한다.

이들 시스템 프로퍼티는 추가되는게 아닌 `application.conf`을 위한 대체값을 명시한다. `application.conf` 파일에서 특정 값을 계속 사용하고 싶다면 여러분의 다른 `.conf` 파일에 파일의 최상단에 `include "application"`을 작성하여 `application.conf`를 포함할 수 있다. 새로운 `.conf` 파일에 `application.conf`의 설정을 포함시킨 이후에 오버라이드 하고 싶은 설정을 명시할 수 있다.

## Akka 사용하기

Akka는 플레이 애플리케이션을 위해 정의된 것과 동일한 환경설정 파일을 사용할 것이다. 이는 `application.conf` 파일에 Akka를 위한 어떤 환경설정을 할 수 있다는 의미이다. 플레이에서 Akka는 `play.akka` 설정으로 설정을 읽을 것이며 `akka` 설정은 읽지 않는다.

## `run` 명령어 사용하기

`run` 명령을 사용하여 애플리케이션을 실행할 때 환경설정에 대해 알아야 할 특별한 몇 가지가 있다.

### 추가적인 `devSettings`

`build.sbt`에 `run` 명령어를 위한 추가적인 설정을 할 수 있다. 이 설정은 애플리케이션의 배포시에는 사용되지 않을 것이다.

```
devSettings := Map("play.akka.loglevel" -> "DEBUG")
```

### `application.conf`에 HTTP 서버 설정

`run`모드에서 플레이의 HTTP 서버는 애플리케이션이 컴파일되기 전에 실행된다. 이는 HTTP 서버가 시작시에 `application.conf` 파일을 접근할 수 없다는 의미이다. 만약 `run` 명령어를 사용하는 것외에 HTTP 서버 설정을 오버라이드 하고 싶다면 `application.conf`파일은 사용할 수 없다. 대신 시스템 프로퍼티를 사용하거나 위에서 설명한 `devSettings`를 사용해야 한다. 서버 설정의 예제는 HTTP 포트이다. 다른 서버 설정은 [[여기|ProductionConfiguration]]에서 살펴볼 수 있다.

```
> run -Dhttp.port=1234
```

## HOCON 문법

HOCON은 JSON과 유사하다. JSON의 명세는 http://json.org/ 에서 살펴볼 수 있다.

### JSON에서 변경되지 않은 것들

 - 파일은 유효한 UTF-8 이어야 한다.
 - 따옴표 문자열은 JSON 문자열과 동일한 형식이다.
 - 값은 가능한 타입을 가진다. 문자열, 숫자, 객체, 배열, 불리언, null
 - 가능한숫자 형식은 JSON과 일치한다. JSON에서 처럼 `NaN`처럼 몇몇 가능한 부동소수점 값은 표현되지 않는다.

### 주석

`//`과 `#`사이에 어떤 것과 다음 새로운 줄은 주석으로 간주되며 `//`나 `#`이 따옴표 문자열 사이가 아닌 한 무시된다.

### 최상위 괄호 생략하기

JSON 문서는 배열이나 객체를 최상단 가져야 한다. 비어있는 파일은 유효하지 않은 문서이며, 배열이나 객체가 아닌 것은 오직 문자열과 같은 값을 포함한 파일이다.

HOCON에서는 파일이 중 괄호나 대 괄호로 시작하지 않아도 `{}` 중괄호가 있는 것 처럼 해석 된다. 

HOCON 파일은 `{`가 열리는 걸 생략 했지만 여전히 `}`로 닫히면 유효하지 않은 것이며, 중 괄호는 한 쌍을 이루어야 한다.

### 키-값 구분자

`=` 문자는 JSON이 `:`이 허용하는 어디서든지 사용할 수 있다.

만약에 키 다음에 `{`가 온다면 `:`나 `=`는 생략될 수 있다. 그래서 `"foo" {}`는 `"foo" : {}"`와 동일한 의미이다.

### 콤마

배열에서 값과 객체에서 필드는 그것들 사이에 하나의 ASCII 뉴라인 (`\n`), 숫자 값 10)을 가지고 있다면 그들 사이에 콤마는 필요하지 않다.

배열의 마지막 요소나 객체의 마지막 필드에는 단일 콤마가 올 수 있다. 이 추가적인 콤마는 무시된다.

 - `[1,2,3,]`는 `[1,2,3]`와 동일한 배열이다.
 - `[1\n2\n3]`는 `[1,2,3]`과 동일한 배열이다.
 - `[1,2,3,,]`는 유효하지 않다. 2개의 콤마가 있기 때문이다.
 - `[,1,2,3]`는 유효하지 않다. 콤마가 처음에 왔기 때문이다.
 - `[1,,2,3]`는 유효하지 않다. 2개의 콤마가 중간에 왔기 때문이다.
 - 객체의 필드에도 위와 동일한 콤마 규칙이 적용된다.

### 키의 중복

JSON 명세에는 동일한 객체에서 중복된 키를 처리할지가 명확하게 되어있지 않다. HOCON에서는 양쪽의 값이 객체가 아닌한 나중에 나온 중복된 키로 이전에 것을 오버라이드한다. 두개의 값이 객체라면 객체는 병합된다.

주의: 여러분들이 JSON이 중복된 키에 대해 이런 행동을 가정할 수는 있겠지만 HOCON은 JSON의 상위 집합이 아니다. 중복된 키에 이런 가정은 유효하지 않은 JSON이 된다.

객체 병합을 위해

 - 현재 필드에 오직 두개의 객체 중 하나를 병합될 객체로 추가하자.
 - 각 객체에 객체값이 아닌 필드를 위해 필드는 두 번째 객체에 반드시 있어야 한다
 - 각 객체에 객체값인 필드를 위해 객체 값은 동일한 규칙을 따라서 순회적으로 병합되어진다.

객체 병합은 다른 값의 키의 설정에 의해 방해를 받을 수 있다. 왜냐하면 병합은 항상 2개의 값에서 이루어지기 때문이다. 만약에 객체나 객체가 아닌 것을 키로 설정한다면 우선 객체가 아닌 것은 객체로 떨어지며(객체가 아닌것이 항상 이긴다), 객체는 객체가 아닌 것으로 떨어진다(병합되지 않고, 객체는 새로운 값이다.) 그래서 두개의 객체는 서로를 볼 수 없다.

다음 두개는 동등하다.

    {
        "foo" : { "a" : 42 },
        "foo" : { "b" : 43 }
    }

    {
        "foo" : { "a" : 42, "b" : 43 }
    }

그리고 다음 두개 역시 동등하다.

    {
        "foo" : { "a" : 42 },
        "foo" : null,
        "foo" : { "b" : 43 }
    }

    {
        "foo" : { "b" : 43 }
    }

`"foo"`의 중간 설정에 `null`은 객체의 병합을 방해한다.

### 키의 경로

키가 다중 요소로 경로 표현이라면 각 경로 요소의 나머지 부분을 객체를 생성하도록 확장된다. 나머지 경로 요소는 값으로 병합되고 가장 많이 중첩된 객체에서 필드가 된다.

이 구문과

    foo.bar : 42

다음 구문은 동일하다.

    foo { bar : 42 }

그리고 이 구문과

    foo.bar.baz : 42

다음 구문은 동일하다.

    foo { bar { baz : 42 } }

다음의 값은 일반적인 방법으로 병합된다. 이 구문과

    a.x : 42, a.y : 43

아래 구문은 동일하다.

    a { x : 42, y : 43 }

값 연결처럼 경로 표현이 동작하기 때문에 키에 공백을 가질 수 있다. 이 구문과

    a b c : 42

아래의 구문은 동일하다.

    "a b c" : 42

경로 표현은 항상 문자열로 변환되기 때문에 심지어 단일 값도 일반적으로 문자열이 된 다른 타입이 된다.

   - `true : 42` is `"true" : 42`
   - `3.14 : 42` is `"3.14" : 42`

특별한 규칙으로 따옴표가 없는 문자열 `include`는 키로 경로 표현식이 되지 않는다. 왜냐하면 특별한 인터프리테이션이기 때문이다.

## 대체하기

Substitutions are a way of referring to other parts of the configuration tree.

The syntax is `${pathexpression}` or `${?pathexpression}` where the `pathexpression` is a path expression as described above. This path expression has the same syntax that you could use for an object key.

The `?` in `${?pathexpression}` must not have whitespace before it; the three characters `${?` must be exactly like that, grouped together.

For substitutions which are not found in the configuration tree, implementations may try to resolve them by looking at system environment variables or other external sources of configuration. (More detail on environment variables in a later section.)

Substitutions are not parsed inside quoted strings. To get a string containing a substitution, you must use value concatenation with the substitution in the unquoted portion:

    key : ${animal.favorite} is my favorite animal

Or you could quote the non-substitution portion:

    key : ${animal.favorite}" is my favorite animal"

Substitutions are resolved by looking up the path in the configuration. The path begins with the root configuration object, i.e. it is "absolute" rather than "relative."

Substitution processing is performed as the last parsing step, so a substitution can look forward in the configuration. If a configuration consists of multiple files, it may even end up retrieving a value from another file. If a key has been specified more than once, the substitution will always evaluate to its latest-assigned value (the merged object or the last non-object value that was set).

If a configuration sets a value to `null` then it should not be looked up in the external source. Unfortunately there is no way to "undo" this in a later configuration file; if you have `{ "HOME" : null }` in a root object, then `${HOME}` will never look at the environment variable. There is no equivalent to JavaScript's `delete` operation in other words.

If a substitution does not match any value present in the configuration and is not resolved by an external source, then it is undefined. An undefined substitution with the `${foo}` syntax is invalid and should generate an error.

If a substitution with the `${?foo}` syntax is undefined:

 - if it is the value of an object field then the field should not
   be created. If the field would have overridden a previously-set
   value for the same field, then the previous value remains.
 - if it is an array element then the element should not be added.
 - if it is part of a value concatenation then it should become an
   empty string.
 - `foo : ${?bar}` would avoid creating field `foo` if `bar` is
   undefined, but `foo : ${?bar} ${?baz}` would be a value
   concatenation so if `bar` or `baz` are not defined, the result
   is an empty string.

Substitutions are only allowed in object field values and array elements (value concatenations), they are not allowed in keys or nested inside other substitutions (path expressions).

A substitution is replaced with any value type (number, object, string, array, true, false, null). If the substitution is the only part of a value, then the type is preserved. Otherwise, it is value-concatenated to form a string.

Circular substitutions are invalid and should generate an error.

Implementations must take care, however, to allow objects to refer to paths within themselves. For example, this must work:

    bar : { foo : 42,
            baz : ${bar.foo}
          }

Here, if an implementation resolved all substitutions in `bar` as part of resolving the substitution `${bar.foo}`, there would be a cycle. The implementation must only resolve the `foo` field in `bar`, rather than recursing the entire `bar` object.

## 포함하기

### Include 문법

_include 구문_은 따옴표가 없는 `include` 포함하여 다음에 바로 나오는 단일 따옴표 문자열이 따라온다. include 구문은 객체의 필드에 나타날 수 있다.

만약 따옴표가 없는 문자열 `include`가 객체 키가 예성되는 곳에 경로 표현식의 시작에 나타나면 키나 경로 표현식으로 인식되지 않는다.

대신 다음 값은 _따옴표가 있는_ 문자열이 되어야 한다. 따옴표가 있는 문자열은 파일 이름이나 포함되어진 리소스의 이름으로 인식된다.

더불어, 따옴표가 없는 `include`나 따옴표가 있는 문자열은 구문상으로 객체의 필드를 대신하며, 따라오는 객체 필드나 일반적인 콤마로 포함된 것으로 구분된다.(그리고 일반적으로 콤마는 새로운 라인이라면 생략할 수 있다.)

만약 따옴표가 없는 `include`가 키의 시작에 있다면 다음으로 단일 따옴표 문자열으로 다른 것이 따라 온다면 이는 유효하지 않으며 오류가 발생할 것이다.

There can be any amount of whitespace, including newlines, between the unquoted `include` and the quoted string.

따옴표가 없는 `include`와 따옴표가 있는 문자열 사이에 새로운 라인을 포함하여 다수의 공백이 올 수 있다.

값 연결은 "인자"에서 `include`로 수행될 수 없다. 인자는 단일 따옴표 문자열이어야 한다. 대체는 없고 인자는 따옴표가 없는 문자열이거나 다른 종류의 값일 수 없다.

만약 키의 경로 표현식의 시작이 아니라면 따옴표가 없는 `include`는 특별한 의미를 갖는다.

키에서 나머지에 나타날 수 있다.

    # 이는 유효하다
    { foo include : 42 }
    # 위와 아래는 동일하다.
    { "foo include" : 42 }

배열의 값이나 객체로 나타날 수 있다.

    { foo : include } # 값은 문자열 "include"이다.
    [ include ]       # 배열의 하나의 문자열이 "include"다.

키가 `"include"`로 시작하는 키를 원하면 따옴표가 있는 `"include"`일 수 있고, 오직 따옴표가 없는 `include`만 특별하다.

    { "include" : 42 }

### Include 기호의 의미: 병합

_including 파일_은 include 구문을 포함하고 _included 파일_은 include 구문의 특별한 경우다. (이들은 파일 시스템의 정규 파일이 필요하지 않지만 그들이 잠시 있다고 가정하자)

포함된 파일은 객체를 포함하여야 하며 배열은 안된다. 이들은 JSON이나 HOCON 둘다 문서의 최상위 값으로 배열을 허용하기 때문에 중요하다.

만약 포함된 파일 최상위 값으로 배열을 포함하면 이는 유효하지 않으며 오류가 발생할 것이다.

포함된 파일은 최상위 객체를 생성하는 것으로 해석되어야 한다. 최상위 객체로 부터 키는 개념적으로 파일을 포함하는 것으로 include 구문으로 대체된다.

 - 만약 객체를 포함하는 include 구문 이전에 객체를 포함하는 것이 발견된 키는
   정확하게 단일 파일에서 중복된 키가 발견되면 포함된 키의 값은 이전 값으로
   오버라이드되거나 머지된다.

 - 이전에 객체 포함하기로부터 키가 반복되는 파일 포함은,
   포함한 파일의 값은 포함한 파일으로부터 하나로 머지 되거나
   오버라이드 된다.

### Include 기호의 의미: 대체

포함된 파일에서 대체는 2가지의 경우로 볼 수 있다. 첫 번째는 포함된 파일의 최상위와 상대적이다. 두 번째는 포함된 환경설정의 최상위와 상대적이다.

마지막 단계로 대체가 발생하여 다시 호출되는 것은 _after_ 분석이다. 이는전체 앱의 환경설을 위해 수행되며 고립된 단일 파일을 위한 것이 아니다.

그러므로 포함된 파일이 대체를 포함한다면, 그들은 앱의 환경설정 최상단으로 상대적으로 "수정 되어야 할"필요가 있다.

여기 최 상단 환경설정의 예제가 있다.

    { a : { include "foo.conf" } }

"foo.conf"는 다음과 같을 수 있다.

    { x : 10, y : ${x} }

만약 고립에서 "foo.conf"를 분석하면 `x` 경로의 값이 10이므로 `${x}`는 10으로 평가된다. 키 `a`에서 객체에게 "foo.conf"를 포함하면 `${x}`대신 `${a.x}`로 수정될 것이다.

여기 최상단 환경설정의 `a.x`로 다시 정의된 것이 있다.

    {
        a : { include "foo.conf" }
        a : { x : 42 }
    }

"foo.conf"에서 `${x}`는 이미 `${a.x}`로 수정되었기 때문에 `10`대신 `42`로 평가된다. 대체는 전체 환경설정이 분석된 _이후_에 발생한다.

그러나 애플리케이션의 최상위 환경을 참조할 의도로 파일을 포함하는 수 많은 경우가 있다. 예를들어 참조된 환경설정에서나 시스템 프로퍼티로부터 값을 얻는 것이다. 그래서 경로를 "수정한다"는 것을 살펴본 것으로 충분하지 않다. 본래의 경로 또한 수정되는 것을 살펴보는 것이 필요하다.

### Include 기호의 의미: 파일 무시하기

만약 포함된 파일이 존재하지 않으면 include 구문은 이를 조용하게 무시한다.(만약 포함된 파일이 오직 비어있는 객체를 포함한다면)

### Include 기호의 의미: 리소스의 위치 나타내기

개념적으로 말해서, inclue 구문에서 따옴표가 있는 문자열은 파일을 식별하거나 분석될 "인접한" 다른 리소스와 분석될 것과 동일한 타입의 것을 식별한다. "인접한"의 의미는 문자열 스스로 각 리소스의 종류를 위해 하나씩 명확하게 되어야 한다.

구현은 그들이 지원하는 리소스의 수 많은 종류로 다양할 것이다.

자바 가상 머신에서 include 구문은 어떤 "인접한" 포함할 리소스를 확인할 수 없다면 구현은 클래스 패스 리소스를 차선책으로 해야 하며 이는 환경설정이 파일이나 URL을 클래스 패스 리소스로 접근하는 것을 허용한다.

자바의 클래스패스에서 지정된 리소스는 다음과 같다.

 - included resources are looked up by calling `getResource()` on
   the same class loader used to look up the including resource.
 - if the included resource name is absolute (starts with '/')
   then it should be passed to `getResource()` with the '/'
   removed.
 - if the included resource name does not start with '/' then it
   should have the "directory" of the including resource.
   prepended to it, before passing it to `getResource()`.  If the
   including resource is not absolute (no '/') and has no "parent
   directory" (is just a single path element), then the included
   relative resource name should be left as-is.
 - it would be wrong to use `getResource()` to get a URL and then
   locate the included name relative to that URL, because a class
   loader is not required to have a one-to-one mapping between
   paths in its URLs and the paths it handles in `getResource()`.
   In other words, the "adjacent to" computation should be done
   on the resource name not on the resource's URL.

파일 시스템에서 일반적인 파일은 다음과 같다.

 - if the included file is an absolute path then it should be kept
   absolute and loaded as such.
 - if the included file is a relative path, then it should be
   located relative to the directory containing the including
   file.  The current working directory of the process parsing a
   file must NOT be used when interpreting included paths.
 - if the file is not found, fall back to the classpath resource.
   The classpath resource should not have any package name added
   in front, it should be relative to the "root"; which means any
   leading "/" should just be removed (absolute is the same as
   relative since it's root-relative). The "/" is handled for
   consistency with including resources from inside other
   classpath resources, where the resource name may not be
   root-relative and "/" allows specifying relative to root.

URL은 다음과 같다.

 - for both filesystem files and Java resources, if the
   included name is a URL (begins with a protocol), it would
   be reasonable behavior to try to load the URL rather than
   treating the name as a filename or resource name.
 - for files loaded from a URL, "adjacent to" should be based
   on parsing the URL's path component, replacing the last
   path element with the included name.
 - file: URLs should behave in exactly the same way as a plain
   filename

## 기간 형식

기간을 위해 지원되는 유닛 문자열은 대소문자를 구분하고 소문자이어야 한다. 정확하게 이런 문자열을 지원한다.

 - `ns`, `nanosecond`, `nanoseconds`
 - `us`, `microsecond`, `microseconds`
 - `ms`, `millisecond`, `milliseconds`
 - `s`, `second`, `seconds`
 - `m`, `minute`, `minutes`
 - `h`, `hour`, `hours`
 - `d`, `day`, `days`

## 바이트 형식의 크기

단일 바이트를 위해 정확하게 이런 문자열을 지원한다.

 - `B`, `b`, `byte`, `bytes`

10의 제곱을 위해 정확하게 이런 문자열을 지원한다.

 - `kB`, `kilobyte`, `kilobytes`
 - `MB`, `megabyte`, `megabytes`
 - `GB`, `gigabyte`, `gigabytes`
 - `TB`, `terabyte`, `terabytes`
 - `PB`, `petabyte`, `petabytes`
 - `EB`, `exabyte`, `exabytes`
 - `ZB`, `zettabyte`, `zettabytes`
 - `YB`, `yottabyte`, `yottabytes`

2의 제곱을 위해 정확하게 이런 문자열을 지원한다.

 - `K`, `k`, `Ki`, `KiB`, `kibibyte`, `kibibytes`
 - `M`, `m`, `Mi`, `MiB`, `mebibyte`, `mebibytes`
 - `G`, `g`, `Gi`, `GiB`, `gibibyte`, `gibibytes`
 - `T`, `t`, `Ti`, `TiB`, `tebibyte`, `tebibytes`
 - `P`, `p`, `Pi`, `PiB`, `pebibyte`, `pebibytes`
 - `E`, `e`, `Ei`, `EiB`, `exbibyte`, `exbibytes`
 - `Z`, `z`, `Zi`, `ZiB`, `zebibyte`, `zebibytes`
 - `Y`, `y`, `Yi`, `YiB`, `yobibyte`, `yobibytes`

## 시스템 프로퍼티로 오버라이드

자바 시스템 프로퍼티는 `application.conf`와 `reference.conf` 파일에서 발견한 설정으로 오버라이드한다. 이는 명령어 라인의 명시적인 설정옵션을 지원한다. 예를들어 `play -Dkey=value run`와 같다.

주의 : 플레이는 테스트를 위해 JVM을 포크한다. - 그리고 그래서 테스트에서 명령어 라인 오버라이드를 사용하기 위해 테스트를 위해 그들을 사용하기 전에 `build.sbt`에 `Keys.fork in Test := false`를 추가해야 한다.
