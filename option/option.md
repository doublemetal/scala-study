# Option
이글은 스칼라의 Option 대하여 설명한 글이며, 유투브 동영상(https://www.youtube.com/watch?v=Laq1vKi6-Ak)을 참조하여 작성 하였습니다.

## NullPointerException is Shit!!

프로그램을 작성하다보면, NullPointerException을 자주 만나게 됩니다.

(혹은 방어하는 코드를 자주 만나게 되죠^^)

아래 코드에서 바로 확인 해보도록 하겠습니다.

```scala
def toLower(l:List[String])  = l.map(_.toLowerCase)

val l1 = List("A" , "B" , "C" , "D")

println(toLower(l1)) // ['a' , 'b' , 'c' , 'd']

val l2 = List("A" , "B" , "C" , null , null , "D" , null)

println(toLower(l2)) // java.lang.NullPointerException
```

NullPointerException을 막기위해 우리가 하는것은..??

(아마 더 좋은 방법이 있을거 같습니다..전 초보라..)

아래와 같습니다.

```scala
def toLowerPreventNull(l:List[String]) : List[String]={
  var newL:List[String] = null;
  l.map(item => if(item != null){
    if(newL == null){
      newL = List(item.toLowerCase)
    }else{
      newL = newL ++ List(item.toLowerCase)
    }
  }
  )
  newL
}

println(toLowerPreventNull(l2)) //['a' , 'b' , 'c' , 'd']
```

더 좋은 방법은 없을까요..??

```scala
def toLowerPreventNull2(l:List[String])  =
  l.map(Option(_)).flatMap(x => x).map(_.toLowerCase)

println(toLowerPreventNull2(l2)) //['a' , 'b' , 'c' , 'd']
```

무슨일이 벌어진걸까요..??

천천히 알아보도록 하겠습니다.

## Scala Option is Some Or None

스칼라에서는 Option이라는 클래스를 제공 하고 있습니다.

Option은 trait이고 , Some과 None은 trait를 상속받은 객체 입니다.

```scala
val some = Option("A") // some: Option[String] = Some(A)
val none = Option(null) // none: Option[Null] = None
```

Option의 apply 메소드에 전달된 파라미터가 null이 아니면 Some을 , 

null이면 None을 리턴하도록 구현되어 있겠죠?^^


아까 위에서 본 예제를 처리하는 코드를 조금더 자세히 보도록 하겠습니다.

```scala
l.map(Option(_)).flatMap(x => x).map(_.toLowerCase)
```
위의 코드에서 map 부분만 분리하면..아래와 같습니다.

(l은 파라미터로 받은 변수였지만..아래 코드는 편의를 위해 직접 l2 변수를 사용하였습니다.)
```scala
l2.map(Option(_))
//List[Option[String]] = List(Some(A), Some(B), Some(C), None, None, Some(D), None)
```

위에서 살펴보았듯이.. not null은 Some , null은 None으로 변경한 List가 리턴 됩니다.

여기에 flatmap을 적용해 보면 아래와 같습니다.

```scala
l2.map(Option(_)).flatMap(x => x) // List(A, B, C, D)
```

앞에서 이야기 했던 flatMap의 정의를 생각해보면.. 위의 내용은 조금 이해가 안가는 부분입니다.

```scala
val ll = List(List(1,2,3) , List(4,5,6))
ll.flatMap(x => x.map(_*2))
```

List의 flatMap은 위와 같이 List[T]를 리턴하는 함수를 파라미터로 가지게 되어 있는데

"l2.map(Option(_)).flatMap(x => x)"에서 "x=>x" 부분은 List를 리턴하는

함수가 아닙니다. 아래 코드를 보면 확실히 이상하다는 것을 알 수 있습니다.

```scala
l2.map(Option(_)).flatMap(x => {print(x); x})
//Some(A) Some(B) Some(C) None None Some(D) None
```

이야기를 정리해보면...List의 flatMap은 List[T]를 리턴하는 함수를 파라미터로

가지게 되어있는데..위의 코드에서 "x=>x"는 List[T]를 반환하는 함수가 아닙니다.

그런데 정상적으로 실행이 된다면...누군가가 List[T] 타입의 형태로 변경 시켰다는 이야기 인데요..^^

implict!!(기억 나시나요^^)

아래 코드는 스칼라의 Option의 구현 소스입니다.

Option을 Iterable로 변환해주는 implict 함수를 가지고 있습니다.^^ 

```scala
object Option extends scala.AnyRef with scala.Serializable {
  implicit def option2Iterable[A](xo : scala.Option[A]) : scala.Iterable[A] = { /* compiled code */ }
  ... 
}
```

"x => x" 함수를 통해 return 될 수 있는 값은 Some(Int) , None 둘중에 하나입니다.

Some(Int)는 List[Int]를,  None은 Nil을 리턴하는 함수가 implict로 선언 되어 있다면

위의 flatMap에 대해서 가졌던 의문이 풀리게 됩니다.^^

그리고 이제 마지막으로 소문자로 변경하는 map을 작용하면...

```scala
l2.map(Option(_)).flatMap(x => x).map(_.toLowerCase)  // res3: List[String] = List(a, b, c, d)
```

소문자 List를 구할 수 있습니다.

마지막으로 위에서 설명한 Option을 List로 바꿔주는 함수를 직접 작성해보도록 하겠습니다.

```scala
def optionToList[T](opt:Option[T]) : List[T] = opt match{
  case Some(a) => List(a)
  case None => Nil
}

optionToList(Some(1)) // List(1)
optionToList(None) // List()

```

Pattern Matching을 통해서 구현해보았는데요..위의 코드가 이해가 되신다면,

Option이 가지고 있는 함수인 get , getOrElse , isDefined와 같은 함수들도

쉽게 이해 하실 수 있습니다.

지금까지 스칼라의 Option에 대해서 알아보았습니다.