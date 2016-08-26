# Future
이글은 스칼라의 Future 대하여 설명한 글이며, coursera 강의를 참조하여 작성 하였습니다.

## Send Mail To Europe

DB에서 데이타를 읽어, 유럽으로 메일을 보내는 프로그램을 작성 해보도록 하겠습니다.

(진짜로 보내지는건 아니고요..^^)

```scala
class DB{
  def loadMail() : String = {
    "Hi, There"
  }
}

class Mail{
  def SendToEurope(mail:String) = {
    println("send mail : " + mail)
  }
}


val db = new DB()
val mail = new Mail()

val msg = db.loadMail();
mail.SendToEurope(msg);
```

너무 순식간에 메일이 보내지네요..^^

조금더 현실감있게...바꿔보도록 하겠습니다.


```scala
class DB{
  def loadMail() : String = {
    // DB SELECT TIME
    Thread.sleep(2 * 1000)
    "Hi, There"
  }
}

class Mail{
  def SendToEurope(mail:String) = {
    // SEND MAIL TIME
    Thread.sleep(3 * 1000)
    println("send mail : " + mail)
  }
}
val db = new DB()
val mail = new Mail()
val msg = db.loadMail();
mail.SendToEurope(msg);
```

Sleep으로 시간을 조금 줘봤습니다.^^

DB 조회 2초, 메일 전송 3초.. 총 5초에 시간이 소요 되었습니다.

한가지 상황을 더 추가해 보도록 하겠습니다.

사실 저 메일은 그렇게 중요한 메일이 아니였습니다.!!

보내도 되고 안보내도 되는..그런..스팸같은..메일이라고 가정해보겠습니다.^^

진짜 중요한 작업은 메일 전송 이후에 시작됩니다.

```scala
mail.SendToEurope(msg);

// THIS IS REAL IMPORTANT JOB
println("i'm important job!! , hurry up!!")
```
스팸 메일을 보내느라 5초를 보냈더니..화가 난것 같습니다.

스팸 메일은 비동기로 보내도록 하겠습니다.

## Send Mail To Europe With Future
스칼라에서는 비동기 작업을 위해 Future를 제공 하고 있습니다.

```scala

import scala.concurrent.{Future}
import scala.concurrent.ExecutionContext.Implicits.global

    class DB{
      def loadMail() : String = {
        // DB SELECT TIME
        Thread.sleep(2 * 1000)
        "Hi, There"
      }
    }
    class Mail{
      def SendToEurope(mail:String) = {
        // SEND MAIL TIME
        Thread.sleep(3 * 1000)
        println("send mail : " + mail)
      }
    }

    val db = new DB()
    val mail = new Mail()
    val f = Future[Unit]{
      mail.SendToEurope(db.loadMail())
    }

    f.onComplete {
      case _ => println("FINISH")
    }

    println("i'm important job!! , hurry up!!")

    Thread.sleep(10 * 1000)
    // 프로그램 종료 방지 입니다.
```

DB를 조회하고 메일을 전송하는 부분을 스칼라의 Future를 사용하여

비동기로 동작하도록 하였습니다.^^

Future로 작성된 코드에 대해서 조금 더 자세히 살펴보도록 하겠습니다.

```scala
val f = Future[Unit]{
  mail.SendToEurope(db.loadMail())
}

// Future object apply method
def apply[T](body : => T)(implicit executor : scala.concurrent.ExecutionContext) : scala.concurrent.Future[T]
```

먼저, Future Object의 apply 함수를 살펴보면 , T를 리턴하는 body와

implict 파라미터로 ExecutionContext를 받고 있습니다.

그리고 코드의 import 문을 보면... 아래와 같이 선언한 것을 확인 할 수 있습니다.

```scala
import scala.concurrent.ExecutionContext.Implicits.global
```

스칼라에서 제공하는 Thread Pool로 생각 하면 될 것 같습니다.

지금까지 스칼라 공부해왔던 내용을 총 동원해서 생각해보면..

"body : => T"는 T를 리턴하지만 실제 참조가 이루어지기 전까지는 호출 되지 않는 네임 파라미터 이고..

implict 파라미터는 직접 파라미터로 값으 넘겨주지 않아도, 스칼라가 알아서(?) 판단하여

넘겨주라고 선언 하는 것입니다.

(import를 따라가보면 implict val이 선언된 것을 확인 할 수 있습니다.^^)

그리고 마지막으로... onComplete 함수를 확인 해보도록 하겠습니다.

```scala

f.onComplete {
  case _ => println("FINISH")
}

// Future trait onComplete method
def onComplete[U](f : scala.Function1[scala.util.Try[T], U])(implicit executor : scala.concurrent.ExecutionContext) : scala.Unit
```
onComplete는 파라미터로 Try를 전달받고 U type을 리턴하는 함수와

apply 함수와 동일하게 ExecutionContext를 implict 파라미터로 전달 받고 있습니다.

Future로 생성한 작업이 완료되면 onComplete에 전달한 함수를 호출 해주는 형식 입니다.^^

(javascript callback과 유사한거 같습니다.)

Try에 대해서는 나중에 다시한번 자세히 살펴 보도록 하겠습니다.

이번에는..이전과는 다르게 여러개의 Future를 연결 시켜보도록 하겠습니다.

## Welcome to CallBack Hell

이번엔, 아침에 일어나서 커피와 빵을 먹는 코드를 스칼라로 작성해 보도록 하겠습니다.

```scala
    println("WAKE UP")

    def bread(bread:String):Future[Unit] = Future[Unit] {
      println("START SPREAD BUTTER ON" + bread)
      Thread.sleep(1 * 1000)
      println("END SPREAD BUTTER ON BREAD"  + bread)
    }

    val toast = Future[Future[Unit]] {
      println("START BAKE BREAD")
      Thread.sleep(5 * 1000)
      println("DING DONG , HOT BREAD READY")
      bread("BAKED BAGEL")
    }

    def coffee:Future[Unit] = Future[Unit] {
      println("START MAKE CUP OF COFFEE")
      Thread.sleep(1 * 1000)
      println("END MAKE CUP OF COFFEE")
    }

    val water = Future[Future[Unit]] {
      println("START BOIL WATER")
      Thread.sleep(1 * 1000)
      println("DING DONG , HOT WATER READY")
      coffee
    }

    toast.onComplete({
      case Success(b) => b.onComplete({
        case Success(c) => water.onComplete({
          case Success(d) => d.onComplete({
            case Success(e) => println("EAT BREAD AND COFFEE")
          })
        })
      })
    })
```

뭔가 보기만 해도 복잡해 보이네요..ㅠㅠ(제가 초보라서 그럴지도..)

먼저 , def인 함수이고 val은 변수죠..?^^

val toast는 선언과 동시에 바로 쓰레드를 할당 받아 작업(?)을 시작 합니다.

반면에 def bread는 누군가 자신을 호출 할때까지 기다리게 됩니다.

toast future의 return 값으로 bread 함수 호출의 결과 값인 future를 return 합니다.

bread 함수가 호출 될때, 생성된 Future는 쓰레드를 할당 받아 작업을 시작 합니다.

water와 coffee도 동일하게 동작 합니다.

마지막으로..onComplete를 살펴 볼텐데요..js의 callback hell이 떠오르네요

먼저 toast.onComplete은 성공 시, bread 함수의 호출 결과인 future를 Success 값으로 return 합니다.

(실패의 경우는 너무 복잡해 질 것 같아서..이번 글에서는 제외하였습니다.)

그리고 성공시 전달되는 bread future의 onComplete이 호출되면 , water.onComplete을 호출 합니다.

이부분이 조금 이해가 안될 수도 있는데요...

시간상으로 계산해보면..boil + bread는 6초가 걸리고 water는 1초만에 종료가 되는데

water.onComplete를 6초후에 호출하는 것은 조금 이상해 보입니다.

scala future는 future의 작업이 먼저 종료 되었다 하더라도,

onComplete의 콜백 함수가 등록 되는 시점에 결과값을 전달 해 줍니다.^^

(javascript도 동일하게 동작했던것 같습니다.)

빵을 굽는 일이 커피를 만드는 일보다 먼저 종료되면..커피가 다 만들어 질때까지 기다리고..

빵을 굽는 일이 커피를 만드는 일보다 늦게 종료되면..빵이 다 만들어 질때까지 기다리는

방식 입니다.^^

아무리 생각해도 이코드는 너무 복잡한 것 같습니다. 조금 바꿔보도록 하겠습니다.^^

## Future's map and flatMap

빵을 굽는 코드를 변경 해 보도록 하겠습니다.

```scala
def bread(bread:String):Future[Unit] = Future[Unit] {
  println("START SPREAD BUTTER ON" + bread)
  Thread.sleep(1 * 1000)
  println("END SPREAD BUTTER ON BREAD"  + bread)
}

val toast = Future[String] {
  println("START BAKE BREAD")
  Thread.sleep(5 * 1000)
  println("DING DONG , HOT BREAD READY")
  "BAKED BAGEL"
}

val t1 : Future[Future[Unit]] = toast.map(b => bread(b))

t1.onComplete({
  case Success(a) => a.onComplete({
    case Success(b) => println("EAT")
  })
})
```

toast Future는 Future를 return하는 대신 String을 리턴하도록 변경되었고,

toast에 map 함수를 사용해서 String을 받아 Future를 리턴하도록 하였습니다.

이번엔 flatMap을 적용해 보도록 하겠습니다. 

```scala
val t1 = toast.flatMap(bread((_)))
  t1.onComplete({
    case Success(a) => println("EAT")
})
```
onComplte 소스가 훨씬 간단해졌죠?^^

이제 아무리 많은 future의 조합도 훨씬 보기 좋은 코드로 작성 할 수 있게 되었습니다.

## Future For Expression

이제, 마지막으로 for expression을 적용해 보도록 하겠습니다.

Future도 Monad 법칙이 성립하기 때문에 for expression을 사용 할 수 있습니다.

빵을 만드는 작업에 for expression을 적용해 보도록 하겠습니다.

```scala
for{
  a <- toast
  b <- bread(a)
}(println("EAT BREAD"))
```

flatMap을 사용할때 보다 조금 간단해졌죠..?^^

처음엔 헷갈리지만 for expression은 사용하다 보면 꽤 좋은 도구 인 것 같습니다.

이번엔 빵을 굽고 커피를 만드는 작업까지 for expression을 적용해 보도록 하겠습니다.

```scala
val t1 = toast.flatMap((bread(_)))
val t2 = water.flatMap(Unit => coffee)

for{
  a <- t1
  b <- t2
}(println("EAT"))
```

처음 코드보다는 훨씬 간결해진 모습입니다.

지금까지 스칼라의 Future에 대하여 알아보았습니다.