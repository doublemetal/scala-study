# Stream
이글은 스칼라의 Stream 대하여 설명한 글이며, coursera의 스칼라 프로그래밍 강의를 참조하여 작성 하였습니다.

## Let's go find prime number

소수를 찾는 프로그램을 작성해 보겠습니다.

먼저, 소수인지 아닌지 결과를 리턴하는 함수를 작성합니다.

```scala
def isPrime(i:Int) : Boolean = {
      if (i <= 1)
          false
      else if (i == 2)
          true
      else
        !(2 to (i-1)).exists(x => i % x == 0)
}

isPrime(4) // false
isPrime(5) // true
```

1000부터 10000까지의 숫자중 첫번채 소수는..어떻게 구할 수 있을까요..?

아래와 같이 간단하게 작성 할 수 있습니다.

```scala
(1000 to 10000).filter(isPrime(_))(0) //1009
```

이 코드의 문제점은 무엇일까요...??

소수인 1009를 찾기위해 불필요한 1000 ~ 10000까지의 리스트를 생성하게 됩니다..

리스트를 전체 생성하지 않고, 첫번째 소수를 찾을 수 있는 방법은 무엇일까요..??

재귀함수를 통해 구현해보도록 하겠습니다.

```scala
def nthPrime(from: Int, to: Int, i: Int):Int = {
  if(from >= to) throw new Error ("NOT FOUND")
  else if(isPrime(from)){
    if (i == 1) from
    else
      nthPrime(from + 1 , to  , i-1)
  }else{
    nthPrime(from + 1 , to  , i)
  }
}

nthPrime(1 , 100 , 1)
```
재귀함수를 사용해서 숫자 배열은 생성하지 않았지만..

이전 코드에 비하면 너무 복잡해 보입니다.

스칼라의 Stream을 이용해서 해결해 보도록 하겠습니다.

## Stream
Stream 아래와 같이 다양한 방법으로 생성 할 수 있습니다.
```scala
val x  = Stream.cons(1 , Stream.cons(2 , Stream.Empty))
var x2 = Stream(1,2)
(1 to 10000).toStream
```
위의 세가지 구문의 결과는....?? Stream(1,?)라고 출력됩니다.

toList와 비교해 보도록 하겠습니다.

```scala
(1 to 10000).toStream // res2: scala.collection.immutable.Stream[Int] = Stream(1, ?)
(1 to 10000).toList // res3: List[Int] = List(1, 2, 3, 4,... 
```
Stream은 tail이 계산되어야 하는 시점까지 평가되지 않습니다.^^

이 말의 의미에 대해서는 차차 알아보도록 하겠습니다.

이제 , Stream을 이용해서 소수를 구하는 프로그램을 작성해 보도록 하겠습니다.

```scala
(1000 to 10000).toStream.filter(isPrime(_))(0) //Int = 1009
``` 

"toStream"이 추가된 것밖에 다른 것이 없지만...

1000 ~ 10000까지의 숫자 배열을 만들지 않고 1009(첫번째 소수)까지만 배열을

생성하게 됩니다. 

Why??

## Implementation of Stream

Stream은 List와 유사한 형태로 구현되어 있습니다.
```scala
object Stream{
  def cons[T](hd:T , tl: => Stream[T]) = new Stream[T]{
    def isEmpty = false;
    def head  =  hd
    def tail = tl
  }

  val emtpy = new Stream[Nothing] {
    def isEmpty = true;
    def head  =  throw new NoSuchElementException("ERROR")
    def tail = throw new NoSuchElementException("ERROR")
  }
}

trait Stream[+A] extends Seq[A]{
  def isEmpty : Boolean
  def head : A
  def tail : Stream[A]
}
```

List 클래스와 거의 동일하지만...

cons 함수의 파라미터 중 tl 부분의 선언이 지금까지의 파라미터와는 조금 다르게 생겼습니다.

그리고 이것이 List와 Stream의 다른점 이기도 합니다.

'=>'의 의미는 tail method를 누군가 호출 하기 전까지는 전달된 tl을 평가하지

않겠다라는 의미입니다.

(val tail이 아니라 , def tail이니까..함수죠?^^)

1000 ~ 10000까지 Stream을 선언하면..

'cons(1000 , cons(1001 , cons(cons(1002 , cons(..)))))' 이렇게

표현이 될텐데.. 그중 1000을 제외한 나머지는 'tail' 이죠..^^

그래서 , 아래와 같은 구문의 출력 값은 'Stream(1000, ?)'이 나옵니다.

(한번도 tail 함수가 호출되지 않아기 때문에..아직 평가 되지 않았습니다.)

```scala
val s = (1000 to 10000).toStream
println(s)
```

스칼라에서는 이것을 'by-name parameters' 라고 부릅니다.

그럼 같은 cons의 tail을 여러번 호출 된다면..어떻게 될까요.??

값을 바로 return 해주는 것이 아니라 호출 된 시점에 계속 '평가' 하게 됩니다.

'평가'라는 것의 의미가 조금 모호하지만... 바로 return 하는 것 보다는 

뭔가 비 효율 적일 것 같다는 느낌이 드시나요?^^

한번만 평가를 하고.. 그 이후에는 바로 그 값을 return 해주면...조금 더 효율 적

일 것 같은데...스칼라에서는 이미 그에 대한 방법을 제공하고 있습니다.

## Lazy evaluation
스칼라는  'by-named parameter'의 평가를 한번만 수행하도록 돕기 위해, 

lazy라는 키워드를 제공합니다. 아래의 코드가 어떻게 동작할 지 한번 생각해 보세요^^  

```scala
def expr: Unit ={
  val a = {println("a") ; 1}
  lazy val b = {println("b") ; 1}
  def c = {println("c") ; 1}
  a  + c + b + a + c + a
}
expr
```
정답은..'a c b c' 입니다.

expr 함수 호출 시, a변수는 바로 '평가'되어

1이 할당되고, lazy val b는 b 변수에 대한 참조가 있기 전까지는 '평가' 되지

않습니다.(바로 'b'가 화면에 출력되지 않은 이유입니다.)

c는 함수이기 때문에 호출 될때마다 'c' 문자를 화면에 출력하게 되고요..

위의 예제를 통해서, lazy 변수는 바로 평가 되지 않는다는 것을 알 수 있습니다.

so..what??

##Parameter vs val vs lazy val

lazy를 사용했을때, 어떻게 동작하는지 간단한 코드를 통해 확인 해보도록 하겠습니다. 

```scala
def printVal(x : => Int) :() => Int =  {
  def t() : Int = {
    x * x
  }
  t
}

val aa = printVal({println("A");10})
aa()
```
위의 코드의 출력은 어떻게 될까요..??
```scala
aa: () => Int = <function0>
A
A
res0: Int = 100
```
"x: => Int" 즉 x 파라미터가 참조 될때마다 '평가' 되게 됩니다.

한번만 평가하게 하려면..어떻게 해야 할까요..??

val 변수에 x값을 입력해보도록 하겠습니다.

```scala
def printVal(x : => Int) :() => Int =  {
  val a = x
  def t() : Int = {
    a * a
  }
  t
}

val aa = printVal({println("A");10})
aa()
```

위의 코드의 출력은 어떻게 될까요..??

```scala
A
aa: () => Int = <function0>
res0: Int = 100
```

우리가 원하는데로 'A'가 한번만 출력되었습니다.즉 x 변수의 평가가 한번만

처리되었습니다. ( 위에서 이야기한 lazy는 필요가 없나..??)

위의 코드에서 함수를 호출하는 aa() 부분을 삭제해보고 실행시켜보면

아래와 같이 출력 됩니다.

```scala
A
aa: () => Int = <function0>
```

???

'val a = x'가 실행되는 순간 x변수가 평가되어 'A'가 출력 됩니다..

이제 lazy를 사용해보도록 하겠습니다.

```scala
def printVal(x : => Int) :() => Int =  {
  lazy val a = x
  def t() : Int = {
    a * a
  }
  t
}

val aa = printVal({println("A");10})
```

이 코드는 아래와같이 출력 됩니다.

```scala
aa: () => Int = <function0>
```

x 파라미터가 아직 평가되지 않았습니다. 이후에  aa 함수를 호출하면..

```scala
A
res0: Int = 100
```

이제 x 파리미터를 평가하게 됩니다. 즉 lazy val에 할된된 값이 실제로

사용이 될때 평가하게 됩니다. 별 의미가 없어보이시나요?^^

아래와 같은 경우를 한번 생각 해보겠습니다.

```scala
def printVal(run:Boolean  , x : => Int) : Int =  {
  lazy val a = x
  if(run == true){
    a * a
  }else{
    0
  }
}

printVal(false ,
  {
    /*long and complicate work run for 10 minute*/
  println("A")
  ;10
  }
)
```
함수에 전달된 변수에 평가에 많은 리소스가 필요한 경우..

조건(run is true or not)에 따라 변수를 사용할 수도 있고,

사용하지 않을 수도 있는 경우라면 성능 향상을 위해서 사용하면 좋을 것 같습니다.

이것을 스칼라에선 lazy evaluation이라고 표현합니다.

(왜 '평가'라는 단어를 이용했는지..이해해주시길...)

지금까지 스칼라의 Stream과 lazy evaluation에 대해서 알아 보았습니다.