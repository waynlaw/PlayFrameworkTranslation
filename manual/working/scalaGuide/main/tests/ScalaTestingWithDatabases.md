<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 데이터베이스와 테스트하기

[[ScalaTest|ScalaFunctionalTestingWithScalaTest]]나 [[specs2|ScalaFunctionalTestingWithSpecs2]]를 사용하여 함수형 테스트들을 작성할 수 있다는 것은 데이터베이스를 포함한 모든 애플리케이션이 기동될때 데이터 베이스에 접근하는 코드도 테스트 할 수 있다는 것을 의미한다. 하지만 전체 애플리케이션을 기동하는 것은 아주 많은 구성요소들을 시작시키고 유지해야하는 복잡
성 때문에 원하지 않게되고, 단지 애플리케이션의 작은 부분만을 테스트하기를 원하게 된다.

플레이는 데이터베이스 접근 코드를 테스트 하는데 도움을 주는 몇가지 도구들을 제공해 준다. 이 도구는 데이터베이스와의 테스트는 허용하지만, 앱의 나머지 부분과는 독립적이다. 이러한 도구들은 ScalaTest나 specs2에서 쉽게 사용할 수 있으며, 무겁고 느린 테스트들 대신 보다 가볍고 빠르게 유닛테스트를 실행할 수 있게 해준다.

## 데이터베이스 사용하기

데이터 베이스에 접근하기 위해서는 적어도 데이터베이스의 이름과 주소를 필요로 하며, [`데이터베이스`](api/scala/index.html#play.api.db.Database$) 컴패니언 오브젝트를 사용하면 된다. MySQL에 접속하는 예제는 아래와 같다.

@[데이터베이스](code/database/ScalaTestingWithDatabases.scala)

위의 예제는 `localhost`에서 MySQL의 `test` 데이터 베이스를 `default`라는 이름으로 만들고, 접속하기 위한 풀을 생성하는 것이다. 데이터베이스의 이름은 오직 플레이 내부적으로만 사용된다. 예를 들면, 진화들과 같은 다른 기능들은, 그 데이터베이스에 관계된 자료들만 불러들인다.

특별한 이름, 사용자 이름과 같은 설정값, 플레이가 지원하는 다양한 커넥션 풀을 위한 패스워드와 같은 것을 포함해서, 데이터베이스를 위한 다른 설정을 지정하는 것을 원할 수 있다. 이런 것들은 특별한 이름 파라메터나, 특별한 설정 파라메터를 동해서 지원된다.:

@[전체 설정](code/database/ScalaTestingWithDatabases.scala)

데이터베이스를 사용한 다음에는, 접속을 열려진 상태로 유지하기 위해서, 일반적으로 접속을 접속풀에 넣는다. 이 접속 풀은 열린 접속들을 유지하며, 실행중인 쓰레드를 가지고 있을 수 있다. 만일 이것을 닫기를 원한다면 `shutdown`함수를 호출하면 된다.

@[shutdown](code/database/ScalaTestingWithDatabases.scala)

데이터 베이스를 수동으로 생성하고 닫는 것을 테스트 프레임워크의 시작과 끝 코드에서 할 경우에는 유용하다. 하지만 그렇지 않는 경우에는 플레이가 접속풀을 관리하도록 하는 것을 추천한다.

### 플레이에게 데이터 베이스를 관리하게 하는 것

플레이는 플레이에 의해 관리되는 데이터베이스 접속 풀에서 코드 블럭이 실행될 수 있도록 해주는 `withDatabase` 도우미를 제공한다. 플레이는 그 코드 블럭이 실행을 끝나치고 나면 올바르게 접속이 종료되는것을 보장해 준다.:

@[데이터베이스와 함께하기](code/database/ScalaTestingWithDatabases.scala)

`Database.apply` 생성 함수와 마찬가지로, `withDatabase`또한 원하는 이름과 설정 쌍을 전달할 수 있다.

일반적으로 `withDatabase`를 모든 테스트에서 직접적으로 사용하는 것은 엄청난 양의 반복되는 코드를 만든다. 테스트를 사용할 때는, 반복되는 코드를 제거하기 위해서 도움을 줄 수 있는 함수를 만드는 것을 권장한다. 예:

@[custom-with-database](code/database/ScalaTestingWithDatabases.scala)

각각의 테스트에서 최소한의 반복되는 코드를 넣어 쉽게 사용할 수 있다.:

@[custom-with-database-use](code/database/ScalaTestingWithDatabases.scala)

> **팁:** 테스트 데이터 베이스 설정을 외부에서 사용하기 위해서, 어떠한 데이터베이스를 사용하고 어떻게 접속할지에 대해 환경 변수나 시스템 속성을 사용할 수 있다. 이 작업을 통해 개발자들은 최대한의 유연성을 가지게 되어 CI 시스템들이 개발을 위해 서로다른 환경을 제공하는 것처럼, 그들이 원하는데로 설정할 수 있는 자신만의 고유한 환경을 가질 수 있다. 

### 메모리내의 데이터베이스 사용하기

어떤사람들은 테스트를 실행하는 과정에서 데이터베이스와 같은 기반 시설을 설치하는것을 원하지 않는다. 플레이는 이러한 목적에 도움을 주기 위해서 H2 메모리내의 데이터베이스를 제공한다.:

@[in-memory](code/database/ScalaTestingWithDatabases.scala)

메모리내의 데이터베이스는 사용자정의된 이름과 URL 인자, 접속 풀 설정에 대한 설정이 가능하다. 아래의 예제에서는 `MySQL`로 H2가 동작하도록 하기 위해서 `MODE` 전달인자를 지원하는 것과 모든 로그를 남기기 위해 접속 풀을 설정하는 것을 보여준다.:

@[in-memory-full-config](code/database/ScalaTestingWithDatabases.scala)

일반적인 데이터베이스 생성자와 관련하여, 항상 메모리내의 데이터베이스 접속 풀을 닫았는지를 확인해야 한다.:

@[in-memory-shutdown](code/database/ScalaTestingWithDatabases.scala)

테스트 프레임워크들의 전/후 호환성들을 사용하지 않는다면, 플레이가 메모리 내의 데이터베이스를 관리하기를 원할 수 있을 것이다. 이것은 `withInMemory`를 이용하여 가능하다.:

@[with-in-memory](code/database/ScalaTestingWithDatabases.scala)

`withDatabase`와 같이, 중복 코드를 줄이기 위해서 `withInMemory`를 부르는 코드를 감싸는 함수를 만드는 것을 권장한다.:

@[with-in-memory-custom](code/database/ScalaTestingWithDatabases.scala)

## 진화들 적용하기

테스트들을 실행할 때, 데이터베이스 관리를 위한 데이터베이스 스키마를 일반적으로 원할 것이다. 만일 이미 진화들을 적용하고 있다면, 그건 종종 테스트에서 같은 진화들을 개발과 실제 환경에서 재사용할 수 있게 해줄 것이다. 또한 테스트를 위해서 고유한 진화들을 만들기를 원할 수 있습니다. 전체 플레이 애플리케이션에 대해 실행하지 않고도 진화를 적용하고 관리할 수 있도록 편리하게 도움을 주는 것을 플레이에서 제공한다.

진화들을 적용하기 위해서는, [`Evolutions`](api/scala/index.html#play.api.db.evolutions.Evolutions$) 컴페니언 오브젝트에서 `applyEvolutions`를 사용할 수 있다.:

@[apply-evolutions](code/database/ScalaTestingWithDatabases.scala)

이것은 `evolutions/<databasename>`폴더 내의 클래스패스 경로에서 진화를 읽어들이고 적용할 것이다.

테스트가 실행된 이후에는, 데이터 베이스 초기화를 원할 것이다. 만일 당신의 진화가 아래와 같은 스크립트로 작성되었다면, 간단히 `cleanupEvolutions`를 호출하여 모든 데이터베이스 테이블들을 삭제할 수 있다.:

@[cleanup-evolutions](code/database/ScalaTestingWithDatabases.scala)

### 고유한 진화 만들기

몇몇 상황에서 테스트내에 고유한 진화를 실행하기를 원할 수 있다. 고유한 진화들은 고유한 [`EvolutionsReader`](api/scala/index.html#play.api.db.evolutions.EvolutionsReader)를 사용하여 이용할 수 있다. 가장 간단한 이런 것들은 [`SimpleEvolutionsReader`](api/scala/index.html#play.api.db.evolutions.SimpleEvolutionsReader)이며, 그것은 데이터베이스 이름들을 [`Evolution`](api/scala/index.html#play.api.db.evolutions.Evolution)스크립트들의 순차열로 바꿔주는 미리 설정된 맵인 혁명 리더이다. [`SimpleEvolutionsReader`](api/scala/index.html#play.api.db.evolutions.SimpleEvolutionsReader$)컴패니언 오브젝트상에서 편리한 함수들을 사용할 수 있도록 만들어져 있다. 예:

@[apply-evolutions-simple](code/database/ScalaTestingWithDatabases.scala)

고유한 진화를들을 정리하는 것은 일반적인 진화들을 정리하는 것과 같은 방법을 사용한다. `cleanupEvolutions`함수를 사용하자.:

@[cleanup-evolutions-simple](code/database/ScalaTestingWithDatabases.scala)

메모: 데이터베이스를 종료하기 위해 사용되는 종료 스크립트를 포함하여, 고유한 진화를 읽는 것을 여기에 전달할 필요는 없다. 진화의 상태는 데이터베이스 내에 보관되어 있기 때문이다.

때로는 고유한 진화 스크립트들을 코드에 넣는것은 비효율적일 수 있다. 만일 그런 경우에는 [`ClassLoaderEvolutionsReader`](api/scala/index.html#play.api.db.evolutions.ClassLoaderEvolutionsReader)를 사용해서 그것들을 설정된 경로 아래의 테스트 자원 경로에 넣을 수 있다. 예:

@[apply-evolutions-custom-path](code/database/ScalaTestingWithDatabases.scala)

이것은 `testdatabase/evolutions/<databasename>/<n>.sql`에서 개발과 실사용을 위해서 동일한 구조와 형식으로 진화를 읽어들일 것이다.

### 플레이가 진화들을 관리할 수 있도록 하기

테스트의 전, 후에 진화의 실행을 관리하기 위해서 테스트 프레임워크를 사용하는 경우에는 `applyEvolutions`와 `cleanupEvolutions` 함수가 유용하다. 플레이는 동일한 관리를 위해서 좀 더 간단한 편의성 함수인 `withEvolutions`를 제공한다.:

@[진화들과 함께하기](code/database/ScalaTestingWithDatabases.scala)

자연적으로 `withEvolutions`는 중복된 코드를 줄이기 위해 `withDatabase`나 `withInMemory`와 함께 사용할 수 있다. 또한 데이터베이스를 초기화하고 진화들을 실행하는것을 허용한다.:

@[with-evolutions-custom](code/database/ScalaTestingWithDatabases.scala)

우리의 테스트들을 위한 고유한 데이터베이스 관리 함수는 선언되있다. 이제는 그것을 바로 사용할 수 있다.:

@[with-evolutions-custom-use](code/database/ScalaTestingWithDatabases.scala)
