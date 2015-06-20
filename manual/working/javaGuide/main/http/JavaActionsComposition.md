<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# Actions 조합하기

이번 챕터에서는 일반적인 action 기능을 정의하는 몇가지 방법에 대해 소개한다.

## action에서 기억해야 하는 것

이전에 말했듯이 action은 `play.mvc.Result`을 리턴하는 Java 메소드이다. 사실 플레이는 내부적으로 function처럼 action을 관리한다. 왜냐하면 Java는 first class functions을 지원하지 않기 때문이다. 사실 Java API에서 제공하는 단일 action은 [`play.mvc.Action`](api/java/play/mvc/Action.html)의 인스턴스이다:

```java
public abstract class Action {
  public abstract Promise<Result> call(Context ctx) throws Throwable;
}
```

플레이는 root action을 빌드하여, 정해진 action method를 호출 할 수 있게 해준다. 이러한 과정을 통해 더욱 복잡한 action 조합이 가능하다.

## action 조합하기

`VerboseAction`에 대한 정의:

@[verbose-action](code/javaguide/http/JavaActionsComposition.java)

`play.mvc.Action`가 제공하는 action 메소드를 통해 코드를 조합하여 사용할 수 있다. `@With` 애노테이션을 사용한다:

@[verbose-index](code/javaguide/http/JavaActionsComposition.java)

`delegate.call(...)`를 사용하여 wrapped(한번 감싸진) action에 위임을 해야 할 때가 있을 수도 있다.

custom action 애노테이션을 이용하면 여러개 action을 조합할 수도 있다:

@[authenticated-cached-index](code/javaguide/http/JavaActionsComposition.java)

> **Note:**  ```play.mvc.Security.Authenticated``` 와 ```play.cache.Cached``` 애노테이션 그리고 이에 상응하는 기선언 된 Action들이 플레이와 함께 제공 된다. 더 자세한 정보는 관련된 API 문서를 참고하자.

## custom action annotation 정의하기

커스텀 애노테이션을 이용해 action 조합을 표기할 수도 있다. 이 경우 반드시 자기자신을 `@With` 애노테이션을 이용해 표기해야 한다:

@[verbose-annotation](code/javaguide/http/JavaActionsComposition.java)

`Action` 정의 시, 애노테이션을 일종의 설정 값으로 불러온다:

@[verbose-annotation-action](code/javaguide/http/JavaActionsComposition.java)

이제 action 메소드를 이용해여 새로운 애노테이션을 사용할 수 있다:

@[verbose-annotation-index](code/javaguide/http/JavaActionsComposition.java)

## Annotating controllers

`Controller` 클래스에 직접적으로 action 조합 애노테이션을 삽입할 수도 있다. 이 경우 아래의 컨트롤러가 정의한 모든 액션 메소드에 애노테이션이 적용되게 된다.

```java
@Authenticated
public class Admin extends Controller {
    
  …
    
}
```

## action에서 controller로 object 전달하기

context args map을 이용하여 action에서 controller로 object을 전달할 수 있다.

@[pass-arg-action](code/javaguide/http/JavaActionsComposition.java)

그리고 이 action에서 아래와 같은 arge를 얻어올 수 있다:

@[pass-arg-action-index](code/javaguide/http/JavaActionsComposition.java)
