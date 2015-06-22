<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 웹서비스 클라이언트 테스트하기

요청을 준비하고, 내용을 직렬화/역직렬화를 하며, 올바른 헤더를 설정 하는 등의 많은 코드들이 웹서비스 클라이언트를 작성하는데 사용될 수 있다.
 이런 많은 코드들은 문자열, 약한 타입의 맵을 이용하기 때문에 코드를 테스트하는 것은 매우 중요하다. 그러나 현재는 테스트를 하는 것 자체도 해결해야 할 몇 가지 문제가 있다. 몇 가지 일반적인 해결 방법을 보자.

### 실제 웹 서비스에 대해서 테스트 하기

클라이언트 코드 내에서 가장 높은 신뢰성을 가지는 테스트이다. 하지만 일반적으로 이루어지지는 않는다. 써드파티 웹 서비스라면 실행 중에 테스트를 하는 것을 방지하는 몇 가지 제약이 있을 것이다. (그리고 자동화된 테스트를 써드파티 서비스에 대해 하는 것은 좋은 네티즌으로 보여지지 않을 것이다.) 서비스에서 필요로 하는 테스트 데이터가 존재하지 않거나 설정할 수 없을 수도 있으며, 테스트가 서비스에 예기치 못한 부작용을 만들어낼 수 있다.

### 웹 서비스의 테스트 인스턴스에 대해 테스트하기

이것은 이전의 방법에 비해 조금 더 나은 방법이지만, 여전히 몇 가지 문제가 있다. 많은 써드파티 웹 서비스들은 테스트 인스턴스를 제공하지 않는다. 또한 이 방법은 테스트들이 테스트 인스턴스가 실행중인지 유무에 의존성을 가지도록 만들기 때문에, 테스트 서비스가 빌드를 실패하게 만들수도 있다. 그리고 테스트 인스턴스가 방화벽뒤에 있다면, 이 또한 테스트에 몇가지 제약사항을 만들 것이다.

### 가상 http클라이언트 만들기

이것은 테스트 코드에 대해 가장 낮은 신뢰성을 보장한다. 종종 이런 종류의 테스트들은 코드가 하는 것 자체를 그대로 테스트 하게되어 아무런 가치가 없는 결과를 만들기도 한다. 가상 웹 서비스 클라이언트들에 대한 테스트들은 확실하게 실행되지만 실제 올바른 HTTP 요청들이 만들어졌는지에 대해서 보장을 해주지 않는다.

### 가상 웹서비스 만들기

이 방법은 실제 웹 서비스를 테스트하는 것과 가상의 http 클라이언트를 만드는 것 사이의 절충안이다. 테스트들은 모든 요청들에 대해서 내용을 직렬화/역직렬화 하거나 다른 여러가지 작업을 하는 과정에서 올바른 HTTP 요청들이 만들어지는지에 대해서 보여줄 것이다. 하지만 완전히 그 자체로만 동작하며, 다른 어떤 써드파티 서비스들에 대해 의존성을 가지지는 않는다.

플레이는 테스트에서 웹 서비스를 가상으로 만들기 위해 몇가지 도움을 주는 기능을 제공한다. 이 방법은  테스트가 더 좋아질 수 있는 매력적인 선택이다.

## GitHub 클라이언트 테스트하기

예를 들어, GitHub클라이언트를 만들었다고 가정하고 그것을 테스트하길 원한다고 해보자. 이 클라이언트는 매우 간단하며, 단순하게 공개된 저장소들의 이름들을 보여주는 기능만을 가지고 있다.

@[client](code/webservice/ScalaTestingWebServiceClients.scala)

이 기능은 URL 기반의 GitHub API를 파라메터로 사용한다는 점에 주의해야 한다. 이 부분이 우리가 테스트에서 재정의 하여 우리의 가상 서버로 연결할 수 있는 지점입니다.

이것을 테스트할 떄, 이 접속지를 구현하기 위해서 플레이 서버를 내장하기를 원할 수 있다. [`Server`](api/scala/index.html#play.core.server.Server) `withRouter` 도우미를 [[Routing DSL에 문자열 넣기|ScalaSirdRouter]]와 사용하여 만들 수 있다.

@[mock-service](code/webservice/ScalaTestingWebServiceClients.scala)

`withRouter` 함수는 서비스가 시작할 포트 번호를 받는 코드 블럭을 받는다. 기본적으로 플레이는 사용할 수 있는 포트 중에의 임의의 포트로 시작한다. 그렇기 때문에 빌드서버에서의 자원 충돌이나 테스트를 위한 포트 할당에 대해 고민할 필요가 없다. 하지만 어떤 포트가 사용되고 있는지에 대해 알려주어야 할 필요는 있다.

이제 GitHub 클라이언트를 테스트 하려면 `WSClient`를 필요로 한다. 플레이는 테스트 클라이언트들을 만드는데 사용하기위한 펙토리 함수로 [`WsTestClient`](api/scala/index.html#play.api.test.WsTestClient$) 트레이트를 제공한다. `withClient`는 포트를 암묵적으로 받는데, 이런점이 `Server.withRouter`함수와 사용하기 편리하다.

`WsTestClient.withClient` 함수가 만드는 클라이언트는 특별한 클라이언트다. 만일 상대적인 URL을 주면 호스트 이름을 `localhost`로 설정할 것이며, 포트 번호는 암묵적으로 받은 포트번호를 사용할 것이다. 그렇기 때문에 이걸 사용해서 우리는 빈 문자열을 GitHub 클라이언트를 위한 기본 URL로 사용할 수 있다.

이것들을 모두 모아서 사용하면, 아래와 같다.

@[full-test](code/webservice/ScalaTestingWebServiceClients.scala)

### 파일들을 반환하기

이전의 예제에서, 가상의 서비스들을 위한 json을 수작업으로 만들어야 했다. 이는 종종 테스트 서버에서 실제의 응답을 받고 반환하는 것 보다 더 좋다. 이런 작업을 좀더 편리하게 만들기 위해 플레이는 클래스 경로에 있는 파일에서 결과를 쉽게 만들 수 있는 `sendResource` 함수를 제공한다.

실제 GitHub API에 요청을 만든 다음에, 테스트 리소스 디렉토리에 내용을 저장할 파일을 만든다. 테스트 디렉토리는 플레이 디렉토리 레이아웃을 따르는 경우에는 `test/resources`이며, 표준 sbt 레이아웃을 따르는 경우에는 `src/test/resources`이다. 이 예제에서는 `github/repositories.json`로 만들고, 아래의 내용을 담을 것이다.

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

만일 테스트의 필요에 따라 내용을 수정하려 할 수 있다. 예를 들어 만일 GitHub 클라이언트가 다른 곳으로 요청을 하도록 만들기 위해서 위의 응답의 URL을 수정하려 한다면, 상대 경로로 만들기 위해 앞에 붙은 `https://api.github.com`을 제거하려 할 것이다. 그러면 테스트 클라이언트에 의해서, 자동으로 로컬 호스트로 연결되고 올바른 포트를 사용하게 될 것이다.

이제, 이 자원을 제공하기 위해 라우터를 수정해보자.

@[send-resource](code/webservice/ScalaTestingWebServiceClients.scala)

플레이가 파일의 확장자가 `.json`으로 끝나기 때문에, 자동으로 내용물의 종류를 `application/json`로 만드는 것에 주목하자.

### 설정 코드 빼내기

만일 하나의 테스트를 돌리고자 한다면 테스트의 구현이 충분할 수 있다. 하지만 여러개의 함수들에 대해 테스트를 하고자 한다면, 가상의 클라이언트를 만들어내는 코드를 하나의 도움 함수로 빼내야 할것이다. 예를 들어 `withGitHubClient` 함수를 정의해 보자.

@[with-github-client](code/webservice/ScalaTestingWebServiceClients.scala)

그리고 그걸 쓰면 테스트는 아래와 같이 보일 것이다.

@[with-github-test](code/webservice/ScalaTestingWebServiceClients.scala)
