<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 개발 & 컨트리뷰터 가이드라인

## 일반적인 작업흐름

이것은 마스터 브랜치로 코드를 커밋하기 위한 과정이다. 이 규칙들에는 물론 마이너 변경을 위한 주석, 문서, 깨진 빌드 수정 등의 예외는 있을 수 있다.

1. [Typesafe CLA](http://www.typesafe.com/contribute/cla)에 서명했는지 확인하길 바란다. 만약 하지 않았다면 온라인 상에서 서명할 수 있다.

2. 기능이나 수정을 위한 작업을 시작하기 전에 다음을 숙지하자.
    1. 여러분의 작업이 프로젝트의 이슈 트래커에 티켓으로 있는지 확인하자. 아니라면 이를 처음으로 생성하자.
    2. 티켓은 현재 마일스톤을 위해 관리되고 있다.
    3. 티켓은 팀에 의해서 평가된다.
    4. 티켓은 팀에 의해서 토론되고 우선순위가 정해질 것이다.
3. 항상 Git의 기능 브랜치에서 작업을 수행해야 한다. 브랜치는 의도를 설명할 수 있는 서술적인 이름을 부여해야 한다.
4. 우리는 코드가 견고하게 포매팅되었는지 확신하기 위해 [scalariform](https://github.com/mdr/scalariform)을 사용한다. 이것은 스칼라 소스를 컴파일링 할 때 자동으로 동작한다.
5. 기능을 추가하거나 수정이 완료되었을 때 Github의 [풀 리퀘스트](https://help.github.com/articles/using-pull-requests)를 열어야 한다. 테스트를 통과하지 못했거나 scalariform의 포매팅을 갖지 않은 풀 리퀘스트는 자동적으로 우리의 CI에서 실패할 것이다.
6. 풀 리퀘스트는 다른 유지보수자에 의해서 가능하면 많이 리뷰되어야 한다. 유지보수자가 플레이 팀 내부의 사람 일수도 외부의 컨트리뷰터가 될수도 있음을 명심하라. 외부의 컨트리뷰터가 리뷰 과정에서 참겨하는 것이 좋으며, 막혀있는 과정이 아니다.
7. 리뷰 후에 필요하면 이슈를 수정해야 하며(새로운 리뷰등을 위해 새로운 커밋이 푸쉬될 것이다), 리뷰어들이 모두 만족할 때까지 반복될 것이다.
8. 코드가 리뷰가 완료된 즉시 풀 리퀘스트는 마스터 브랜치로 머지될 수 있다.

## 개발자 그룹 & 토론

기능을 토론하기 위해, 풀 리퀘스트나 제안을 위해 https://groups.google.com/forum/#!forum/play-framework-dev에서 전용 그룹을 사용한다.

## 풀 리퀘스트 요구사항

풀 리퀘스트가 고려되기 위해서 다음과 같은 요구사항들이 있다.

1. 현재 코드 표준이 다음에 부합해야 한다.
   - [DRY](http://programmer.97things.oreilly.com/wiki/index.php/Don%27t_Repeat_Yourself)를 위반하면 안된다.
   - [Boy Scout Rule](http://programmer.97things.oreilly.com/wiki/index.php/The_Boy_Scout_Rule) 가 적용되어 있어야 한다.
2. 코드가 새로운 기능이나 버그를 수정한 것인것과는 관계없이 광범위하게 테스트 되어야 한다.
3. 코드는 플레이 표준 문서 형식에 맞게 작성되어야 한다(아래의 ‘문서’ 섹션을 확인하라). 각 API는 문서의 변경에 맞춰 변경되어야 한다.
4. 구현에 대해서는 다음과 같은 것들은 최대한 피해야 한다.
   * Global state
   * Public mutable state
   * Implicit conversions
   * ThreadLocal
   * Locks
   * Casting
   * Introducing new, heavy external dependencies
5. 플레이 API는 다음과 같은 규칙을 따른다.
   * 플레이는 자바와 스칼라의 프레임워크다. 여러분의 변경사항이 자바, 스칼라의 API에서 동작하는지 확인하자.
   * 자바 API는 ```framework/play/src/main/java```에 있으며, 패키지 구조는 ```play.myapipackage.xxxx``` 이다.
   * 스칼라 API는 ```framework/play/src/main/scala```에 있으며, 패키지 구조 ```play.api.myapipackage``` 이다.
   * 자바나 스칼라의 API는 다음과 같은 방식으로 구현되어야 한다.
     * 핵심 API는 스칼라로 구현하자 (```play.api.xxx```)
     * 만약 여러분의 컴포넌트가 생명주기 관리가 필요하거나 스왑이 필요하다면 플러그인을 생성하거나 이 단계를 생략하라.
     * wrap core API for scala users ([example](https://github.com/playframework/playframework/blob/master/framework/src/play-cache/src/main/scala/play/api/cache/Cache.scala#L69))
     * 스칼라 사용자를 위한 핵심 API를 감싸자. ([예제](https://github.com/playframework/playframework/blob/master/framework/src/play-cache/src/main/scala/play/api/cache/Cache.scala#L69))
     * 자바 사용자를 위해 스칼라 API를 감싸자. ([example](https://github.com/playframework/playframework/blob/master/framework/src/play-cache/src/main/java/play/cache/Cache.java))
   * 기능은 영원하며, 항상 새로운 기능이 정말 핵심 프레임워크에 있어야 하는지 플러그인으로 구현되어야 하는지에 대해서 생각하자.

이 요구사항을 만족하지 못하면  좋거나 중요하거나와 상관없이 마스터에 머지될 수 없고 리뷰조차 하지 않는다.

## 문서

문서는 `documentation/manual` 디렉토리에 마크다운 페이지로 존재한다. 각 플레이 브랜치는 그에 맞는 문서 버전을 가지고 있다.

## 진행중인 작업

Github 저장소에 공개된 기능 브랜치로 작업해도 무방하다. 때로 빠른 피드백을 위해 유용할 수 있다. 그래서 브랜치의 이름을 적절하게 하는 것이 바람직하다. `진행중인 작업`으로 ``wip-``로 이름의 접두어를 주거나 ``wip/..``, ``feature/..`` or ``topic/..``와 같이 계층적인 이름을 사용할 수 있다. 어느쪽이든 작업중이거나 아직 머지가 준비되지 않았다는 것을 아주 명확하게 알 수 있으면 된다. 이 작업은 일시적으로 낮은 표준을 가질 수 있다. 그러나 마스터로 머지되기 위해 위에서 설명한 리뷰나 풀 리퀘스트등의 정규 과정을 거쳐야 할 것이다.

물론 더 나은 형식의 커밋과 작업을 수월하게 하기 위해 the ``wip``나 ``feature``/``topic`` 식별자는 특별한 의미를 갖는다. ``wip``라고 명명된 브랜치는 “git-unstable”이며 리베이스 될 수 있고, 이력이 새로 작성된 것 일수도 있다. ``feature``/``topic``라고 명명된 브랜치는 다른 그룹이 기능을 작동할 때 의존하기에 충분히 "안정"적이어야 한다.

## 커밋 만들기 및 커밋 메시지 작성하기

다음은 공개적인 커밋을 만들고 커밋 메시지를 작성할 때 가이드라인이다.

1. 작업하는 동안 여러개의 로컬 커밋을 생성했다면(예를들어 기능 브랜치에 작업하는 동안 안정된 포인트에 커밋들이거나 머지/리베이스 등 오랜기간 동안 작업한 브랜치) 모든 커밋을 하지 말고 좋은 커밋 메시지를 작성하여 하나의 큰 커밋으로 커밋을 뭉개서 이력을 새로 작성하자 (다음섹션에 설명할 것 처럼). 더 많은 정보를 얻기 위해 [Git Workflow](https://sandofsky.com/blog/git-workflow.html)를 읽어보자. 모든 커밋은 분리, 체리 피킹 등을 사용하여 할 수 있다.
2. 첫 번째 줄은 커밋이 무엇을 하는 것인지 설명하는 문장 이어야한다. 이는 한 문장을 읽는 것만으로 커밋이 하는 것이 무엇인지 완벽하게 이해할 수 있어야 한다. 이는 단순히 티켓 넘버나 "minor fix"와 같은 것이면 안된다. 티켓 넘버를 가리키기 위해 # 접두어를 사용하여 첫번째 라인 끝에 포함시켜라. 커밋이 아주 작은 변경일 경우는 여기서 끝이며, 아니라면 3번의 내용으로 가자.
3. 첫 번째 라인 설명에 이어서 커밋의 자세한 목록을 나열하기 전에 공백 한 줄을 넣어야 한다.
4. 이제 여러분의 커밋을 위해 키워드들을 추가하자.( 자동화의 정도에 따라서 우리가 도달할 수 있도록하고 목록은 시간에 따라 변경될 수 있다.)

    * ``Review by @gituser`` - 팀의 누구의 리뷰를 받았는지 명시할 수 있다. 다른 사람들도 역시 리뷰에 참여할 수 있으며 우리는 이런 참가를 장려한다.
    * ``Fix/Fixing/Fixes/Close/Closing/Refs #ticket`` - 이슈 트래커에서 수정사항을 명시할 수 있다.
    * ``backport to _branch name_`` - if the fix needs to be cherry-picked to another branch (like 2.9.x, 2.10.x, etc)

예를 들면 다음과 같다.

    Adding monadic API to Future. Fixes #2731

      * Details 1
      * Details 2
      * Details 3

