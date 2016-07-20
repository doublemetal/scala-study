# Monad
이글은 Monad 대하여 설명한 글이며, coursera의 스칼라 프로그래밍 강의를 참조하여 작성하였습니다.

## What is Monad

Monad란??
(https://en.wikipedia.org/wiki/Monad_(functional_programming))

- 간단한 컴포넌트들을 조합하여 프로그램을 만드는 방법
- 순서대로 정의된 계산식을 표현하는 구조
- 프로그래머가 순서대로 데이타를 처리 할 수 있는 파이프라인을 만들 수있도록 해줌

(직접 위키를 보시면 더 잘 이해가 되실 것 같네요.^^)

그런데... 이런거 우리 해보지 않았나요..??

```scala
for{
  x <- t1
  y <- t2
  if ( x+y ) % 2 == 0
}yield (x,y)
```
- flatmap과 filter 그리고 map을 조합하여 프로그램을 만들었음
- flatmap과 filter 그리고 map의 순서를 정의하여 프로그램
- t1의 x를 다음 프로세스의 입력으로 사용(pipeline)

우리도 Monad를 사용하고 있습니다.!!

이렇게 넘어가기는 찝찝하니까... Monad에 대해서 조금더 자세히 알아보도록 하겠습니다.

## Formal definition about Monad

1. A type constructor that defines, for every underlying type,how to obtain a corresponding monadic type.
2. A unit function that injects a value in an underlying type to a value in the corresponding monadic type. The unit function has the polymorphic type t→M t. The result is normally the "simplest" value in the corresponding type that completely preserves the original value (simplicity being understood appropriately to the monad). 
3. A binding operation of polymorphic type (M t)→(t→M u)→(M u).ts first argument is a value in a monadic type, its second argument is a function that maps from the underlying type of the first argument to another monadic type, and its result is in that other monadic type

위의 내용은 위키 페이지에 Monad의 정의 입니다.(영어 압박이네요..ㅠㅠ)

차근차근 하나씩 살펴보도록 하겠습니다.

1번 내용은 monadic type을 얻을 수 있는 생성자(?)를 의미하는 것 같습니다..

스칼라로 표현하면 아래와 같습니다.

```scala
trait M[T]{
...
}
```
숫자 1은 monadic type이 아니지만 , List(1)은 monadic type이니까..이 이정도로

생각하면 될 것 같습니다.

2번 내용은 value를 입력받아서 mondict type을 만드는 unit 함수를 이야기 하고 있습니다.

스칼라로 표현하면 아래와 같습니다.

```scala
def unit[T](x:T) : M[T]
```

3번 내용은 bind 함수에 대한 이야기 인데요...

첫번째 argument가 monadic typed이고 , 두번째 argument가 첫번째 argument의

type을 다른 monadic type으로 변환하는 함수이면, 결과는 다른 monadic type 이라는 의미인데요..

이건 아래 코드를 보는 것이 훨씬 좋을 것 같습니다.

```scala
trait M[T]{
  def bind[U](f:T=>M[U]):M[U]
}
```

그런데, 이건 어디선가 본 정의 아닌가요??^^

스칼라의 flatmap은 Monad에서 정의한 bind와 동일하게 동작 합니다.

List와 비교해서 다시 한번 정리 보면 아래와 같습니다.

```scala
List[+A]{ ...}  // 1번 정의
List(1) // 2번 정의(unit)
List(1).flatMap(x => List((x,x+1))) // 3번 정의(bind)
```

## Moand Laws
Moand 아래 세가지 규칙이 있습니다.

```scala
// Associativity
m flatMap f flatMap g == m flatMap (x => f(x) flatMap g)

//Left Unit
unit(x) flatMap f == f(x)

//Right Unit
m flatMap unit = m  
```

스칼라의 Option class가 Monad Laws를 지키고 있는지 확인해보도록 하겠습니다.

(Option의 자세한 내용은 다음에 살펴보도록 하겠습니다.)

Optcion class flatMap의 구현은 아래와 같습니다.

```scala
abstract class Option[+T]{
  def flatMap[U](f: T => Option[U]) : Option[U] = this match{
    case Some(x) => f(x) // extends Option[T]
    case None => None // None extends Option[Nothing]
  }
}
```

Left Unit 조건부터 확인 해 보도록 하겠습니다.
```scala
unit(x) flatMap f == f(x)
```
위의 표현식을 Option에 적용하면 아래와 같습니다.(Some은 Option을 상속받고 있습니다.)
```scala
Some(x) flatMap f == f(x)
```
flatMap을 적용해보면...

```scala
Some(x) match{
  case Some(x) => f(x)
  case None => None
} == f(x)
```

실제로 테스트 해보면..

```scala
val num = 10
val a = Some(num)
val a2 = a.flatMap(t) // Some(20)
val a3 = t(num) // Some(20)

def t(b:Int):Option[Int] = {
  if (b > 5) Some(b*2)
  else None
}
```

Left Unit 조건과 비교해보면 아래와 같습니다.

```scala
Some(num).flatMap(t) = t(num) // Some(x) flatMap f = f(x) 
```

Option 클래스는 Monad LeftUint 법칙을 준수합니다.

이번엔 Right Unit 조건을 확인해 보겠습니다.

```scala
m flatMap unit == m
```

위의 표현식을 Option에 적용하면 아래와 같습니다.

(unit 함수는 Some과 대응 됩니다. Formal definition 2번째 정의!!)
```scala
opt flatMap Some == opt
```

flatMap을 적용해보면..
```scala
opt match{
  case Some(x) => Some(x)
  case None => None
}
```

실제로 테스트 해보면...
```scala
val tt = Some(num).flatMap(x => Some(x)) // Some(10)
val tt2 = Some(num) // Some(10)
```

Right Unit 조건과 비교해보면 아래와 같습니다.

```scala
Some(num).flatMap(x => Some(x)) == Some(num) //opt flatMap Some == opt 
```

Option 클래스는 Monad RightUint 법칙을 준수합니다.

마지막으로, Associativity 조건을 확인해 보겠습니다.

(마지막이니까..조금 복잡하겠죠?^^)

```scala
m flatMap f flatMap g == m flatMap (x => f(x) flatMap g)
```

위의 표현식을 Option에 적용해보면..
```scala
opt flatMap f flatMap g
```

flatMap을 적용해보면...

```scala
opt match {case Some(x) => f(x) case None => None}
    match {case Some(y) => g(y) case None => None}
```

opt flatMap f가 f(x)나 None 둘중에 하나를 return 해도,

"match {case Some(y) => g(y) case None => None}"을 적용해야 하기 때문에...

아래와 같이 첫번째 match 안쪽으로 코드를 옮겨도 동일하게 동작합니다.

```scala
opt match {
            case Some(x) => f(x) match 
                  {
                    case Some(y) => g(y) 
                    case None => None
                  } 
            case None => None match
                  {
                    case Some(y) => g(y) 
                    case None => None
                  }                   
          }
```

None match는 무조건 None이니까...

아래와 같이 정리합니다.
```scala
opt match {
            case Some(x) => f(x) match 
                  {
                    case Some(y) => g(y) 
                    case None => None
                  } 
            case None => None         
          }
```

"match { case Some(y) => g(y) case None => None }"는 flatMap g이니까..

아래와 같이 정리합니다.

```scala
opt match {
            case Some(x) => f(x) flatMap g
            case None => None         
          }
```

이것은 마지막으로 아래와 같이 정리 할 수 있습니다.

``` scala
opt flatMap (x => f(x) flatMap g)
```

Option 클래스는 Associativity 법칙까지 준수합니다.(Moand!!)

마지막으로 For Expression과 Monad에 대해서 알아보도록 하겠습니다.

## For Expression With Monad

아래의 For Expression을 확인 해보도록 하겠습니다.

```scala
val a = Some(10)

def f(b:Int):Option[Int] = {
  if (b > 5) Some(b*2)
  else None
}

def g(b:Int):Option[Int] = {
  if (b > 5) Some(b*2)
  else None
}

for(
     y <- for( x <- a ; y <- f(x) ) yield y
     ; z <- g(y)
   ) yield z
```

복잡하네요... 위의 코드는 결국 아래와 같은 2개의 작업을 수행 합니다.

- "y <- for( x <- a ; y <- f(x) ) yield y"
- "z <- g(y)"

a의 flatMap을 사용하여 x값을 얻고 , x 값을 f함수에 대입하여 y값을 얻고,

y 값을 g 함수에 대입하여 z 값을 얻습니다. 즉 , 아래와 같은 절차로 수행됩니다..

(잘 이해가 안되셔도...좌절하지마세요..저도 지금 보니까 이해가 잘 안되네요..ㅠㅠ)

```scala
m flatMap (x => f(x) flatMap g)
```

Monad에 Associativity 규칙 기억나시나요..아래와 같습니다..

```scala
m flatMap f flatMap g == m flatMap (x => f(x) flatMap g)
```
이 규칙을 적용하면..

```scala
for(x <- a;
    y <- f(x);
    z <- g(y)
) yield z
```

이런 코드를 만들어 낼 수 있습니다!!

(사실, 이렇게 코드를 먼저 작성 할 줄 안다면...복잡한 for expression을

사용할 일도 없겠죠?^^)

지금까지 Monad에 대해서 알아보았습니다. 개념적인 부분들..증명하는 부분들이 많아서

조금 어려운 것 같습니다. 이해가 안되는 부분은 스칼라에 조금 더 익숙해진 다음...

봐도 될 것 같습니다.