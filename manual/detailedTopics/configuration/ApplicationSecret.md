# 애플리케이션 비밀

플레이는 다음과 같은 상황을 위해 비밀 키를 사용한다

* 세션 쿠키나 CSRF 토큰 서명
* 암호화 유틸리티 내장

이는 `application.conf`에서 `play.crypto.secret` 프로퍼티 이름으로 환경설정할 수 있으며 기본은 `changeme`이다. 기본에서 알 수 있듯이 프로덕션에서는 변경되어야 한다.

> prod 모드로 시작할 때, 플레이가 비밀이 설정되지 않음을 발견하거나 `changeme`로 설정되어 있으면 플레이는 오류를 던지게 된다.

## 모범 사례

비밀에 대한 접근을 얻을 수 있는 사람은 효과적으로 어떤 사용자가 시스템으로 로그인하도록 허용하기 위해 그들이 원하면 어떤 세션을 생성할 수 있을 것이다. 따라서 소스 컨트롤으로 여러분의 애플리케이션 비밀을 확인할 수 없도록 하는 것이 좋다. 대신 프로덕션 서버에서 환경설정을 할 수 있다. 이는 `application.conf`에 프로덕션 애플리케이션 비밀을 넣는 것이 나쁜 관행으로 간주된다는 것을 의미한다.

프로덕션 서버에서 애플리케이션 비밀을 환경설정하는 한 가지 방법은 시작 스크립트에 시스템 프로퍼티로 전달하는 것이다. 예를 들어 다음과 같다.

```bash
/path/to/yourapp/bin/yourapp -Dplay.crypto.secret="QCY?tAnfk?aZ?iwrNwnxIlR6CTf:G3gf:90Latabg@5241AB`R5W:1uDFN];Ik@n"
```

이 방법은 매우 간단하며, 애플리케이션 비밀이 설정될 필요가 있다는 것을 암시하여 프로덕션 모드로 여러분의 앱을 실행하기 위해 플레이 문서에 있는 방법을 사용할 것이다. 그러나 일부 환경에서는 명령어 라인의 인자로 비밀을 배치하는 것은 좋지 않은 사례로 간주 될 수 있다.
이 문제를 해결하는 2가지 방법이 있다.

### 환경 변수

첫 번째는 환경 변수로 애플리케이션 비밀을 배치하는 것이다. 이런 경우에는 `application.conf` 파일에 다음과 같이 환경설정을 배치하는 것을 추천한다.

    play.crypto.secret="changeme"
    play.crypto.secret=${?APPLICATION_SECRET}

두 번째 라인은 만약 `APPLICATION_SECRET`으로 불리는 환경변수가 설정되어 있으면 그 환경 변수값으로 비밀을 설정하고 반면에 설정되어 있지 않으면 이전 값을 변경하지 못할 것이다.

이 방법의 일반적인 사례는 클라우드 프로바이더를 위한 API를 통해 환경설정 될 수 있는 환경변수를 통해 다른 비밀과 패스워드를 설정하는 경우처럼 특히 클라우드 기반 배포 시나리오에서 잘 동작한다.

### 프로덕션 환경설정 파일

다른 방법은 애플리케이션의 비밀과 패스워드처럼 영향을 받기 쉬운 환경설정을 오버라이드하는 것으로 `application.conf`을 포함시키는 서버에만 존재하는 `production.conf`파일을 생성하는 방법이다.

예를 들면 다음과 같다.

    include "application"

    play.crypto.secret="QCY?tAnfk?aZ?iwrNwnxIlR6CTf:G3gf:90Latabg@5241AB`R5W:1uDFN];Ik@n"

다음과 같이 플레이를 시작할 수 있다.

```bash
/path/to/yourapp/bin/yourapp -Dconfig.file=/path/to/production.conf
```

## 애플리케이션 비밀 생성하기

플레이는 새로운 비밀을 생성하기 위해 사용할 수 있는 유틸리티를 제공한다. 플레이 콘솔에서 `play-generate-secret`를 수행하자. 이 명령어는 다음과 같이 여러분의 애플리케이션에 사용할 수 있는 새로운 비밀을 생성할 것이다.

```
[my-first-app] $ playGenerateSecret
[info] Generated new secret: QCYtAnfkaZiwrNwnxIlR6CTfG3gf90Latabg5241ABR5W1uDFNIkn
[success] Total time: 0 s, completed 28/03/2014 2:26:09 PM
```

## application.conf에 애플리케이션 비밀 갱신하기

플레이는 `application.conf`에 비밀을 갱신하기 위한 편리한 유틸리티를 제공하며, 이 명령어는 테스트 서버나 개발 서버를 위해 특별한 비밀을 설정하고 싶을 때 사용할 수 있다. 이는 애플리케이션 비밀을 사용하여 데이터를 암호화 하거나 동일한 비밀이 개발 모드에서 애플리케이션이 실행될 때 모든 순간에 사용된다는 것을 확실하고 싶을 때 유용하다.

`application.conf`에 비밀을 갱신하기 위해, 플레이 콘솔에서 `play-update-secret`를 수행하자.

```
[my-first-app] $ playUpdateSecret
[info] Generated new secret: B4FvQWnTp718vr6AHyvdGlrHBGNcvuM4y3jUeRCgXxIwBZIbt
[info] Updating application secret in /Users/jroper/tmp/my-first-app/conf/application.conf
[info] Replacing old application secret: play.crypto.secret="changeme"
[success] Total time: 0 s, completed 28/03/2014 2:36:54 PM
```
