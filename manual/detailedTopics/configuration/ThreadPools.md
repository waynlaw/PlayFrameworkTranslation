<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 플레이 스레드 풀 이해하기

플레이 프레임워크는 비동기 웹 프레임워크이다. 스트림은 이터레이티를 사용하여 비동기적으로 처리된다. 플레이에서 스레드 풀은 전통적인 웹 프레임워크와는 다르게 소수의 스레드를 사용하게 되며, 그렇기 때문에 플레이-코어에서 I/O는 절대 블록되지 않는다.

하여 블로킹 I/O 코드를 작성할 계획이거나 CPU 집약적인 수 많은 동작을 잠재적으로 수행할 것이라면 특히 스레드 풀이 작업량의 영향이 있다는 것을 알아야 하며 적절하게 튜닝이 필요하다. 플레이 프레임워크에서 블로킹 I/O에 이러한 튜닝이 없다면 결과적으로 아주 좋지 못한 성능을 나타나게 된다. 예를들어 CPU의 사용량은 5%에 머무르면서 초당 몇개의 요청만 처리 되는 모습을 보게 될 수도 있다.
반대로 적절하게 튜닝된다면 개발의 대표적인 하드웨어(예, 맥북 프로)의 벤치마크에서 플레이는 별 무리 없이 초당 100에서 1000이 넘는 작업량을 처리됨을 보여주기도 했다.

## 블로킹일 때를 알아야 한다

실제 플레이 애플리케이션에서 블록이 일어나는 대부분의 경우는 데이터베이스와 작업할 때이다. 불행하게도 주요 데이터베이스에는 JVM을 위한 비동기 데이터베이스 드라이버를 제공하는 것은 없으므로 대부분의 데이터베이스에서 블로킹 I/O를 사용할 수 밖에 없다. 주목할만한 예외사항은 [ReactiveMongo](http://reactivemongo.org/)로 MongoDB를 위한 드라이버가 있으며, 플레이의 이터레이티 라이브러리를 사용하여 MongoDB와 작업한다.

코드에서 블록을 당하는 다른 경우는 다음과 같다.

* 서드파티 클라이언트 라이브러리를 통하여 REST/WebService API를 사용한다. (플레이의 비동기 WS API는 아니다)
* 몇몇 메시징 기술에서 오직 메시지를 보내기 위해 동기적 API를 제공한다.
* 스스로 직접 파일이나 소켓을 열었을 때
* CPU 집약적인 명령은 수행에 오랜시간이 소요되기 때문에 블럭될 수 있다.

일반적으로 API에서 `Future` 반환을 사용하면 논-블로킹이고, 그렇지 않으면 블로킹이다.

> 주의 : 여러분의 블로킹 코드에서 Future로 감싸기를 유혹받을 수도 있다. 이렇게 해서 논-블로킹을 만들 수 있는 것은 아니고 다른 스레드에 의해서 블로킹이 일어난다는 것을 의미한다. 블로킹을 처리하기 위해 충분한 스레드를 가지고 사용해야 하는 스레드 풀에 대해서 여전히 확실하게 알 필요가 있다.

대조적으로 I/O의 다음 타입들은 블록되지 않는다.

* 플레이 WS API
* ReactiveMongo와 같은 비동기 데이터베이스 드라이버
* Akka 액터의 메시지 보내기 / 액터로 부터 메시지 받기

## 플레이의 스레드 풀

플레이는 여러가지 목적에 따라 다양한 스레드 풀을 사용한다.

* **Netty boss/worker thread pools** - These are used internally by Netty for handling Netty IO.  An applications code should never be executed by a thread in these thread pools.
* **Play Internal Thread Pool** - This is used internally by Play.  No application code should ever be executed by a thread in this thread pool, and no blocking should ever be done in this thread pool.  Its size can be configured by setting `internal-threadpool-size` in `application.conf`, and it defaults to the number of available processors.
* **Play default thread pool** - This is the default thread pool in which all of your application code in Play Framework is executed.  It is an Akka dispatcher, and can be configured by configuring Akka, described below. By default, it has one thread per processor.
* **Akka thread pool** - This is used by the Play Akka plugin, and can be configured the same way that you would configure Akka.

* **Netty boss/worker 스레드 풀** - 네티 I/O를 처리하기 위해 내부족으로 내티를 사용한다. 애플리케이션 코드는 절대 이들 스레드 풀의 스레드에 의해서 수행되지 않는다.
* **플레이 내부 스레드 풀** - 플레이에 의해 내부적으로 사용되어진다. 이 스레드 풀 안에서는 어떤 애플리케이션 코드도 스레드에 의해서 수행되어서는 안되고, 이 스레드 풀 안에서는 어떤 블로킹도 일어나서는 안된다. 이 크기의 환경설정은 `application.conf`의 `internal-threadpool-size`로 설정할 수 있다. 가능한 프로세스의 수가 기본 설정이다.
* **Play 기본 스레드 풀** - 플레이 프레임워크에서 모든 애플리케이션 코드를 실행하기 위한 기본 스레드 풀이다. Akka 디스패쳐이며, Akka를 환경설정하는 것으로 환경설정할 수 있고, 아래에서 설명했다. 기본적으로 프로세서 하나당 스레드 하나를 가진다.
* **Akka 스레드 풀** - 플레이 Akka 플러그인에 의해서 사용되어지는 것이며 Akka를 환경설정하는 것과 동일한 방법으로 설정할 수 있다.

## 기본 스레드 풀 사용하기

플레이 프레임워크에서 모든 액션은 기본 스레드 풀을 사용한다. 특정 비동기 명령을 수행할 때 예를들어 future에서 `map`이나 `flatMap`을 호출한다면 주어신 함수를 실행하기 위해 implicit 실행 컨텍스트(execution context)를 제공할 필요가 있을 것이다. 실행 컨텍스트는 기본적으로 `ThreadPool`을 위한 다른 이름이다.

대다수의 상황에서 적절한 실행 컨텍스트는 **플레이 기본 스레드 풀**이 될 것이다. 스칼라 소스 파일으로 임포트하여 사용할 수 있다.

@[global-thread-pool](code/ThreadPools.scala)

### 플레이 기본 스레드 풀 환경설정하기

기본 스레드 풀은 `application.conf`에 `play` 네임스페이스로 표준 Akka 환경설정을 사용하여 구성할 수 있다. 기본 환경설정은 다음과 같다.

@[default-config](code/ThreadPools.scala)

이 환경설정은 Akka에게 풀에 최대 24개의 스레드로 가능한 프로세서 하나당 하나의 스레드를 생성하라고 지시한다. 전체 사용가능한 환경설정 옵션은 [여기](http://doc.akka.io/docs/akka/2.2.0/general/configuration.html#Listing_of_the_Reference_Configuration)에서 찾아볼 수 있다.

> 이 환경설정은 플레이 Akka 플러그인에서 사용하는 환경설정에서 분리되었음을 주의하라. 플레이 Akka 플러그인은 최상단 네임스페이스( play { } 감싸는 것 없이) Akka 환경설정에 의해서 개별적으로 환경설정할 수 있다.

## 다른 스레드 풀 사용하기

특정 상황에서 다른 스레드 풀에 작업을 파견할 수 있다. 이는 CPU가 많이 사용되는 일이나 데이터베이스 접근 같은 I/O 작업일 것이다. 이렇게 하기 위해 우선 `ThreadPool`을 생성해야 하며, 스칼라에서는 다음과 같이 할 수 있다.

@[my-context-usage](code/ThreadPools.scala)

이 경우 우리는 `ExecutionContext`를 생성하기 위해 Akka를 사용한다. 그러나 자바 executor나 Scala 포크/조인 스레드 풀을 사용하여 독자적인 `ExecutionContext`도 쉽게 생성할 수 있다. Akka 실행 컨텍스트를 환경설정하기 위해서 다음 환경설정을 `application.conf`에 추가할 수 있다.

@[my-context-config](code/ThreadPools.scala)

스칼라에서 이 실행 컨텍스트를 사용하기 위해 간단하게 `Future` 컴패니언 오브젝트 함수를 사용할 수 있다.

@[my-context-explicit](code/ThreadPools.scala)

또는 implicit 으로 사용할 수도 있다.

@[my-context-implicit](code/ThreadPools.scala)

## 클래스 로더와 스레드 로컬

클래스 로더와 스레드 로컬은 플레이 프로그램 처럼 멀티스레드 환경에서 특별한 처리가 필요하다.

### 애플리케이션 클래스 로더

플레이 애플리케이션에서 [스레드 컨텍스트 클래스 로더](http://docs.oracle.com/javase/6/docs/api/java/lang/Thread.html#getContextClassLoader\(\))는 항상 애플리케이션 클래스들을 로드할 수 있는 것은 아니다. 명시적으로 애플리케이션 클래스 로더를 클래스들을 로드하기 위해 사용해야 한다.

자바
: @[using-app-classloader](code/ThreadPoolsJava.java)

스칼라
: @[using-app-classloader](code/ThreadPools.scala)

클래스 로드에 대해서 명시적으로 하는 것은 프로덕션 모드보다 (`run`을 사용한 ) 개발 모드로 플레이가 실행중일 때 더 중요하다. 왜냐하면 플레이의 개발 모드는 다중 클래스 로더를 사용하고, 그래서 원자적 애플리케이션 리로딩을 지원하기 때문이다. 플레이의 스레드들 중 몇몇은 애플리케이션의 클래스의 부분집합에 대해서 알고 있는 클래스 로더로 묶이게 된다.

몇몇의 경우 애플리케이션 클래스로더를 명시적으로 하지 않아도 괜찮다. 이는 때때로 서드파티 라이브러리를 사용할 때이다. 이런 경우 서드 파티 코드를 호출하기 전에 명시적으로 [스레드 컨텍스트 클래스 로더](http://docs.oracle.com/javase/6/docs/api/java/lang/Thread.html#getContextClassLoader\(\))를 설정할 필요가 있다. 만약 이렇게 한다면 서드 파티 코드를 한번 호출하고 난 뒤에는 이전 값으로 컨텍스트 클래스 로더를 되돌림을 기억하자.

### 자바 스레드 로컬

플레이에서 자바 코드는 현재 HTTP 요청처럼 컨텍스트 정보에 대해서 찾기 위해 `ThreadLocal`을 사용한다. 스칼라 코드는 컨텍스트를 전달하는 것 대신에 implicit 파라미터를 사용하기 때문에 `ThreadLocal`을 사용할 필요가 없다. `ThreadLocal`은 자바에서 사용되며 그래서 자바 코드는 어디서든지 컨텍스트 파라미터를 전달할 필요 없이 컨텍스트 정보를 접근할 수 있다.

적절한 컨텍스트 `ClassLoader`에 따라서 자바 `ThreadLocal`은 `HttpExecution`클래스를 통하여 제공되는 `ExecutionContextExecutor`에 의해 자동적으로 전파된다. (`ExecutionContextExecutor`는 스칼라의 `ExecutionContext` 또는 자바의 `Executor`이다. ) 이들 특별한 `ExecutionContextExecutor` 객체는 자동으로 생성되며 자바 액션이나 자바 `Promise`메소드에 의해 사용된다. 기본 객체는 기본 사용자 스레드 풀으로 감싸진다. 만약에 스스로 스레딩을 하고 싶으면 `HttpExecution` 클래스의 헬퍼메소드를 사용하여 스스로 `ExecutionContextExecutor`스레드를 얻어야 한다.

아래 예제에서 사용자 스레드 풀은 적절하게 스레드 로컬로 전파되는 새로운 `ExecutionContext`를 생성하기 위해 감싸졌다.

@[async-explicit-ec-imports](../../working/javaGuide/main/async/code/javaguide/async/controllers/Application.java)
@[async-explicit-ec](../../working/javaGuide/main/async/code/javaguide/async/controllers/Application.java)

## 모범 사례

여러분의 애플리케이션이 수행하는 작업의 종류에 매우 의존적인 다양한 스레드 풀 사이에서 어떻게 해야 여러분의 애플리케이션의 작업을 최고로 분할 할 수 있으며, 여러분이 원하는 컨트롤이 병렬로 수행될 수 있는 작업은 얼마나 될까. 문제를 위한 모든 해결책에 꼭 맞는 경우는 없으며, 여러분을 위한 최선의 선택은 스레드 풀에서 가지고 있는 구현체에서 애플리케이션의 블로킹 I/O에 대해 이해하는 것에서 온다는 것이다.

> 한가시 사실을 알려주면 JDBC는 데이터 베이스 엑세스를 위해 스레드 풀이 베타적으로 사용되어 DB 풀을 추정하여 커넥션의 갯수를 조절할 수 있는 블로킹 스레드 풀이다. 더 적은 양의 스레드는 가능한 커넥션의 수를 소비하지 못할 것이다. 반면에 사용 가능한 연결의 수보다 더 많은 스레드는 연결을 위한 주어진 커넥션에 아주 소비적일 수 있다.

아래에 플레이 프레임워크를 사용하고 싶은 사람들의 몇가지 프로필을 간략하게 설명했다.

### 순수 비동기성

이런 경우 여러분의 애플리케이션에서 블로킹 I/O를 수행하지 않을 수 있다. 절대 블로킹이 되지 않기 때문에 한 스레드 당 한 프로세서의 기본 환경설정이 여러분의 사용 방법에 아주 적합할 수 있고, 완료를 위해 추가적인 환경설정이 없다. 플레이 기본 실행 컨텍스트는 모든 케이스에서 사용할 수 있다.

### 높은 동기성

이 프로필은 자바 서블릿 컨테이너 처럼 웹 프레임워크 기반의 전통적인 동기적 I/O에 해당한다. 블로킹 I/O를 처리하기 위해 큰 스레드 풀을 사용한다. 대부분의 액션이 데이터베이스에 접근하는 것 처럼 데이터 베이스 동기적 I/O 호출을 하는 애플리케이션에 유용하며, 다양한 작업의 종류를 위해 동시성에 대한 통제가 필요가 없거나 원하지 않을 것이다. 이 프로필은 블로킹 I/O를 처리하기 위한 가장 간단한 것이다.

이 프로필에서 어디서든디 기본 실행 컨텍스트를 간단하게 사용할 수 있지만 다음과 같이 이 풀에서 아주 많은 수의 스레드를 가지도록 설정해야 한다.

@[highly-synchronous](code/ThreadPools.scala)

자바에서는 다른 스레드로 작업을 파견하기가 어렵기 때문에 이 프로필은 동기적 I/O를 수행하는 자바 애플리케이션을 위해 추천 된다.

`parallelism-min`와 `parallelism-max`를 위해 동일한 값을 사용했음을 주의하라. 그 이유는 스레드의 수가 다음 관용어구에 의해서 정의되었기 때문이다.

>base-nb-threads = nb-processors * parallelism-factor  
 parallelism-min <= actual-nb-threads <= parallelism-max

그래서 가능한 프로세서가 충분하지 않으면 절대로 `parallelism-max` 설정에 닿지 않을 것이다.

### 많은 특정 스레드 풀

This profile is for when you want to do a lot of synchronous IO, but you also want to control exactly how much of which types of operations your application does at once.  In this profile, you would only do non blocking operations in the default execution context, and then dispatch blocking operations to different execution contexts for those specific operations.

이 경우에는 다음과 같이 명령의 다양한 종류를 위해 수 많은 다양한 실행 컨텍스트를 생성할 수도 있을 것이다.

@[many-specific-contexts](code/ThreadPools.scala)

이들은 다음과 같이 환경설정 할 수 있다.

@[many-specific-config](code/ThreadPools.scala)

따라서 여러분의 코드에서, `Future`를 생성하고 `Future`가 수행할 작업의 종류를 위한 `ExecutionContext`를 전달해야 할 것이다.

> **주의:** 환경설정 네임스페이스는 `Akka.system.dispatchers.lookup`에서 전달되는 디스패처 ID에 매치되는 동안 자유롭게 선택할 수 있다.

### 몇몇 특정 스레드 풀

이는 많은 특정 스레드풀과 높은 동기성 프로필 사이에 결합이다. 기본 실행 컨텍스트에서 대부분의 간단한 I/O를 수행할 수 있으며 적절하게 높은 수의 스레드 수를 설정할 수 있지만 한 번에 종료되는 스레드의 수를 한정한다면 디스패치는 특정 컨텍스트를 위해 특히 비싼 명령일 수 있다.
