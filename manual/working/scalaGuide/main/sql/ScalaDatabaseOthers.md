<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# 다른 데이터베이스 라이브러리들과 통합하기

어떤 **SQL** 데이터베이스에 접근하는 라이브러리도 플레이와 함께 사용할 수 있고, `play.api.db.DB` 도우미를 통해 `Connection`과 `Datasource`로 변환할 수 있다.

## ScalaQuery와 통합하기

어떤 JDBC 데이터 소스를 필요로 하는 어떤 JDBC 접근 계층과도 통합할 수 있다. [ScalaQuery](https://github.com/szeiger/scala-query)와 통합하는 예제는 아래와 같다.

```scala
import play.api.db._
import play.api.Play.current

import org.scalaquery.ql._
import org.scalaquery.ql.TypeMapper._
import org.scalaquery.ql.extended.{ExtendedTable => Table}

import org.scalaquery.ql.extended.H2Driver.Implicit._ 

import org.scalaquery.session._

object Task extends Table[(Long, String, Date, Boolean)]("tasks") {
    
  lazy val database = Database.forDataSource(DB.getDataSource())
  
  def id = column[Long]("id", O PrimaryKey, O AutoInc)
  def name = column[String]("name", O NotNull)
  def dueDate = column[Date]("due_date")
  def done = column[Boolean]("done")
  def * = id ~ name ~ dueDate ~ done
  
  def findAll = database.withSession { implicit db:Session =>
      (for(t <- this) yield t.id ~ t.name).list
  }
  
}
```

## JNDI를 통해 데이터 소스 노출하기

어떤 라이브러리들은 `Datasource` 참조를 JNDI에서 가져오기를 원할 수 있다. `conf/application.conf`에 설정을 추가하면, 플레이의 어떤 데이터 소스라도 JNDI를 통해서 노출할 수 있다.

```
db.default.driver=org.h2.Driver
db.default.url="jdbc:h2:mem:play"
db.default.jndiName=DefaultDS
```
