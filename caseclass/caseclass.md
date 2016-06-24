# CASE CLASS
이글은 스칼라의 Case Class에 대하여 설명한 글이며, coursera의 스칼라 프로그래밍 강의를 참조하여 작성하였습니다.

## Im' a Good Programmer
숫자와 "+"를 입력받아 합을 계산하는 스칼라 프로그램을 작성해 보도록 하겠습니다. 계산식은 오직 '+'와 숫자만을 가질 수 있다고
가정합니다.(너무 복잡해져서요...ㅠㅠ)

아래코드는 eval 이라는 함수를 통해 계산을 수행하는 코드입니다.

Expr이라는 상위 trait를 만들고 이를 상속받는 Number , Sum Class를 생성합니다.

eval 함수 원형은 파라미터로 expr trait을 선언하고 있게 때문에, Number와 Sum을 파라미터로 전달 할 수 있습니다.

객체지향스럽나요?^^

```scala
trait Expr{
  def isNumber : Boolean
  def isSum : Boolean
  def numValue : Int
  def leftOp : Expr
  def rightOp : Expr
}

class Number(n:Int) extends Expr{
  def isNumber : Boolean = true
  def isSum : Boolean = false
  def numValue : Int = n
  def leftOp : Expr = throw new Error("Not Support")
  def rightOp : Expr = throw new Error("Not Support")
}

class Sum(l:Expr , r :Expr) extends Expr{
  def isNumber : Boolean = false
  def isSum : Boolean = true
  def numValue : Int = throw new Error("Not Support")
  def leftOp : Expr = l
  def rightOp : Expr = r
}

class ExprPrinter {
  def eval(e : Expr) : Int = {
    if(e.isNumber) e.numValue
    else if(e.isSum) eval(e.leftOp) + eval(e.rightOp)
    else throw new Exception("Not Support")
  }

  def Test : Int = {
    eval(new Sum(new Number(1) , new Number(2))) // 3
  }
}
```

비교적, 간단하게(?) 구현 할 수 있습니다. 그런데... 마이너스 연산을 추가 해달라는 요청을 받는다면..??

아래와 같은 작업을 수행하기만 하면 됩니다.!!

```
1) expr triat에 isMinus 함수 추가

2) expr을 상속받은 class에 isMiuns 함수 추가

3) eval에 Minus 처리 로직 추가
```

그런데.. 곱하가 연산을 추가 해달라는 요청을 받는다면... 그런데 제곱 연산을 추가 해달라는 요청을 받는다면..??

그런데...그만하겠습니다. 조금 더 나은 방법으로 구현하는 방법을 알아보도록 하겠습니다.

## Im'a Java Programmer

결론부터 말하자면..java의 "instance of"를 사용하면 하위클래스에 isNumber와 같은

함수를 추가 할 필요는 없어집니다. 바로 소스로 확인 해 보도록 하겠습니다.

```scala

def eval2(e:Expr) : Int = {
    if(e.isInstanceOf[Number]) e.numValue
    else if(e.isInstanceOf[Sum]) eval(e.leftOp) + eval(e.rightOp)
    else throw new Exception("Not Support")
  }

```

더이상 expr trait와 하위클래스에 이상한 함수를 추가 할 필요는 없어졌습니다.

이렇게 끝내기에는 뭔가 좀 찝찝하다면... 다음으로 넘어가 보도록 하겠습니다.

## Im'a Scala Programmer

스칼라에는 Pattern Matching이라는 조금 더 우아한 방법이 있습니다. 스칼라의 case 클래스와

pattern matching을 통해
아래와 같이 변경 할 수 있습니다.

```scala
trait NewExpr
case class NewNumber(n:Int) extends NewExpr
case class NewSum(l:NewExpr , r: NewExpr) extends NewExpr

def eval3(e:NewExpr) : Int = e match {
    case NewNumber(n) => n
    case NewSum(l,r) => eval3(l) + eval3(r)
  } 

eval3(NewSum(NewNumber(1) , NewNumber(2)))  
```

case clsss는 내부적으로 아래와 같이 Object를 선언하기 때문에 new Number(1) 대신 Number(1)로 선언 할 수 있습니다.

```scala
object NewNumber{
 def apply(n:Int) = new NewNumber(n)
}
```

이번에는 수식을 출력하는 함수를 작성해 보도록 하겠습니다.

```scala
def show(e:NewExpr) : String = e match{
    case NewNumber(n) => n.toString
    case NewSum(l,r) => show(l)  + "+"  + show(r)
  }

show(NewSum(NewNumber(1) , NewNumber(2))) // 1+2  
```

그럼... 1 + 2 \* 3 인 경우.. ( 1 + 2 ) \* 3으로 출력하는 프로그램은 어떻게 작성할까요..?^^

지금까지 스칼라의 case class와 pattern matching에 대하여 알아보았습니다.