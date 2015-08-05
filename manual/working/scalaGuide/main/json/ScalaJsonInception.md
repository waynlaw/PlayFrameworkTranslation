<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# JSON 메크로 시작

> 이 문서는 초기에 [mandubian.com](http://mandubian.com/2012/11/11/JSON-inception/)의  ([@mandubian](https://github.com/mandubian))에서 Pascal Voitot에 의해서 초기에 작성되었다.


> **이 기능은 Scala의 메크로가 Scala 2.10.0에서 아직 실험중인 기능이기 때문에 아직 실험적이다. 만일 Scala의 실험적인 기능을 사용하지 않길 원한다면, 직접 작성된 동일한 기능의 Reads/Writes/Format를 사용해야 한다.**

## <a name="wtf-inception-boring">기본 케이스 클래스 Reads/Writes/Format를 작성하는 것은 몹시 지루하다!</a>

`Reads[T]`를 위한 케이스 클래스를 작성한 것을 떠올려 보자.

```
import play.api.libs.json._
import play.api.libs.functional.syntax._

case class Person(name: String, age: Int, lovesChocolate: Boolean)

implicit val personReads = (
  (__ \ 'name).read[String] and
  (__ \ 'age).read[Int] and
  (__ \ 'lovesChocolate).read[Boolean]
)(Person)
```

이 케이스 클래스를 위해서 4라인을 작성했다.
무엇을 알 수 있는가?
`Reads[TheirClass]`를 작성하는 것은 멋지지 않다고 생각하는 몇몇 사람들이 불만이 있다. 일반적으로 Jackson이나 Gson과 같은 Java JSON 프레임워크는 아무것도 작성하지 않아도 되기 때문이다. 이에 대해 Play2.1 JSON 직렬화/역직렬화에는 몇가지 논쟁이 있었다.

- 완전한 형식 안정성.
- 완전히 컴파일 되어야 할 것,
- 실행 시간에는 introspection/reflection를 전혀 하지 않을 것.

하지만 이런 점들이, 케이스 클래스를 위한 몇 줄의 코드를 작성해야하는 것을 정당화해주지는 않는다.

우리는 이것이 정말 좋은 방법이라고 생각했기 때문에 이 방법을 고수했다.

- JSON 간략화된 문법
- JSON 합성자
- JSON 변환자

기능을 추가했지만, 부가적인 4 줄에는 변화가 없었다.

## <a name="wtf-inception-minimalist">최소로 만들자.</a>
완벽주의자로써, 동일한 코드를 작성하는 새로운 방법을 제안한다.

```
import play.api.libs.json._
import play.api.libs.functional.syntax._

case class Person(name: String, age: Int, lovesChocolate: Boolean)

implicit val personReads = Json.reads[Person]
```

단 1 줄이다.
즉시 물어볼 질문은 아래와 같다.

> 실행 중 바이트코드 향상을 사용하는가? -> 아니다

> 실행 중 구조 결정을 사용하는가? -> 아니다

> 타입 안정성을 해치는가? -> 아니다

**어떻게 되는가?**

> **JSON coast-to-coast design**라는 전문 용어를 만든 다음, **JSON INCEPTION** 라고 하자.

<br/>
<br/>
# <a name="json-incept">JSON Inception</a>

## <a name="json-incept-eq">코드 동일성</a>
이전에 설명한 바와 같이,

```
import play.api.libs.json._
// import functional.syntax._를 추가하지 않고, 메크로가 관리하도록 한다.

implicit val personReads = Json.reads[Person]

// 아래를 작성하는 것과 동일하다.

implicit val personReads = (
  (__ \ 'name).read[String] and
  (__ \ 'age).read[Int] and
  (__ \ 'lovesChocolate).read[Boolean]
)(Person)
```	

## <a name="json-incept">Inception 동질성</a>

 _Inception_ 컨셉을 소개하는 방정식이 있다.

```
(Case Class INSPECTION) + (Code INJECTION) + (컴파일 시간) = INCEPTION
```

<br/>
####Case Class 조사
예상하는 바와 같이, 코드 동일성을 보장하기 위해서는 아래가 필요하다.

- `Person` 케이스 클래스 조사 
- `name`, `age`, `lovesChocolate` 3개의 필드와 타입을 추출
- 타입 클래스의 암묵적인 변환을 반영
- `Person.apply` 탐색


<br/>
####주입?
아니, 바로 중단해야 한다. 

>**Code 주입은 의존성 주입이 아니다.**  
>inception뒤에는 아무것도 없다… IOC도 아니고, DI도 아니고… 아니 아니 아니...)  
  
그 주입은 이제 즉각 IOC와 Spring과 연결되기 때문에, 이 항목을 의도적으로 사용했다. 하지만 이 단어를 실제 의미로 재정의 하는것이 좋겠다.
여기서 코드 주입은 단지 **컴파일 시간에 컴파일된 스칼라 AST(Abstract Syntax Tree)에 코드를 주입한 것**을 의미한다.

그러므로 컴파일 AST에서 `Json.reads[Person]`는 아래 처럼 컴파일되고 변환된다.

```
(
  (__ \ 'name).read[String] and
  (__ \ 'age).read[Int] and
  (__ \ 'lovesChocolate).read[Boolean]
)(Person)
```

더 적게도, 더 많이도 아니다.

<br/>
####컴파일 시간
모든 것은 컴파일 시간에 이루어진다.
실행 중 바이트 향상도 없고, 실행 중 구조 결정도 없다.

> 모든 것이 컴파일 시간에 이루어지기 때문에, 필드들의 모든 타입들에 대해 필요한 암묵적인 것들을 포함하지 않았다면 컴파일 에러를 얻을 것이다.

<br/>
# <a name="scala-macros">Json inception is Scala 2.10 메크로들</a>

스칼라의 다음 기능을 활성화 해야 한다.

- 컴파일 시간 코드 향상
- 컴파일 시간 클래스/암묵적 inspection
- 컴파일 시간 코드 주입

Scala 2.10에 포함된 새 실험적인 기능들로 활성화 됩니다. [Scala 메크로들](http://scalamacros.org/)  

스칼라 메크로들은 거대한 잠재력을 가진 (아직 실험중인)새 기능입니다. 이 기능을 이용해 아래의 것들을 할 수 있습니다.

- 컴파일 시간의 introspect 코드는 스칼라 리플렉션 API를 이용한다.
- 현재 컴파일 문맥에서 모든 import와 암시적인 것에 접근한다.
- 새 코드 표현을 생성하고, 컴파일 오류을 생성하며, 그들을 컴파일 체인에 추가한다.

아래의 항목을 주의해야 한다.

- **스칼라 메크로는 정확히 우리의 요구사항과 일치하기 때문에 이를 사용해야 한다.**
- **스칼라 메크로를 종단점이 아닌 활성자로 사용해야 한다.**
- **메크로는 당신이 작성할 수 있는 코드를 작성해주는 도우미다.**
- **이는 추가하지는 않지만, 기대하지 않은 코드는 장막에 가린다.**
- **우리는 *놀랍지 않게도* 원칙을 따른다.**

발견한 것과 같이, 메크로를 작성하는 것은 메크로 코드가 컴파일러 실행시점(또는 전역으로)에 실행될 때부터 시시한 과정이 아니다.

    그러므로 메크로 코드를 작성해야 한다. 
      컴파일되고 실행되는 
      코드를 생성하는 실행시간에
         컴파일되고 실행되도록 
         미래 시점에…           
**이 점이 *Inception* 이라고 부르는 이유이다.)**

그러므로 이 과정은 정확히 원하는 것을 따를 수 있도록 하는 마음의 수련을 필요로 한다. API는 매우 복잡하고 아직 완벽하게 문서화가 되어있진 않다. 그러므로, 반드시 메크로를 사용하기 전에 꾸준히 노력해야한다.

스칼라 메크로에 대해서는 말할 것이 많기 때문에, 이에 대한 다른 글도 작성하였다.
이 글은 또한 **스칼라 메크로를 사용하는 올바른 리플렉션 시작에 대한** 것이다.
큰 힘은 큰 책임을 의미한다. 그러므로 모두와 논의하는 게 좋으며, 약간의 매너가 필요하다.

<br/>
# <a name="writes-format">Writes[T] & Format[T]</a>

>JSON inception은 동일한 입/출력 형식을 가지는 `unapply/apply`함수를 가진 구조체에 사용할 수 있다는 것에 주의해야한다.

자연적으로, `Writes[T]`와 `Format[T]`또한 _포함_할 수 있다.

## <a name="writes">Writes[T]</a>

```
import play.api.libs.json._
 
implicit val personWrites = Json.writes[Person]
```

## <a name="format">Format[T]</a>

```
import play.api.libs.json._

implicit val personWrites = Json.format[Person]
```

## <a name="cando">특별한 패턴들</a>

- **컴페니언 오브젝트에 Reads/Writes를 선언할 수 있다.**
이것은 암묵적인 Reads/Writes가 암시적으로 추론되자마자 클래스의 인스턴스를 조작할 수 있기 때문에 유용하다.

```
import play.api.libs.json._

case class Person(name: String, age: Int)

object Person{
  implicit val personFmt = Json.format[Person]
}
```

- **단일 필드의 케이스 클래스를 위한 Reads/Writes를 선언할 수 있다.** (2.1-RC2까지는 알려진 제약이 있다.)

```
import play.api.libs.json._

case class Person(names: List[String])

object Person{
  implicit val personFmt = Json.format[Person]
}
```


## <a name="limitations">알려진 제약</a>

- **컴페니언 오브젝트의 apply 함수를 재정의 하면 안된다.** 매크로는 몇가지 apply 함수들이 있으면 선택하지 못한다.
- **Json 메크로는 apply와 unapply가 같은 입/출력 형식을 가져야 동작한다.**: 이는 캐이스 클래스를 위한 자연적인 경우이다. 하지만 만일 동일한 트레잇을 원한다면, 동일한 apply/unapply 를 케이스 클래스에 구현해야 한다.
- **Json 메크로는 Option/Seq/List/Set&Map[String, _]을 수용하도록 되어 있다.** 다른 일반적인 형식에도 테스트후에 동작하지 않는다면, 예전의 방식으로 Reads/Writes를 직접 작성해야 한다.

