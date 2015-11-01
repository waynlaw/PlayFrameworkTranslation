<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# JDBC 환경 설정하기.

플레이의 JDBS 데이서소스는 [HikariCP](http://brettwooldridge.github.io/HikariCP/)에 의해서 관리되고 있다.

## 특별한 URL

플레이는 **MySQL**나 **PostgreSQL**를 위한 특별한 URL 형식을 지원한다.

```
# To configure MySQL
db.default.url="mysql://user:password@localhost/database"

# To configure PostgreSQL
db.default.url="postgres://user:password@localhost/database"
```

표준이 아닌 데이터 베이스의 포트 서비스는 다음과 같이 지정할 수 있다.

```
# To configure MySQL running in Docker
db.default.url="mysql://user:password@localhost:port/database"

# To configure PostgreSQL running in Docker
db.default.url="postgres://user:password@localhost:port/database"
```

## 레퍼런스

`driver`, `url`, `username`, `password` 환경설정 프로퍼티를 추가하기 위해 또한 추가적인 설정이 지원되어야 한다. 플레이의 JDBC `reference.conf`로부터 `play.db.prototype` 환경설정은 모든 데이터베이스 연결을 위한 환경설정의 프로토타입으로 사용된다. 가능한 모든 환경설정 옵션을 위한 기본 값은 다음에서 확인할 수 있다.

@[](/confs/play-jdbc/reference.conf)