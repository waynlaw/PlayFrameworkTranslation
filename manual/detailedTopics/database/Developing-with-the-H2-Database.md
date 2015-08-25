<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# H2 데이터 베이스

H2 인 메모리 데이터베이스는 개발을 위한 매우 편리한 데이터베이스다. 왜냐하면 플레이가 재시작될 때 이볼루션이 실행되기 때문이다. anorm을 사용하고 있다면 아마 계획하고 있는 프로덕션의 데이터베이스를 밀접하게 흉내낼 수 있는 것이 필요할 것이다. 예를들어 application.conf 파일에 데이터베이스 url을 위한 파라미터로 명시적으로 데이터베이스를 추가할 수 있다.

```
db.default.url="jdbc:h2:mem:play;MODE=MYSQL"
```

## 대상 데이터베이스

<table>
<tr>
<tr><td>MySql</td><td>MODE=MYSQL</td>
<td><ul><li>H2 does not have a uuid() function. You can use random_uuid() instead.  Or insert the following line into your 1.sql file: <pre><code>CREATE ALIAS UUID FOR 
"org.h2.value.ValueUuid.getNewRandom";</code></pre></li>  

<li>Text comparison in MySQL is case insensitive by default, while in H2 it is case sensitive (as in most other databases). H2 does support case insensitive text comparison, but it needs to be set separately, using SET IGNORECASE TRUE. This affects comparison using =, LIKE, REGEXP.</li></td></tr>
<tr><td>DB2</td><td>
MODE=DB2</td><td></td></tr>
<tr><td>Derby</td><td>
MODE=DERBY</td><td></td></tr>
<tr><td>HSQLDB</td><td>
MODE=HSQLDB</td><td></td></tr>
<tr><td>MS SQL</td><td>
MODE=MSSQLServer</td><td></td></tr>
<tr><td>Oracle</td><td>
MODE=Oracle</td><td></td></tr>
<tr><td>PostgreSQL</td><td>
MODE=PostgreSQL</td><td></td></tr>
</table>

## 인 메모리 DB 초기화 방지하기

H2는 어떤 연결이 없으면 여러분의 데이터베이스를 드롭시킨다. 아마 이런 상황이 벌어지길 원하지 않을 것이다. 이를 방지하기 위해 `jdbc:h2:mem:play;MODE=MYSQL;DB_CLOSE_DELAY=-1`처럼  세미콜론 구분자를 사용하여 url에 `DB_CLOSE_DELAY=-1`를 추가하자.

## H2 브라우저

플레이 콘솔에서 `h2-browser`를 입력하는 것을 여러분의 데이터베이스 항목을 살펴볼 수 있다. SQL 브라우저는 웹 브라우저로 실행될 것이다.

## H2 문서

더 많은 H2 문서는 [H2 데이터베이스](http://www.h2database.com/html/features.html) 사이트에서 확인할 수 있다.
