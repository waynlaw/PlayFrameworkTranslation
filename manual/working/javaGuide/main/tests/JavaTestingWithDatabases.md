# 데이터베이스를 이용하여 테스트 하기

데이터베이스를 포함한 전체 애플리케이션을 기동하여 데이터베이스에 접속을하는 [[함수적인 테스트|JavaFunctionalTest]]를 작성하기 위해 전체 애플리케이션을 기동하는 것은 지향할만한 상황은 아니다. 기동시에 많은 컴포넌트의 복잡성을 가지고 있기 때문이기도 하고, 애플리케이션의 작은 부분을 테스트하기 위해 실행하는 목적이 주이기 때문이다.

플레이는 데이터베이스를 테스트 하기 위해 데이터베이스 접속 테스트를 도와주는 몇몇의 유틸리티를 제공하고 있다. 하지만 작성한 app의 부분과는 분리되어 있는 상태이다. 이 유틸리티들은 ScalaTest나 specs2에서 쉽게 사용 가능하다. 그리고 데이터베이스 테스트를 하기위해 무겁고 느린 함수형 테스트 대신, 더욱 경량화 되어 있고, 빠른 유닛 테스트를 실행할 수 있도록 도와준다.

## 데이터베이스 사용하기

데이터페이스에 접속하기 위해선 최소한 데이터배이스 드라이버 이름과 데이터베이스의 url이 필요하고, [`Database`](api/java/play/db/Database.html) 스테틱 팩토리 메소드를 이용하면 된다. 예를 들어 MySQL에 접속하기 위해, 아래와 같이 사용해야 한다:

@[database](code/javaguide/tests/JavaTestingWithDatabases.java)

이 경우 `localhost`에 실행중인 MySQL `test` 데이스 데이터베이스에 접속하는 데이터베이스 접속 풀에 `default`라 이름을 붙여 생성하게 된다. 데이터베이스 이름은 플레이 내부에서만 사용된다. 예를 들어 해당 데이터베이스와 관련된 리소스를 로드 하기 위해 evolutions와 같은 다른 기능에 의해 쓰이기도 한다.

아마 커스텀 이름, 혹은 유저 이름, 패스워드와 같은 설정 프로퍼티, 그리고 플레이가 제공하는 다양한 커넥션 풀 설정 요소등을 포함한 데이터베이스의 다른 설정을 추가하고 싶을 수도 있다. 커스텀 파라메터 이름을 이용하거나, 커스텀 설정 파라미터를 이용하여 이런 기능을 제공할 수 있다:

@[full-config](code/javaguide/tests/JavaTestingWithDatabases.java)

데이터베이스 사용 후에는, 실행중인 쓰레드에서 소유한 커넥션을 들고 있는 데이터베이스 커넥션 풀에 커넥션을 돌려줘야 하기 때문에 셧다운 처리를 해줘야 한다. `shutdown` 메소드를 호출하여 실행이 가능하다:

@[shutdown](code/javaguide/tests/JavaTestingWithDatabases.java)

이 메소드는 JUnit의 `@Before` 와 `@After`와 함께 사용하면 유용하게 사용할 수 있다, 예를 들어:

@[database-junit](code/javaguide/tests/JavaTestingWithDatabases.java)

> **팁:** 어떻게 데이터베이스에 접속하는지나 어떤 데이터베이스를 사용하는지 설정하기 위해 환경변수나 시스템 프로퍼티를 사용하여 데이터베이스 설정 테스트를 외부화시킬 수도 있다. 이를 통해 개발자가 즐거워할만한 방식으로 자신만으로 환경을 설정할 수 있도록 최대한의 유연성을 제공하는 동시에 개발과정에서 각자 다른 특정 환경에 대해 CI 시스템을 제공할 수 있게 된다.

### 인메모리 데이터베이스 사용하기

일부 사람들은 테스를 실행하기 위해서 데이터베이스를 설치하는 인프라가 필요없는 것을 선호하기도 한다. 플레이는 이러한 목적을 위해 H2 인메모리 데이터베이스를 생성하기 위한 간단한 헬퍼를 제공한다:

@[in-memory](code/javaguide/tests/JavaTestingWithDatabases.java)

인메모리 데이터베이스는 커스텀 URL 매개변수, 커스텀 커넥션 풀 설정, 커스텀 이름 등을 제공하여 설정이 가능하다. 아래의 경우 `MODE` 매개변수를 제공하여 모든 스테이트먼트를 기록하도록 설정되어 있는 커넥션 풀을 설정함과 동시에,  H2에서 `MySQL`를 흉내낼 수 있도록 해주었다:

@[in-memory-full-config](code/javaguide/tests/JavaTestingWithDatabases.java)

일반적인 데이터베이스 팩토리를 이용하게 되면, 항상 인메모리 데이터베이스 커넥션 풀을 셧다운 시켜야 한다:

@[in-memory-shutdown](code/javaguide/tests/JavaTestingWithDatabases.java)

## evolutions 적용하기

테스트를 실행할 때, 전형적으로 데이터베이스 스키마를 관리하기를 원한다. 이미 evolutions를 사용하고 있다면, 개발과정과 프로적션 단계 테스트에서 사용하였던 evolutions과 같은 것을 재사용하는 것이 상식에 맞을 것이다. 아마 테스트만 진행하기 위해 커스텀 evolutions을 생성하기를 원할 수도 있다. 플레이는 전체 플레이 애플리케이션을 실행할 필요 없이 evolutions을 관리하고 적용하기 위해 간편한 헬퍼들을 제공하고 있다.

evolutions을 적용하기 위해, [`Evolutions`](api/java/play/db/evolutions/Evolutions.html) 스테닉 클레서에서 `applyEvolutions`를 사용할 수 있다:

@[apply-evolutions](code/javaguide/tests/JavaTestingWithDatabases.java)

이를 통해 `evolutions/<databasename>` 디렉토리에 위치한 클레스패스에서 evolutions를 불러와 적용할 수 있다.

테스트가 실행된 후, 원래 상태로 데이터베이스를 원복시키고 싶을 수 있다. 모든 데이터베이스 테이블을 제거하는 등의 사용자 evolutions down 스크립트를 구현해햐 한다면, `cleanupEvolutions` 메소드르 호출하여 간단히 이 과정을 시행할 수 있다:  

@[cleanup-evolutions](code/javaguide/tests/JavaTestingWithDatabases.java)

### 커스텀 evolutions

특정 경우에 테스트에서 커스텀 evolutions를 시행시키고 싶을 때가 있다. 커스텀 [`EvolutionsReader`](api/java/play/db/evolutions/EvolutionsReader.html)를 사용하여 커스텀 evolutions를 사용할 수 있다. 이를 수행하는 가장 쉬운 방법은 `Evolutions`의 스테틱 팩토리 메소드를 사용하는 것이다. 예를 들어 `forDefault`는 기본 데이터베이스를 타깃으로 한 간단한 [`Evolution`](api/java/play/db/evolutions/Evolution.html) 스크립트 리스트를 읽어오는 evolutions 리더를 생성해 준다. 예를 들어:

@[apply-evolutions-simple](code/javaguide/tests/JavaTestingWithDatabases.java)

커스텀 evolutions를 제거해주는 작업은 regular evolutions를 제거하는 방법과 같은 방법이다. `cleanupEvolutions` 메소드를 사용한다:

@[cleanup-evolutions-simple](code/javaguide/tests/JavaTestingWithDatabases.java)

여기서 커스텀 evolutions 리더를 꼭 넘겨주지 않아도 된다. 왜냐하면 데이터베이스를 해제할 때 사용되는 down 스크립트를 포함하고 있는 evolutions 상태는 데이터베이스에 저장이 되어있기 때문이다. 

때로는 코드에 커스텀 evolution 스크립트를 넢는 과정이 실용적이지 않을 수도 있다. 이런경우에는 `fromClassLoader` 팩토리 메소드를 사용하여 테스트 리소스 폴더에 스크립트를 넣어도 된다: tory method:

@[apply-evolutions-custom-path](code/javaguide/tests/JavaTestingWithDatabases.java)

이를 통해 `testdatabase/evolutions/<databasename>/<n>.sql`에서 개발과 프로덕션 단계에서 사용되었던 형식과 구조를 그대로 이용하여 evolutions를 불러올 수 있게 된다. 

### JUnit과 통합하기

전형적으로 같은 evolutions을 이용하는 테스트를 많이 가지고 있을 수도 있다. 따라서 before와 after 메소드에 evolutions 설정 코드를 따로 빼서 데이터베이스 기동과 해지시에 연동이 될 수 있도록 만드는 것이 실용적이다. 여기에 복잡하게 구성되어 있는 예시가 있다:

@[database-test](code/javaguide/tests/DatabaseTest.java)
