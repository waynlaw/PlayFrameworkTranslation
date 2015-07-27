<!--- Copyright (C) 2009-2015 Typesafe Inc. <http://www.typesafe.com> -->
# <a name="json-to-json">JSON 변환</a>

> 이 문서는 초기에 [mandubian.com](http://mandubian.com/2012/10/29/unveiling-play-2-dot-1-json-api-part3-json-transformers/)에서 Pascal Voitot [@mandubian](https://github.com/mandubian))에 의해 작성되었다.

이제 JSON을 검증하는 법과자, Scala로 작성한 다른 구조체로 변경하고 다시 JSON으로 돌려놓는 법에 대해서 알아보았다. 하지만 이런 조합자들을 이용하여 웹 개발을 시작하면 JSON을 네트워크에서 읽어오거나, 검증하고 다시 JSON으로 만드는 것과 같은 새로운 문제에 직면하게 된다.

## <a name="json-to-json">JSON _coast-to-coast_ 디자인 소개</a>

### <a name="doomed-to-OO">JSON을 OO로 변경해야만 하는 불운한 운명인가?</a>

최신 몇년간의 대부분의 웹 프레임워크에서 (최근의 JSON을 기본 데이터 구조로 가지는 몇몇 JS 서버측 프레임워크를 제외하고), 우리는 네트워크에서 JSON을 받아서 **JSON을 OO 클래스들과 같은 (스칼라에서는 케이스 클래스) 구조체로 변경**했다. 왜 일까?

- 좋은 이유 : **OO 구조체는 "언어에서의 기본"** 이며, 웹 계층에서 비지니스 로직의 독립성을 보장하면서, **데이터를 비지니스 로직을 고려하며** 다룰 수 있다.
- 의문의 여지가 있는 이유 : **ORM 프레임워크는 DB를 다룰 때 OO 구조체만을 활용하며,** 우리는 이미 알고 있는 ORM들의 좋고 나쁜 기능들을 기반으로 다른 방식은 없다고 스스로 확인하고 있기 때문이다. (여기에 대한 비판은 일단 접어두자.)


### <a name="is-default-case">OO 변환이 정말 기본일까?</a>

**많은 경우에, 데이터를 가지고 실제 비지니스 로직을 수행하는 것이 필요한게 아니라, 데이터를 보관하기 전이나 추출한 후에 검증과 변환이 필요할 것이다.**  

CRUD 경우를 보자.

- 데이터를 네트워크에서 받아온 경우라면, 데이터를 검증하고 DB에 추가하거나 갱신할 것이다.
- 만일 다른 경우라면, 데이터를 DB에서 가져와서 외부로 전송할 것이다.

그러므로 일반적으로 CRUD 연산을 위해서는 프레임워크가 오직 OO만을 이용할 수 있기 때문에, JSON을 OO구조체로 변환해야 한다.

>**JSON을 OO로 변환하면 안된다고 하지 않았고, 이미 그러지 않는 경향이 있다고 해서, 위의 경우가 대부분의 경우이고 비지니스 로직을 채워넣는 용도로만 변환을 사용해야 한다는 것은 아니다.**

### <a name="new-players">JSON을 다루는 방법을 바꾸는 새로운 기술</a>

위와 같은 형식 이외에도, 우리는 Mongo(또는 CouchDB)와 같이 JSON 구조(Binary JSON과 같은)와 거의 같아보이는 문서 구조를 수용하는 몇가지 새로운 DB형식을 가지고 있습니다.
이러한 DB 형식에서는, Mongo에서나 Mongo로 매우 자연스러운 방식으로 스트림 데이터와의 반응형 환경을 제공해주고 [ReactiveMongo](http://www.reactivemongo.org)와 같은 좋은 도구가 있습니다.
[Play2-ReactiveMongo module](https://github.com/zenexity/Play-ReactiveMongo)를 작성하는 동안, ReactiveMongo를 Play2.1과 통합하기 위해 Stephane Godbillon와 작업해 왔습니다. Play2.1을 위한 Mongo 도구들을 떠나, 이 모듈은 _BSON 으로 부터, 또는 BSON으로의 형식 변환_을 제공합니다.

> **DB에서 또는 DB로 직접적으로 OO로의 변환 없이 JSON 흐름을 다룰 수 있음을 의미한다.**

### <a name="new-players">JSON _coast-to-coast_ 디자인</a>

계정에 대해서는, 아래의 것들을 쉽게 생각할 수 있다.

- JSON 받기,
- JSON 검증,
- DB문서 구조에 맞도록 JSON 변환,
- JSON을 DB에 직접 전달. (또는 다른 방식으로)

이는 DB에서 데이터를 가져오는 경우와 완전히 동일하다.

- 데이터를 DB에서 JSON으로 바로 가져온다.
- 클라이언트가 필요로 하는 형식에 맞춘 데이터가 있는 경우만을 전달하기 위해서 JSON을 변환하거나 거른다. (예: 일부 보안자료는 나가지 않기를 바랄 수 있다.)
- 직접적으로 JSON을 클라이언트로 전송한다.

이런 경우에 우리는 클라이언트에서 DB로 **JSON 데이터 흐름 다루기**와, JSON에 대한 어떤 변화없이 돌려놓는 것을 쉽게 생각해 볼 수 있다.
일반적으로는, 이 변환 흐름을 **Play2.1이 제공하는 반응형 구조**에서 다루면, 새로운 지평을 느낄 수 있을 것이다.

> 이것이 **JSON coast-to-coast 디자인**이라 불린다. 
> 
> - JSON 데이터를 구간별로 고려하지 않고, **서버를 지나는 클라이언트에서 DB(또는 다른 것들)로의 지속적인 데이터의 흐름**으로 다룬다,
> - 변환과 같은 수정을 적용할 때, **JSON 흐름을 다른 파이프들과 연결된 파이프**와 같이 다룬다,
> - **완전한 비동기/멈추게 하지않는** 방식으로 흐름을 다룬다.
>
> 이는 Play2.1 반응형 구조를 사용하는 또 하나의 이유가 된다…  
> 일반적인 웹앱의 디자인 하는 방식을 과감하게 데이터 변경의 흐름으로 고려하기를 기대한다. 이는 고전적인 구조에 비해 오늘날의 웹앱의 요구에 보다 더 잘 맞는 함수형 구조에 더 잘 맞을 것이다. 어쨋든 이 장에서 다루고자 하는 내용은 아니다. ;)

그리고 스스로 발견한 것과 같이, 검증과 변환을 직접 하는 것에 기반을 둔 Json 흐름을 다루는 것이 가능하기 위해서는 새로운 도구가 필요하다. JSON 합성자는 좋은 후보이지만, 이는 너무나도 일반적이다. 그렇기 때문에 이를 위해 **JSON 변환자**라는 특수한 결합자 API를 만들었다.

<br/>
# <a name="json-transf-are-reads">JSON 변환자인 `Reads[T <: JsValue]`</a>

JSON 변환자를 단지 `f:JSON => JSON`라 할 수 있다.
그러면 또 다시 JSON변환자를 간단히 `Writes[A <: JsValue]`로 할 수 있다.
하지만, JSON 변환자는 단지 함수일 뿐 아니라, 우리가 이전에 이야기 한 것처럼 우리는 변환하는 동안 JSON의 검증을 원한다.
결과적으로 JSON 변환자는 `Reads[A <: Jsvalue]`이다.

> **Reads[A <: JsValue]가 읽고 검증하는 것이 가능할 뿐만 아니라, 변환도 가능하다는 것을 상기해야 한다.**

## <a name="step-pick">`JsValue.validate` 대신 `JsValue.transform` 사용하기 </a>

`Reads[T]`가 단순한 검증자가 아니라 변환자라는 것을 고려할 수 있도록 하기 위해 `JsValue`에서 함수 도우미를 제공하였다.

`JsValue.transform[A <: JsValue](reads: Reads[A]): JsResult[A]`

이는 `JsValue.validate(reads)`와 완전히 동일하다.

## 세부 사항

예제코드는 아래에 나오며, 아래의 JSON을 사용 할 것이다.

```
{
  "key1" : "value1",
  "key2" : {
    "key21" : 123,
    "key22" : true,
    "key23" : [ "alpha", "beta", "gamma"]
    "key24" : {
      "key241" : 234.123,
      "key242" : "value242"
    }
  },
  "key3" : 234
}
```

## <a name="step-pick">경우 1: JsPath에서 JSON값 고르기</a>


### <a name="step-pick-1">JsValue로 값 고르기</a>

```
import play.api.libs.json._

val jsonTransformer = (__ \ 'key2 \ 'key23).json.pick

scala> json.transform(jsonTransformer)
res9: play.api.libs.json.JsResult[play.api.libs.json.JsValue] = 
	JsSuccess(
	  ["alpha","beta","gamma"],
	  /key2/key23
	)	
```

####`(__ \ 'key2 \ 'key23).json...` 
  - 모든 JSON 변환자는 `JsPath.json.`에 있다.

####`(__ \ 'key2 \ 'key23).json.pick` 
  - `pick`은 주어진 경로 **안**에서 선택된 값인 `Reads[JsValue]` 이다. 여기서는 `["alpha","beta","gamma"]` 이다.
  
####`JsSuccess(["alpha","beta","gamma"],/key2/key23)`
  - 이는 단지 성공한 `JsResult` 이다.
  - 예를 들어, `/key2/key23` JsPath는 데이터를 가져올 곳을 표현하지만, 그것을 다루지는 않는다. 주로 Play API에서 JsResult(s)의 결합을 위해 사용된다.
  - 덮어쓴 `toString` 때문에 `["alpha","beta","gamma"]`로 나온다.

<br/>
> **Reminder** 
> `jsPath.json.pick` JsPath내의 한 값만을 얻는다.
<br/>


### <a name="step-pick-2">형식만으로 값 선택하기</a>

```
import play.api.libs.json._

val jsonTransformer = (__ \ 'key2 \ 'key23).json.pick[JsArray]

scala> json.transform(jsonTransformer)
res10: play.api.libs.json.JsResult[play.api.libs.json.JsArray] = 
	JsSuccess(
	  ["alpha","beta","gamma"],
	  /key2/key23
	)

```

####`(__ \ 'key2 \ 'key23).json.pick[JsArray]` 
  - `pick[T]`은 주어진 JsPath 경로 **안**에서 우리가 선택한 값(예제에서는 `JsArray`)인 `Reads[T <: JsValue]`이다.
  
<br/>
> **Reminder**
> `jsPath.json.pick[T <: JsValue]`는 JsPath에서 오직 형식적인 값을 추출한다.

<br/>

## <a name="step-pickbranch">Case 2: JsPath에서 분기 선택하기</a>

### <a name="step-pickbranch-1">분기를 JsValue로 선택하기</a>

```
import play.api.libs.json._

val jsonTransformer = (__ \ 'key2 \ 'key24 \ 'key241).json.pickBranch

scala> json.transform(jsonTransformer)
res11: play.api.libs.json.JsResult[play.api.libs.json.JsObject] = 
  JsSuccess(
	{
	  "key2": {
	    "key24":{
	      "key241":234.123
	    }
	  }
	},
	/key2/key24/key241
  )

```

####`(__ \ 'key2 \ 'key23).json.pickBranch` 
  - `pickBranch`는 최상단에서 주어진 JsPath로의 가지를 선택한 `Reads[JsValue]`이다.
  
####`{"key2":{"key24":{"key242":"value242"}}}`
  - 최상단에서 주어진 JsPath로의 가지에서 JsPath내의 JsValue를 포함한 결과이다.
  
<br/>
> **되새기기:**
> `jsPath.json.pickBranch`는 JsPath로의 단일 가지룰 추출 + JsPath내의 값

<br/>


## <a name="step-copyfrom">경우 3: 입력된 JsPath에서 값을 새 JsPath로 복사하기</a>

```
import play.api.libs.json._

val jsonTransformer = (__ \ 'key25 \ 'key251).json.copyFrom( (__ \ 'key2 \ 'key21).json.pick )

scala> json.transform(jsonTransformer)
res12: play.api.libs.json.JsResult[play.api.libs.json.JsObject] 
  JsSuccess( 
    {
      "key25":{
        "key251":123
      }
    },
    /key2/key21
  )

```

####`(__ \ 'key25 \ 'key251).json.copyFrom( reads: Reads[A <: JsValue] )` 
  - `copyFrom`는 `Reads[JsValue]`이다.
  - `copyFrom`는 제공된 Reads[A]를 이용해서 받은 JSON에서 JsValue를 읽는다.
  - `copyFrom`는 추출된 JsValue를 주어진 JsPath에 대응되는 새 가지의 말단으로 복사한다.
  
####`{"key25":{"key251":123}}`
  - `copyFrom`는 값 `123`을 읽는다.
  - `copyFrom`는 이 값을 새로 가지 `(__ \ 'key25 \ 'key251)`로 복사한다.

> **되새기기:**
> `jsPath.json.copyFrom(Reads[A <: JsValue])`는 주어진 입력 JSON에서 값을 읽고, 말단으로써의 결과를 가진 새 가지를 생성한다.

<br/>
## <a name="step-update">Case 4: 전체 입력 Json을 복사하고 가지를 갱신하기</a>

```
import play.api.libs.json._

val jsonTransformer = (__ \ 'key2 \ 'key24).json.update( 
	__.read[JsObject].map{ o => o ++ Json.obj( "field243" -> "coucou" ) }
)

scala> json.transform(jsonTransformer)
res13: play.api.libs.json.JsResult[play.api.libs.json.JsObject] = 
  JsSuccess(
    {
      "key1":"value1",
      "key2":{
        "key21":123,
        "key22":true,
        "key23":["alpha","beta","gamma"],
        "key24":{
          "key241":234.123,
          "key242":"value242",
          "field243":"coucou"
        }
      },
      "key3":234
    },
  )

```

####`(__ \ 'key2).json.update(reads: Reads[A < JsValue])` 
- `Reads[JsObject]`이다.

####`(__ \ 'key2 \ 'key24).json.update(reads)` 는 3가지를 수행한다. 
- 입력된 JSON에서 JsPath `(__ \ 'key2 \ 'key24)`의 값을 추출한다.
- `reads`를 관계값에 적용하고, `(__ \ 'key2 \ 'key24)`에 `reads`의 결과를 말단으로 추가하여 가지를 재 생성한다.
- 이 가지를 전체 입력된 JSON 기본의 가지들의 변환과 합친다. (그러므로, 이는 입력된 JsObject와만 동작하며, JsValue의 다른 형식과는 동작하지 않는다.)

####`JsSuccess({…},)`
- 단지 정보를 위한 것이다. JsPath의 최 상위에서 JSON을 다루는 것이 끝났기 때문에, 두번째 파라메터로 JsPath를 넣지 않는다.

<br/>
> **되새기기:**
> `jsPath.json.update(Reads[A <: JsValue])`는 오직 JsObject와만 동작한다. 모든 입력된 `JsObject`를 복사하고 jsPath를 주어진 `Reads[A <: JsValue]`로 갱신한다.


<br/>
## <a name="step-put">경우 5: 주어진 값을 새 가지에 넣기</a>

```
import play.api.libs.json._

val jsonTransformer = (__ \ 'key24 \ 'key241).json.put(JsNumber(456))

scala> json.transform(jsonTransformer)
res14: play.api.libs.json.JsResult[play.api.libs.json.JsObject] = 
  JsSuccess(
    {
      "key24":{
        "key241":456
      }
    },
  )

```

####`(__ \ 'key24 \ 'key241).json.put( a: => JsValue )` 
- 는 Reads[JsObject]이다.

####`(__ \ 'key24 \ 'key241).json.put( a: => JsValue )`
- 새 가지 `(__ \ 'key24 \ 'key241)`를 만든다.
- `a`를 이 가지의 말단으로 넣는다.

####`jsPath.json.put( a: => JsValue )`
- 이름으로 전달된 닫힌 경우에도 통과를 허가해주는 JsValue 전달인자를 받는다.

####`jsPath.json.put` 
- 입력된 JSON의 모든 부분을 다루지는 않는다.
- 단순히 입력된 JSON을 주어진 값으로 변환한다.

<br/>
> **되새기기:**
> `jsPath.json.put( a: => Jsvalue )`는 입력 JSON을 받지 않고, 주어진 값으로 새 가지를 생성한다.

<br/>
## <a name="step-prune">경우 6: 입력된 JSON의 가지를 간결하게 만들기</a>

```
import play.api.libs.json._

val jsonTransformer = (__ \ 'key2 \ 'key22).json.prune

scala> json.transform(jsonTransformer)
res15: play.api.libs.json.JsResult[play.api.libs.json.JsObject] = 
  JsSuccess(
    {
      "key1":"value1",
      "key3":234,
      "key2":{
        "key21":123,
        "key23":["alpha","beta","gamma"],
        "key24":{
          "key241":234.123,
          "key242":"value242"
        }
      }
    },
    /key2/key22/key22
  )

```

####`(__ \ 'key2 \ 'key22).json.prune` 
- 는 JsObject에만 동작하는 `Reads[JsObject]`이다.

####`(__ \ 'key2 \ 'key22).json.prune`
- 입력된 JSON에서 주어진 JsPath를 제거한다. (`key2`에서 `key22`를 제거한다.)

결과인 JsObject는 입력된 JsObject와 동일한 키의 순서를 가지지 않는다는 것에 주의하자. 이는 JsObject의 구현과 합성 과정 때문이다. 그러나 이것은 `JsObject.equals`를 재정의 했기 때문에 중요하지 않다.

> **Reminder:**
> `jsPath.json.prune`는 오직 JsObject와만 동작하며, 주어진 JsPath를 입력된 JSON에서 제거한다. 
> 
> 아래의 사항에 주의하자.
> - `prune` 는 당분간 재귀로 구성된 JsPath에는 동작하지 않는다.
> - `prune`이 제거해야할 가지를 찾지 못하면, 어떠한 오류도 생성하지 않고 변경되지 않은 JSON을 반환한다.

# <a name="more-complicated">더 복잡한 경우</a>

## <a name="more-complicated-pick-update">경우 7: 2 장소에서 가지를 선택하고 내용을 갱신하기</a>

```
import play.api.libs.json._
import play.api.libs.json.Reads._

val jsonTransformer = (__ \ 'key2).json.pickBranch(
  (__ \ 'key21).json.update( 
    of[JsNumber].map{ case JsNumber(nb) => JsNumber(nb + 10) }
  ) andThen 
  (__ \ 'key23).json.update( 
    of[JsArray].map{ case JsArray(arr) => JsArray(arr :+ JsString("delta")) }
  )
)

scala> json.transform(jsonTransformer)
res16: play.api.libs.json.JsResult[play.api.libs.json.JsObject] = 
  JsSuccess(
    {
      "key2":{
        "key21":133,
        "key22":true,
        "key23":["alpha","beta","gamma","delta"],
        "key24":{
          "key241":234.123,
          "key242":"value242"
        }
      }
    },
    /key2
  )

```

####`(__ \ 'key2).json.pickBranch(reads: Reads[A <: JsValue])` 
- 입력된 JSON에서 가지 `__ \ 'key2`를 추출하고 `reads`를 이 가지의 상대적인 말단(오직 내용에 대한)에 적용한다.

####`(__ \ 'key21).json.update(reads: Reads[A <: JsValue])` 
- `(__ \ 'key21)`가지를 갱신한다.

####`of[JsNumber]` 
- 단지 `Reads[JsNumber]` 이다.
- `(__ \ 'key21)`에서 JsNumber를 추출한다.

####`of[JsNumber].map{ case JsNumber(nb) => JsNumber(nb + 10) }` 
- JsNumber를 읽는다. (`__ \ 'key21` 내의 _value 123_)
- 값을 _10_ 증가시키기 위해서 `Reads[A].map`를 사용한다. (값을 갱신하지 않는 방식이 자연적이다.)

####`andThen` 
- 두 `Reads[A]` 의 결합이다.
- 첫번째 읽기가 적용되고, 그 결과는 두변째 읽기에 연결된다.

####`of[JsArray].map{ case JsArray(arr) => JsArray(arr :+ JsString("delta")` 
- JsArray를 읽는다. (`__ \ 'key23` 내의 값 [alpha, beta, gamma])
- `JsString("delta")`를 덧붙이기 위해서 `Reads[A].map`를 사용한다.

>이 가지를 선택했기 때문에 결과가 단지 `__ \ 'key2` 가지인 것에 주의하자.

<br/>
## <a name="more-complicated-pick-prune">경우 8: 가지를 선택하고 부가지들을 간결하게 하기</a>

```
import play.api.libs.json._

val jsonTransformer = (__ \ 'key2).json.pickBranch(
  (__ \ 'key23).json.prune
)

scala> json.transform(jsonTransformer)
res18: play.api.libs.json.JsResult[play.api.libs.json.JsObject] = 
  JsSuccess(
    {
      "key2":{
        "key21":123,
        "key22":true,
        "key24":{
          "key241":234.123,
          "key242":"value242"
        }
      }
    },
    /key2/key23
  )

```

####`(__ \ 'key2).json.pickBranch(reads: Reads[A <: JsValue])` 
- 입력된 JSON에서 가지 `__ \ 'key2`를 추출하고, 이 가지의 상대 말단에 `reads`를 적용한다. (내용에만 적용한다.)


####`(__ \ 'key23).json.prune` 
- 가지 `__ \ 'key23`를 상대 JSON에서 제거한다.

>결과가 단지 `key23` 항목이 없는 `__ \ 'key2`가지 임에 주의하자.


# <a name="combinators">결합자에 대해선 어떨까?</a>

이제 슬슬 지루해지기 떄문에, 위의 내용들을 멈추겠다. (아직 아니더라도)…

단지 일반적인 JSON 변환을 할 수 있는 큰 도구가 생겼다는 것을 마음에 담아두길 바란다.  
합성하고 map, flatmap변환을 다른 변환자들과 함께 사용할 수 있다. 이 가능성은 거의 무한대이다.

하지만 마지막으로 집고 넘어갈 것이 있다. 이런 훌륭한 새 JSON 변환자들을 이전에 보여주었던 Reads 합성자들과 함께 섞어 사용하는 것이.
이것은 JSON 변환자가 단지 `Reads[A <: JsValue]`인 것처럼 순회한다.

__Gizmo to Gremlin__ JSON 변환자에 대한 예를 들어보자.

Gizmo는 아래와 같다.

```
val gizmo = Json.obj(
  "name" -> "gizmo",
  "description" -> Json.obj(
    "features" -> Json.arr( "hairy", "cute", "gentle"),
    "size" -> 10,
    "sex" -> "undefined",
    "life_expectancy" -> "very old",
    "danger" -> Json.obj(
      "wet" -> "multiplies",
      "feed after midnight" -> "becomes gremlin"
    )
  ),
  "loves" -> "all"
)
```

Gremlin은 아래와 같다.

```
val gremlin = Json.obj(
  "name" -> "gremlin",
  "description" -> Json.obj(
    "features" -> Json.arr("skinny", "ugly", "evil"),
    "size" -> 30,
    "sex" -> "undefined",
    "life_expectancy" -> "very old",
    "danger" -> "always"
  ),
  "hates" -> "all"
)
```

이 변환을 해주는 JSON 변환자를 작성해보자.

```
import play.api.libs.json._
import play.api.libs.json.Reads._
import play.api.libs.functional.syntax._

val gizmo2gremlin = (
	(__ \ 'name).json.put(JsString("gremlin")) and
	(__ \ 'description).json.pickBranch(
		(__ \ 'size).json.update( of[JsNumber].map{ case JsNumber(size) => JsNumber(size * 3) } ) and
		(__ \ 'features).json.put( Json.arr("skinny", "ugly", "evil") ) and
		(__ \ 'danger).json.put(JsString("always"))
		reduce
	) and
	(__ \ 'hates).json.copyFrom( (__ \ 'loves).json.pick )
) reduce

scala> gizmo.transform(gizmo2gremlin)
res22: play.api.libs.json.JsResult[play.api.libs.json.JsObject] = 
  JsSuccess(
    {
      "name":"gremlin",
      "description":{
        "features":["skinny","ugly","evil"],
        "size":30,
        "sex":"undefined",
        "life_expectancy":
        "very old","danger":"always"
      },
      "hates":"all"
    },
  )
```

위의 부분이 이해가 가능하다고 가정하고, 위의 모든 내용을 설명하지는 않을 것이다.
단지 아래의 항목을 되새기자.

####`(__ \ 'features).json.put(…)`는 `(__ \ 'size).json.update`이후이기 때문에 원래의 `(__ \ 'features)`를 덮어쓴다.
<br/>
####`(Reads[JsObject]와 Reads[JsObject]) 줄이기` 
- 이것은 `Reads[JsObject]` 양쪽의 결과를 모두 합성한다. (JsObject ++ JsObject)
- 이는 또한 첫번째 결과 읽어서 두번째에 적용하는 `andThen`와는 다르게 동일한 JSON을 양쪽 `Reads[JsObject]`에 적용한다.
