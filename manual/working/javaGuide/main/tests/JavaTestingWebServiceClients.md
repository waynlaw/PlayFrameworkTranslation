<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 웹 서비스 클라이언트 테스트하기

웹 서비스 클라이언트에는 많은 코드가 삽입될 수 있다 - request를 준비하고, body를 직렬화하거나 역직렬화하거나, 올바른 header를 작성하는 작업등이 필요하다. 많은 작업들이 스트링과 약한 타입인 맵을 통해 이루어지기 때문에 테스트 작업이 더욱 중요하다. 하지만 이를 테스트하는 것은 일종의 도전이기도 하다. 일반적인 접근 방법은 다음과 같다:

### 실제 웹 서비스에 대한 테스트

실제 테스트를 통해 클라이언트 코드에서 높은 신뢰도를 얻을 수는 있지만, 사실 실용적이지는 않다. 만약 서드 파티(third party) 웹 애플리케이션의 경우, 테스트 실행을 제한하는 환경 제한이 일정 비율 있을 수 밖에 없다. (모범적인 인터넷 사용자를 염두해 두지 않은 서드 파티 웹을 자동으로 테스트를 실행해야하는 경우도 마찬가지이다) 서비스상에서 테스트에 필요한 데이터를 설정 하거나, 반드시 존재하도록 보장한다는 것은 거의 불가능 하다. 그리고 테스트 중 예상치 못한 사이드 이펙트가 발생할 수 있다. 

### 웹서비스의 테스트 인스턴스를 테스트

이전의 작업보다는 더 낫다. 하지만 아직도 몇몇 문제점들이 있다. 많은 서드 파티 웹 서비스는 테스트 인스턴스를 제공하지 않는다. 또한 테스트가 진행중인 테스트 인스턴스에 테스트 의존성이 생기게 되는데, 이 말은 테스트 서비스가 빌드 실패를 유발할 수도 있다는 뜻이 된다. 만약 테스트 인스턴스가 방화벽 뒤에 있다면, 테스트가 실행되는 위치에도 제약 사항이 생긴다.

### Http 클라이언트 흉내내기(Mock)

이 접근 방법은 테스트 코드에서 최소한 보장을 해주는 방법이다 - 이런 종류의 테스트는 따로 값이 존재하지 않는 상태로 코드가 무엇을 하는지만 보여주는 의미 이상이 되지 않는다. 웹 서비스 클라이언트를 흉내내는 테스트는 코드를 실행하고 특정 행동을 시행한다. 하지만 그 코드가 실제로 HTTP request가 실행될 때 올바르게 상응하는지는 확신을 가질 수 없다.

### 웹 서비스 흉내내기(Mock)

이 접근 방법은 실제 웹 서비스와 웹 클리이언트를 흉내내는 것 사이에서 좋은 타협점이 될 수 있다. 테스트에서 모든 유효한 HTTP 요청을 보여주고, 이 요청들의 body는 직렬화/역직렬화 등의 작업을 거치게 된다. 하지만 전적으로 내부에만 국한된 것이며, 어떤 서드파티 서비스에도 의존적이지 않다.

플레이는 테스트에서 웹 서비스를 흉내내기 위한 헬퍼 유틸리티를 제공하고 있다. 이러한 접근 방법은 테스트에서 매우 실용적이고 매력적인 옵션을 제공한다. 

## GitHub 클라이언트 테스트하기

예를들어, 당신이 GitHub 클라이언트를 사용한다고 해보자. 이것을 테스트하고 싶을 것이다. 클라이언트는 매우 간단하다. 아래 코드에서는 공용 리파지토리의 이름을 찾는 기능을 제공한다:

@[client](code/javaguide/tests/GitHubClient.java)

이 코드에서 GitHub API 기반 URL을 파라메터로 받아온다는 점을 주의하자 - 테스트 코드 작성 시, 목(mock) 서버를 바라보도록 오버라이딩 하게 될 것이다.

테스트를 위해,  엔드포인트 역할을 할 임베디드 플레이 서버가 필요하다. [[라우팅 DSL|JavaRoutingDsl]] 을 이용해 [[임베디드 서버 생성하기|JavaEmbeddingPlay]]가 가능하다:

@[mock-service](code/javaguide/tests/JavaTestingWebServiceClients.java)

이 서버는 이제 임의의 포트로 시행된다, 따라서 `httpPort` 메소드를 통해서 접근을 해야한다. 해당 메소드를 이용하여 `GitHubClient`를 통과하게 될 기본 URL을 만들 수 있다. 하지만 플레이는 더 간단한 구조를 가지고 있다. [`WS`](api/java/play/libs/ws/WS.java) 클래스는 포트 번호를 파라미터로 넘기는 `newClient` 메소드를 제공하고 있다. 상대경로 URL로 클라이언트(예를 들어 `/repositories`로)를 사용하여 요청을 만들 때, 이 클라이언트는 해당 포트를 통과할 로컬호스트로 요청을 보내게 된다. 이 말은 우리가 `GitHubClient`에서 기본 URL을 `""`로 설정할 수 있다는 의미이다. 또한 다음 요청에서 사용하게 될 다른 리소스에 대한 URL 링크와 리소스를 클라이언트가 반환하게 된다면, 상대경로 URL을 확인한 후 (해당 경로) 그대로 사용하기만 하면 된다는 의미가 된다.

이제 서버를 만들 수 있다. `@Before` 애노테이션 메소드에 WS 클라이언트와 `GitHubClient`를 만들고, `@After` 애노테이션 메소드에서 서버를 내리도록 하자. 그리고 테스트에서 클라이언트를 테스트할 수 있다:

@[content](code/javaguide/tests/GitHubClientTest.java)

### 파일 반환하기

이전 예시에서 서비스를 흉내내기 위해 json을 만들었다. 서비스에서 테스트를 할때 실제 응답을 확인하고 응답을 넘기는 것이 훨씬 수월해지기 때문이다. 이를 지원하기 위해 플레이는 `sendResource` 메소드를 제공하여 클래스패스 파일에서 쉽게 결과를 생성할 수 있도록 지원해준다.

실제 GitHub API에서 요청을 생성한 후, 테스트 리소스 디렉토리에 파일을 만들어 저장한다. 테스트 리소스 디렉토리는 `test/resources` 이거나, 플레이 디렉토리 형식이나 표준 sbt 디렉토리 형식을 사용한다면 `src/test/resources` 이다. 이 경우 `github/repositories.json`라는 파일에 아래의 내용을 포함할 것이다:

```json
[
  {
    "id": 1296269,
    "owner": {
      "login": "octocat",
      "id": 1,
      "avatar_url": "https://github.com/images/error/octocat_happy.gif",
      "gravatar_id": "",
      "url": "https://api.github.com/users/octocat",
      "html_url": "https://github.com/octocat",
      "followers_url": "https://api.github.com/users/octocat/followers",
      "following_url": "https://api.github.com/users/octocat/following{/other_user}",
      "gists_url": "https://api.github.com/users/octocat/gists{/gist_id}",
      "starred_url": "https://api.github.com/users/octocat/starred{/owner}{/repo}",
      "subscriptions_url": "https://api.github.com/users/octocat/subscriptions",
      "organizations_url": "https://api.github.com/users/octocat/orgs",
      "repos_url": "https://api.github.com/users/octocat/repos",
      "events_url": "https://api.github.com/users/octocat/events{/privacy}",
      "received_events_url": "https://api.github.com/users/octocat/received_events",
      "type": "User",
      "site_admin": false
    },
    "name": "Hello-World",
    "full_name": "octocat/Hello-World",
    "description": "This your first repo!",
    "private": false,
    "fork": false,
    "url": "https://api.github.com/repos/octocat/Hello-World",
    "html_url": "https://github.com/octocat/Hello-World"
  }
]
```

테스트에 필요한 입맛대로 수정을 하고 싶을 수도 있다. 예를 들어 GitHub 클라이언트가 위 응답에 있는 URL을 이용해 다른 앤드포인트로 요청을 보내고 싶은 경우, `https://api.github.com` 접두어를 지워 상대경로로 이용하고 싶을 수도 있다. 상대경로로 지정이 되면, 테스트 클라이언트를 통해 로컬호스트로 자동 라우팅 될 것이기 때문이다.

이제 아래 리소스를 제공하기 위해 라우터를 수정하자:

@[send-resource](code/javaguide/tests/JavaTestingWebServiceClients.java)

플레이는 자동으로 `application/json` 컨텐트 타입을 설정한다는 점을 주의하자. 파일 확장자가 `.json`이기 때문이다.