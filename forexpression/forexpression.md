# For Expression
이글은 스칼라의 For Expression 대하여 설명한 글이며, coursera의 스칼라 프로그래밍 강의를 참조하여 작성하였습니다.

## Map Or FlatMap Or Filter

스칼라 For Expression에 대하여 이야기 하기전에, 스칼라 Collection들이 가지고 있는 몇가지 Api에 대하여 알아보도록 하겠습니다.

### Map
숫자형 배열의 각 항목에 2배수 배열은, 아래와 같이 map 함수를 이용하여 간단히 구현 할 수 있습니다.

```scala
val l  = List(1 , 2 , 3)
val l2 = l.map((x) => x*2) // 2 , 4 , 6
val l3 = l.map(_ * 2) // 2 , 4 , 6
```

List의 맵과 동일하게 동작하는 함수를 만들어 보면 아래와 같을 것입니다.

(직접 구현해보시는것도..^^)

```scala
def likeMap[T,U](l:List[T])(f:T => U) : List[U] = l match{
  case head :: tail => f(head) :: likeMap(tail)(f)
  case Nil => Nil
}

val l4 = likeMap(l)(_*2) // 2 , 4 , 6
```

### FlatMap
숫자형 배열을 원소로 가지고 있는 배열의 2배수는 어떻게 만들 수 있을까요??

위의 Map 함수를 이용하여 , 2배수 배열을 생성한 뒤 두개의 배열을 합치는 방법도 있지만...

스칼라의 FlatMap 함수를 사용하여 조금 더 쉽게(?) 결과를 만들어 낼 수 있습니다.

```scala
val fl = List(List(1 , 2 , 3) , List(4 ,5 , 6))
val fl2 = fl.flatMap( x => x.map(_ * 2)) // 2 , 4, 6, 8, 10, 12
```

Map과 마찬가지로 FlatMap과 동일하게 동작하는 함수를 직접 만들어 보도록 하겠습니다.

```scala
def likeFlatMap[T , U](l:List[T])(f:T => List[U]) : List[U] =  l match{
  case head :: tail => f(head) ++ likeFlatMap(tail)(f)
  case Nil => Nil
}

val f3 = likeFlatMap(fl)(x => x.map(_*2)) // 2 , 4, 6, 8, 10, 12
```

### Filter
숫자형 배열 중에서 2의 배수 항목만 가지는 리스트는 어떻게 만들 수 있을까요??

스칼라의 Filter 함수를 이용해서 간단하게 구현 할 수 있습니다.

```scala
val fil = List(1 ,2 ,3 ,4 , 5 , 6 , 7 , 8 , 9 , 10)
val fil2 = fil.filter(_ % 2 == 0) // 2, 4, 6, 8, 10
```

Filter와 동일하게 동작하는 함수도 만들어 보도록 하겠습니다.

```scala
def likeFilter[T](l:List[T])(f:T=>Boolean) : List[T] = l match{
  case head :: tail =>{
    if(f(head)){
      head :: likeFilter(tail)(f)
    }else{
      likeFilter(tail)(f)
    }
  }
  case Nil => Nil
}

val fil3 = likeFilter(fil)(_%2 == 0) // 2, 4, 6, 8, 10
```

## Map And FlatMap And Filter
이번에는 위에서 살펴본 세가지 함수를 이용해서 조금 더 복잡한 프로그램을 작성해보도록 하겠습니다.

두개의 숫자형 배열중 각 항목의 합이 짝수인 쌍(Tuple)을 출력하는 프로그램은 어떻게 구현 할 수 있을까요?

예를들어,

(1 ,2) 배열과 (3, 4)배열이 있는 경우 출력은..

(1,3) , (2,4)가 됩니다.^^

For문을 이용해도 되지만.. 우리가 스칼라 프로그래머라는 것을 잊지말고요^^

(Hint : FlatMap , Filter , Map 순으로 사용해보세요..)

```scala
val t1 = List(1, 2)
val t2 = List(3, 4)

val t3 = t1.flatMap(
  x => t2.filter(y => ( ( x+y ) % 2 == 0)).map(z => (x , z))
)
//List((1,3), (2,4))
```

다양한 방법들이 있겠지만.. 세가지 함수를 사용해서 위와 같이 구현 할 수 있습니다.

프로그램이 동작하는 방식을 살펴보면..

- 'x'라는 변수에 1 입력
- t2 리스트에 Filter 함수 호출 , 1과의 합이 짝수인 리스트 출력
- 출력된 리스트의 값과 1의 튜플 생성
- t1의 항목만큼 반복

스칼라에서는 FlatMap , Map , Filter가 복잡하게 사용되는 다소 복잡한 코드를

조금더 간결하게 만들 수 있는 방법을 제공 합니다.

## For Epxressions

```scala
for{
  x <- t1
  y <- t2
  if ( x+y ) % 2 == 0
}yield (x,y)
////List((1,3), (2,4))
```

이전의 코드를 이렇게 간단히(?) 표현 할 수 있습니다. 하나씩 천천히 살펴보도록 하겠습니다.

#### 1.yield(map)

```scala
for(x <- l)yield (x*2) 
```
스칼라는 위의 For Expression을 아래와 같이 변경합니다.

```scala
l.map(x=>x*2)
```

#### 2.if(withFilter)
```scala
for(x <-l if x>1)yield x
```
스칼라는 위의 For Expression을 아래와 같이 변경합니다.

(withFilter에 대해서는 나중에 자세히 알아보도록 하겠습니다. 지금은 filter와 동일하다고 생각하시면..^^)
```scala
for(x <-l.withFilter(_>1)) yield x
```

#### 3.flatmap
```scala
for(x<-l ;y<-l2 )yield (x+y)
```

스칼라는 위의 For Expression을 아래와 같이 변경합니다.
```scala
l.flatMap(x => for(y <-l2)yield (x+y))
```

어떻게 For Expression으로 이전의 표현식을 변경 할 수 있었는지..

조금 이해가 되시나요?^^

(이해가 되시길 바랍니다...사실 저도 볼때마다 헷갈리네요..ㅠㅠ)

이번에는 For Expression으로 조금 더 재미있는 코드를 만들어 보도록 하겠습니다.

## More
지금까지는 List의 For Expression에 대해서 알아 보았습니다.

혹시 For Expression을 다른 클래스에서도 사용 할 수 있을까요??

정답은 "YES"

한번 천천히 알아보도록 하겠습니다.

먼저 , Java의 Random 클래스를 이용해서 Random 숫자를 출력하는 예제를 작성해보겠습니다.

```scala
import java.util.Random
val r = new Random
val x = r.nextInt
```

이 Random 클래스를 이용해서 Boolean형 Random , Pari형 Random을 만들어 보도록 하겠습니다.

generate라는 함수를 가지고 있는 RandomGenerator trait를 생성합니다.

```scala
trait RandomGenerator[+T]{
  def generate:T
}
```

랜덤한 숫자를 리턴하는 RandomGenerator를 아래와 같이 작성합니다.
```scala
val intGen = new RandomGenerator[Int] {
  override def generate: Int = new Random().nextInt
}
val ir = intGen.generate // plus or minus number
```

이번엔 참/거짓 둘중 하나만 리턴하는 RandomGenerator를 아래와 같이 작성합니다.
```scala
val booleanGen = new RandomGenerator[Boolean] {
  override def generate: Boolean = intGen.generate > 0
}

val br = booleanGen.generate // true or false
```

마지막으로 랜던 숫자쌍을 리턴하는 RandomGenerator를 아래과 같이 작성합니다.
```scala
val pairGen = new RandomGenerator[(Int , Int)] {
  override def generate: (Int, Int) = (intGen.generate , intGen.generate)
}

val pr = pairGen.generate // random tuple
```

음..또 다른 RandomGenerator를 생성하려면...계속해서 중복되는 코드가 나타나게 됩니다.

```scala
val boolGen = for(x <- integer) yield x > 0

val pairGen = for(x<-t , y<-u) yield(x.y)
```

만약 , 위와 같은 For Expression으로 표현 할 수 있다면 조금 더 간결한 코드를 작성 할 수 있을 것 같습니다.

먼저 위의 코드를 map과 flatMap으로 표현해 보도록 하겠습니다.

```scala
val boolGen = integer map (x => x > 0)

t flatMap ( x => u map ( y => (x , y))  ) 

```

이제 RandomGenerator trait에 map과 flatMap 함수를 추가 해보도록 하겠습니다.

```scala
trait RandomGenerator[+T]{
  self =>

  def generate:T

  def map[S](f:T => S) : RandomGenerator[S]= new RandomGenerator[S]{
    def generate:S = f(self.generate)
  }

  def flatMap[S](f:T => RandomGenerator[S]) : RandomGenerator[S] = new RandomGenerator[S] {
    def generate: S = f(self.generate).generate
  }

}
```

'self =>'라는 새로운 문법이 나타났습니다.!!

'self =>'의 의미는 this를 self라는 키워드로 사용하겠다라는 정의로 생각하면 됩니다.

map 함수의 정의를 보면, self.generate 함수를 호출하고 나온 값을 f 함수에 넘기게 됩니다.

이때, self라는 키워드가 없다면..어떻게 될까요..?? 'this' 키워드를 사용하면 어떻게 될까요..??

(한번 고민해보세요~^^;)

위의 코드들이 이해가 가신다면... 여기까지만 읽으셔도 OK!!

but.. 조금더 설명이 필요하시다면..조금 더 진행해보도록 하겠습니다.^^

조금 만만해 보이는 map 표현식 부터 이야기 해보겠습니다.

```scala
val boolGen = for(x <- integer) yield x > 0
```
위의 For Expression은 아래와 같이 표현 할 수 있습니다.

```scala
val boolGen2 = integer map (x => x > 0)
```

이것을 함수로 표현하면...
(RandmoGenerator를 파라미터로 받는 함수로 표현합니다.)
```scala
def map[T,U](t:RandomGenerator[T])(f:T => U):RandomGenerator[U] = t map(x => f(x))
```

마지막으로 RandomGemerator Map으로 치환해 보면...
(RandomGemerator map 함수를 확인해 주세요^^)
```scala
def map[T,U](t:RandomGenerator[T])(f:T => U):RandomGenerator[U]
    = new RandomGenerator[U] {
      def generate: U = f(t.generate)
}

val newBool = map(intGen)(x => x > 0).generate // true or false
```
이렇게 표현 할 수 있습니다.

다음은 flatmap 표현식을 이야기 해보겠습니다.

(조금 복잡합니다..ㅠㅠ)

```scala
for(x<-t , y<-u) yield(x.y)
```

flatmap For Expression은 위와 같습니다.

```scala
t flatMap ( x => u map ( y => (x , y))  ) 
```

이 표현식을 풀어보면 위와 같습니다.

이것을 함수로 표현해보면...
(RandomGenerator를 두개 받는 함수로..)

```scala
def pairs[T , U](t:RandomGenerator[T] , u:RandomGenerator[U]) = t flatMap(
     x => u map(y => (x , y))
  )
```

map을 치환하면...
```scala
def pairs[T , U](t:RandomGenerator[T] , u:RandomGenerator[U]) = t flatMap(
  x => new RandomGenerator[(T,U)] {
     def generate: (T, U) = (x , u.generate)
  }
)
```

마지막으로 flatmap을 치환하면...
```scala
def pairs[T , U](t:RandomGenerator[T] , u:RandomGenerator[U]) =
    new RandomGenerator[(T,U)] {
       def generate: (T, U) = (new RandomGenerator[(T,U)] {
         def generate: (T, U) = (t.generate , u.generate)
      }).generate
    }

val newp = pairs(intGen , intGen).generate // random int tuple    
```

음...음... 쉽게 설명을 하려고 해도...쉽지가 않네요..ㅠㅠ

지금까지 스칼라의 For Expression에 대해서 알아 보았습니다.