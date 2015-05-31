<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 플레이 설치하기

## 요구사항

JDK 1.8(또는 그 이상)이 장비에 설치되어 있어야 한다.([전반적인 설치 작업](#JDK-설치))

## 액티베이터 설치하기

* 최신 버전의 [Typesafe Activator](http://typesafe.com/activator)를 **다운로드**하라.
* 원하는 위치에 파일의 압축을 **해제**하라.
* 명령행 윈도우를 **실행**하고 아래의 내용을 입력하라.

```
cd activator*
activator ui
```

[http://localhost:8888](http://localhost:8888)를 통해 로컬 플레이 설치에 접근할 수 있다.

문서와 즉시 실행할 수 있는 애플리케이션의 샘플 목록을 확인할 수 있다. 간단한 실행을 확인하기 위해 **play-java**를 선택해보자.

### 명령어 라인

파일 시스템의 다른 곳에서 플레이를 사용하기 위해 다음을 수행하라:

* **activator** 디렉터리를 패스에 추가하라 ([전반적인 설치 작업](#실행-가능하도록-경로-추가)을 살펴보자).

아주 간단하게 `play-java`기반의 `my-first-app` 템플릿을 생성할 수 있다:

```
activator new my-first-app play-java
cd my-first-app
activator run
```

[http://localhost:9000](http://localhost:9000)를 통해 애플리케이션에 접속할 수 있다.

이제 플레이로 작업할 준비가 끝났다!

## 전반적인 설치 작업

시스템에 플레이!를 설치하기 위해 전반적인 작업을 처리해야 한다.

### JDK 설치

JDK 버전 1.8 이상이 장비에 설치되었는지 확인하라. 간단하게 다음 명령어로 확인할 수 있다:

```bash
java -version
javac -version
```

만약에 JDK가 없다면 다음 방법으로 설치할 수 있다:

* **MacOS**, 자바가 기본 설치되어 있지만 [최신버전으로 업데이트](https://www.java.com/en/download/help/mac_install.xml) 해야 할 수 도 있다.
* **Windows** [최신버전의 JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 패키지를 다운로드하고 설치하라.
* **Linux**, 오라클 JDK나 OpenJDK를 사용하라.(gcj는 사용하지 말자)
 
### 실행 가능하도록 경로 추가

편의를 위해 액티베이터 설치 디렉터리를 시스템 `PATH`에 추가할 필요가 있다.

**Unix**

```bash
export PATH=/path/to/activator:$PATH
```

**Windows**

`;C:\path\to\activator`를 `PATH`환경 변수를 추가하라. 공백을 사용하지 않도록 주의히라.

### 파일 권한

**Unix**

`activator`를 실행하면 배포 디렉터리에 파일들을 내려받기 때문에 특별한 권한이 필요한 `/opt`, `/usr/local`에는 액티베이터를 설치하지 않는 것이 좋다.

`activator`가 잘 실행되는지 확인하라. 만약 실행되지 않는다면 `chmod u+x /path/to/activator`를 실행하라.

### 프록시 설정

프록시를 사용하기 위해 윈도우에서 `set HTTP_PROXY=http://<host>:<port>`를 정의하거나 유닉스에서 `export  HTTP_PROXY=http://<host>:<port>`를 확인하라.
