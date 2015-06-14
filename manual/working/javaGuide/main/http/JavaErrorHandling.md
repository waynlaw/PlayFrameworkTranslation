# Error 핸들링

HTTP 애플리케이션이 리턴할 수 있는 2가지 메인 유형의 에러가 있다 - 클라이언트 에러와 서버 에러이다. 클라이언트 에러는 클라이언트가 무언가 잘못 실행하는 가운데 접속하는 경우를 뜻한다. 서버 에러의 경우 서버에 이상이 있다는 것을 뜻한다.

Play는 다양한 환경 속에서 자동으로 클라이언트 에러를 감지한다 - 예를 들어 잘못된 헤더 값과 같은 경우를 포함하여 지원하지 않는 content 타입, 찾을 수 없는 자원에 대한 요청들이 그러하다. Play는 또한 다양한 환경 속에서 자동으로 서버 에러를 감지한다 - action 코드에서 예외가 발생하는 경우 Play는 에러를 catch하여 클라이언트에게 전송할 서버 에러 페이지를 생성할 것이다.

Play에서 이런 에러를 제어할 수 있는 인터페이스는 [`HttpErrorHandler`](api/java/play/http/HttpErrorHandler.html) 이다. 해당 인터페이스는 `onClientError`, 와 `onServerError` 2가지 메소드를 정의하고 있다.

## 커스텀 에러 핸들러 제공하기

루트 패키지에서 [`HttpErrorHandler`](api/java/play/http/HttpErrorHandler.html)를 구현한 `ErrorHandler` 클래스를 생성하여 커스텀 에러 핸들러를 제공할 수 있다. 예를 들어:

@[root](code/javaguide/http/root/ErrorHandler.java)

루트 페이지에 에러 핸들러를 두고 싶지 않거나, 서로 다른 환경일 때 다른 에러 핸들러를 설정하고 싶을때는, `application.conf`에서 `play.http.errorHandler` 값을 설정하여 변경이 가능하다:

    play.http.errorHandler = "com.example.ErrorHandler"

## 디폴트 에러 핸들러 확장하기

별도의 설치나 설정 없이, Play에서는 많은 유용한 기능을 가진 에러 핸들러를 제공하고 있다. 예를 들어 개발(dev) 모드에서 서버에러가 발생하는 경우, Play는 해당 위치를 탐색하고, 예외가 발생한 애플리케이션 코드 조각을 생성해준다. 그래서 빠르게 확인 후, 원인이 무엇인지 찾을 수 있도록 해준다. 개발하는 도중 기존 기능은 유지하면서도 production 단계에서 커스텀 서버 에러를 사용하고 싶을 수도 있다. 이를 편리하게 하기 위해서 Play는 [`DefaultHttpErrorHandler`](api/java/play/http/DefaultHttpErrorHandler.html)를 제공한다. 이 핸들러를 오버라이딩 하면 Play에 존재하는 기능을 이용하면서도 커스텀 로직을 섞어 사용하는 것이 가능하다.

예를 들어,  production 단계에서, 개발 에러메시지는 접근을 못한 채로 남겨둔 채 커스텀 서버 에러만 제공하고 싶은 경우, 특정 forbidden 페이지를 제공하고 싶은 경우가 있다:

@[default](code/javaguide/http/def/ErrorHandler.java)

[`DefaultHttpErrorHandler`](api/java/play/http/DefaultHttpErrorHandler.html)에 대해 어떤 메서드를 오버라이드 가능한지, 혹은 어떤 장점을 취할 수 있는지 알고 싶다면 전체 API 문서를 참조하도록 한다.