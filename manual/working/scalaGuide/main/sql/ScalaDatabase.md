<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# SQL 데이터베이스에 접근하기

## JDBC 접속 풀 설정하기

플레이는 JDBC 접속 풀을 설정할 수 있는 플러그인을 제공한다. 원하는 만큼의 데이터베이스를 설정할 수 있다.

데이터베이스 플러그인을 활성화 하기 위해서는, 빌드 의존성에 JDBC를 추가해야 한다.

```scala
libraryDependencies += jdbc
```

그리고 `conf/application.conf` 파일 안의 접속 풀을 설정해야 한다. 관습에 따르면 기본 JDBC 데이터의 소스는 `default` 라고 이름지어야 한다. 그리고 `db.default.driver`와 `db.default.url` 설정을 일치시켜야 한다.

만일 뭔가 올바르게 설정되어 있지 않다면, 바로 브라우져에서 확인할 수 있다.

[[images/dbError.png]]

> **주의:** JDBC URL 설정 값을 따옴표로 둘러싸야 할 필요가 있을 수 있다. 이후의 설정 문법에서는 ':' 가 예약 문자이다.

### H2 데이터베이스 엔진 접속 속성

```properties
# 메모리 내에서 동작하는 H2 데이터베이스 엔진을 사용하는 기본적인 데이터베이스 설정
db.default.driver=org.h2.Driver
db.default.url="jdbc:h2:mem:play"
```

```properties
# 보관 상태로 동작하는 H2 데이터베이스 엔진을 사용하는 기본적인 데이터베이스 설정
db.default.driver=org.h2.Driver
db.default.url="jdbc:h2:/path/to/db-file"
```

H2 데이터베이스 URL들의 상세한 내용은 [H2 Database Engine Cheat Sheet](http://www.h2database.com/html/cheatSheet.html)에서 확인할 수 있다.

### SQLite 데이터베이스 접속 속성

```properties
# SQLite 데이터베이스 엔진을 사용하는 기본적인 데이터베이스 설정
db.default.driver=org.sqlite.JDBC
db.default.url="jdbc:sqlite:/path/to/db-file"
```

### PostgreSQL 데이터베이스 엔진 접속 속성

```properties
# PostgreSQL 데이터베이스 엔진을 사용하는 기본적인 데이터베이스 설정
db.default.driver=org.postgresql.Driver
db.default.url="jdbc:postgresql://database.example.com/playdb"
```

### MySQL 데이터베이스 엔진 접속 속성

```properties
# MySQL 데이터베이스 엔진을 사용하는 기본적인 데이터베이스 설정
# playdb를 playdbuser로 접속하기
db.default.driver=com.mysql.jdbc.Driver
db.default.url="jdbc:mysql://localhost/playdb"
db.default.user=playdbuser
db.default.password="a strong password"
```

## 몇 가지 데이터 소스 설정하는 방법

```properties
# Orders database
db.orders.driver=org.h2.Driver
db.orders.url="jdbc:h2:mem:orders"

# Customers database
db.customers.driver=org.h2.Driver
db.customers.url="jdbc:h2:mem:customers"
```

## JDBC 드라이버 설정하기

플레이는 [H2](http://www.h2database.com) 데이터베이스 드라이버만 기본 패키지에 포함하고 있다. 그렇기 때문에, 실제로 배포하기 위해서는 데이터베이스 드라이버를 의존성에 추가해야한다.

예를 들면 만일 MySQL5를 사용한다면, 접속자를 위한 [[의존성 | SBTDependencies]]을 추가해야한다.

```scala
libraryDependencies += "mysql" % "mysql-connector-java" % "5.1.34"
```

또는 저장소들에서 드라이버를 찾을 수 있다면, 프로젝트의 [[관리되지 않는 의존성|Anatomy]] `lib` 폴더에 드라이버를 넣으면 된다.

## JDBC 데이터소스 접근하기

`play.api.db` 패키지는 데이터 소스들을 설정하기 위해서 접근할 수 있도록 해준다.

```scala
import play.api.db._

val ds = DB.getDataSource()
```

## JDBC 접속 획득하기

JDBC 접속을 얻는 몇가지 방법이 있다. 가장 간단한 방법은 아래와 같다.

```scala
val connection = DB.getConnection()
```

아래의 코드는 MySQL 5.*을 사용하는 매우 간단한 JDBC예를 보여준다.

```scala
package controllers
import play.api.Play.current
import play.api.mvc._
import play.api.db._

object Application extends Controller {

  def index = Action {
    var outString = "Number is "
    val conn = DB.getConnection()
    try {
      val stmt = conn.createStatement
      val rs = stmt.executeQuery("SELECT 9 as testkey ")
      while (rs.next()) {
        outString += rs.getString("testkey")
      }
    } finally {
      conn.close()
    }
    Ok(outString)
  }

}
```

하지만 접속 풀에 열었던 접속을 돌려놓기 위해서는, 어느 지점에서 `close()`를 호출해야 할 필요가 있다. 다른 방법은 플레이가 접속을 닫는 것을 관리하도록 하는 것이다.

```scala
// "default" 데이터베이스 접근
DB.withConnection { conn =>
  // 접속해서 할일을 수행한다.
}
```

기본 데이터베이스가 아닌 데이터베이스 사용하기.

```scala
// "default" 대신에 "orders" 데이터베이스 접근
DB.withConnection("orders") { conn =>
  // 접속해서 할일을 수행한다.
}
```

접속은 블럭이 끝나는 지점에서 자동으로 닫힐 것이다.

> **팁:** 접속을 잘 닫을 수 있도록, 각각의 `Statement`와 `ResultSet`는 이 접속을 사용해서 만들어진다.

다른 방법은 접속의 자동 제출을 `false` 로 설정하고, 구간에서 트렌젝션을 사용한다.

```scala
DB.withTransaction { conn =>
  // 접속해서 할일을 수행한다.
}
```

## 접속풀의 설정과 선택하기

틀에서 벗어나서 플레이는 [HikariCP](https://github.com/brettwooldridge/HikariCP)와 [BoneCP](http://jolbox.com/)인 두가지 접속 풀 구현을 제공한다. 기본은 HikariCP지만 `play.db.pool` 속성을 설정해서 변경할 수 있다.

```
play.db.pool=bonecp
```

접속 풀의 모든 설정 옵션들은 Play의 JDBC [`reference.conf`](resources/confs/play-jdbc/reference.conf)안에 있는 `play.db.prototype` 속성을 확인해서 찾을 수 있다.

## 테스트하기

메모리내의 데이터베이스 설정하기와 같은, 데이터베이스를 테스트하기 위한 정보는 [[데이터베이스 테스트하기|ScalaTestingWithDatabases]]에서 얻을 수 있다.
