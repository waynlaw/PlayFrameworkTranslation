<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 로깅 환경설정하기

플레이는 로깅 엔진으로 [Logback](http://logback.qos.ch/)을 사용한다. 환경설정의 자세한 내용을 위해 [Logback 문서](http://logback.qos.ch/manual/configuration.html)를 살펴보자.

## 기본 환경설정

플레이는 프로덕션에서 다음과 같은 기본 환경설정을 사용한다.

@[](/confs/play/logback-play-default.xml)

이 환경설정에 대해서 몇 가지 내용을 살펴보자.

* 파일 어펜더에 로그를 `logs/application.log`에 작성하라고 명시한다.
* 파일 로거는 모든 예외 스택 트레이스를 로그로 남기며, 반면에 콘솔 로거는 오직 예외 스택 트레이스의 10 줄만 로그로 남긴다.
* 플레이는 레벨 메시지에서 기본으로 ANSI 색갈 코드를 사용한다.
* 플레이는 로그백의 [AsyncAppender](http://logback.qos.ch/manual/appenders.html#AsyncAppender)로 파일 로거와 콘솔 양쪽에 넘긴다. 더 자세한 성능 영향도를 확인하기 위해 이 [블로그](http://blog.takipi.com/how-to-instantly-improve-your-java-logging-with-7-logback-tweaks/)를 살펴보자.

## 커스텀 환경설정

커스텀 환경설정을 위해 로그백 환경설정 파일을 명시해야 한다.

### 프로젝트 소스로부터 환경설정 파일 사용하기

`logback.xml`이라고 불리는 제공되는 파일으로 기본 로깅 환경설정을 제작할 수 있다.

### 외부 환경설정 파일 사용하기

또한 시스템 프로퍼티로 환경설정 파일을 명시할 수도 있다. 이는 특히 환경설정 파일이 애플리케이션 소스에서 관리되지 않는 프로덕션 환경에서 유용하다.

> 주의: 로깅 시스템은 시스템 프로퍼티에 의해서 명시된 환경설정 파일을 최우선으로 하며, 두번째는 `conf` 디렉터리에 있는 파일이고, 마지막은 기본이다. 이는 애플리케이션의 로깅 환경설정을 취향에 맞게 설정하고 특정 환경이나 개발자 설정을 위해 오버라이드 할 수 있다.

#### `-Dlogger.resource` 사용하기

클래스패스로부터 환경설정 파일을 불러오기를 명시할 수 있다.

```
$ start -Dlogger.resource=prod-logger.xml
```

#### Using `-Dlogger.file`

파일 시스템으로부터 환경설정 파일을 불러오기를 명시할 수 있다.

```
$ start -Dlogger.file=/opt/prod/logger.xml
```

### 예제

파일 어펜더 롤링을 사용할 뿐만 아니라 접근 로그 산출을 위해 개별 어펜더를 사용하는 환경설정의 예제가 있다.

```xml
<configuration>

    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${user.dir}/web/logs/application.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- Daily rollover with compression -->
            <fileNamePattern>application-log-%d{yyyy-MM-dd}.gz</fileNamePattern>
            <!-- keep 30 days worth of history -->
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%date{yyyy-MM-dd HH:mm:ss ZZZZ} [%level] from %logger in %thread - %message%n%xException</pattern>
        </encoder>
    </appender>
    
    <appender name="ACCESS_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${user.dir}/web/logs/access.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- daily rollover with compression -->
            <fileNamePattern>access-log-%d{yyyy-MM-dd}.gz</fileNamePattern>
            <!-- keep 1 week worth of history -->
            <maxHistory>7</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%date{yyyy-MM-dd HH:mm:ss ZZZZ} %message%n</pattern>
            <!-- this quadruples logging throughput -->
            <immediateFlush>false</immediateFlush>
        </encoder>
    </appender>

    <!-- additivity=false ensures access log data only goes to the access log -->
    <logger name="access" level="INFO" additivity="false">
        <appender-ref ref="ACCESS_FILE" />
    </logger>
    
    <root level="INFO">
        <appender-ref ref="FILE"/>
    </root>

</configuration>

```

이 예제에서 알 수 있는 몇 가지 유용한 내용이 있다.

- 점점 증가하는 로그 파일 관리를 도와주는 `RollingFileAppender`를 사용한다.
- 애플리케이션 외부의 디렉터리에 파일로 로그를 작성한다. 그래서 애플리케이션의 업그레이드에 영향을 받지 않는다.
- `FILE` 어펜더는 확장된 메시지 포맷을 사용한다. 이 포맷은 Sumo Logic같은 서드파티 로그 분석 프로바이더에 의해서 분석될 수 있다.
- `access`로거는 `ACCESS_FILE_APPENDER`를 사용하여 개별 로그 파일로 전송한다.
- 모든 로거는 프로덕선 로깅을 위한 기본적인 선택인 `INFO`로 기준이 정해져 있다. 

## Akka 로깅 환경설정

Akka는 개별 로깅 시스템을 가지고 있다. 이는 어떻게 환경설정하는지에 따라 플레이의 기초를 이루는 로깅 엔진을 사용할수도 사용하지 않을 수도 있다.

기본적으로, Akka는 플레이의 로깅 환경설정을 무시하고 로그 메시지를 STDOUT에 아카 내의 자체 포멧을 사용하여 출력한다. `application.conf`에 로그 레벨을 설정할 수 있다.

```properties
akka {
  loglevel="INFO"
}
```

플레이의 로깅 엔진을 사용하는 Akka를 지정하려면, 환경설정을 하는데 몇가지 주의할 필요가 있다. 우선 `application.conf`에 다음 설정을 추가한다.

```properties
akka {
  loggers = ["akka.event.slf4j.Slf4jLogger"]
  loglevel="DEBUG"
}
```

몇 가지 사항에 주의하자.

- `akka.loggers`를 `["akka.event.slf4j.Slf4jLogger"]`로 설정하는 것은 Akka가 플레이에 내재된 로깅 엔진을 사용할 것처럼 된다.
- `akka.loglever` 프로퍼티는 Akka가 로그 엔진으로 로그 요청을 전달한다는 기준을 설정한다. 그러나 로깅 결과는 제어할 수 없다. 한번 로그 요청이 전달 되면, 로그백 환경설정은 기본으로 어펜더와 로그 레벨을 제어한다. 아카 컴포넌트를 위한 로그백 환경설정을 사용할 수 있는 가장 작은 기준인 `akka.loglevel`를 설정할 필요가 있다.

 
다음으로 로그백 환경설정에서 Akka 로깅 설정을 조금 수정한다.

```xml
<!-- Set logging for all Akka library classes to INFO -->
<logger name="akka" level="INFO" />
<!-- Set a specific actor to DEBUG -->
<logger name="actors.MyActor" level="DEBUG" />
```

스레드나 액터 주소처럼 유용한 프롵퍼티들을 포함한 Akka 로거를 위한 어펜더를 설정해주길 바란다.


Akka의 로깅 환경설정하는 것에 대해서 더 많은 정보를 위해, 로그백과 Slf4j 합성의 자세한 정보를 포함하고 있는 [Akka 문서](http://doc.akka.io/docs/akka/current/scala/logging.html)를 살펴보자.


