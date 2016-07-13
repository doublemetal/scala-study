# Implicit
이글은 스칼라의 Implicit 대하여 설명한 글이며, coursera의 스칼라 프로그래밍 강의를 참조하여 작성하였습니다.

## Normal MergeSort

스칼라를 이용해서 숫자 배열을 정렬하는 MergeSort를 구현해 보도록 하겠습니다.

먼저 완성된 코드는 아래와 같습니다.

```scala
def mSort(l : List[Int]): List[Int] ={
    val n = l.length / 2
    if(n == 0) l
    else{
      def merge(ll : List[Int] , rl : List[Int]) : List[Int] = (ll , rl)  match{
        case (Nil , rl) => rl
        case(ll , Nil) => ll
        case(l :: ll1 , r :: rl1) =>
          if(l < r) l :: merge(ll1 , rl)
          else r :: merge(ll , rl1)
      }
      val(fst , snd) = l splitAt n
      merge(mSort(fst) , mSort(snd))
    }
````
 java로 구현된 merge sort와 조금 다른 부분은 merge 하는 부분인데요...

 조금 자세히 알아보도록 하겠습니다.

 ```scala
      def merge(ll : List[Int] , rl : List[Int]) : List[Int] = (ll , rl)  match{
        case (Nil , rl) => rl
        case(ll , Nil) => ll
        case(l :: ll1 , r :: rl1) =>
          if(l < r) l :: merge(ll1 , rl)
          else r :: merge(ll , rl1)
      } 
 ```

merge 함수에 전달된 ll과 rl은 이미 정렬되어 있는 상태 입니다.

전달 된 두 배열 중 첫번째 값을 비교하여 , 작은 숫자를 배열에 앞쪽으로 위치하게 하고 

다시 merge 함수를 호출할 때는 앞쪽으로 위치 시킨 숫자를 제외한 배열을 다시 넘기게 됩니다.

숫자만 정렬하는 merge sort를 문자도 정렬 할 수 있는 merge sort로 변경하려면 어떻게 해야 할까요..?? 

.

.

.

다양한 방법이 있겠지만, 함수형 언어를 배우고 있는 만큼 함수를 파라미터로 전달 받는 merge sort를

구현해 보도록 하겠습니다.


## Various Type MergeSort

```scala
  def mSort2[T](l : List[T])(lt : (T,T) => Boolean): List[T] ={
    val n = l.length / 2
    if(n == 0) l
    else{
      def merge(ll : List[T] , rl : List[T]) : List[T] = (ll , rl)  match{
        case (Nil , rl) => rl
        case(ll , Nil) => ll
        case(l :: ll1 , r :: rl1) =>
          if(lt(l , r)) l :: merge(ll1 , rl)
          else r :: merge(ll , rl1)
      }
      val(fst , snd) = l splitAt n
      merge(mSort2(fst)(lt) , mSort2(snd)(lt))
    }
  }
```

이전 코드와 거의 유사하지만 , 두 원소간의 대/소를 비교하는 함수를 전달 받고 다양한 타입을

파라미터로 전달 받을 수 있도록 수정 했습니다.

이제, 아래와 같이 String 배열도 정렬 할 수 있게 되었습니다.

```scala
val strs = List("1" , "5" , "3" , "2")
mSort2(strs)((x:String,y:String) => x.compareTo(y) < 0)
```

두개의 String을 입력받아 두 문자열간의 대/소 관계를 리턴하는 함수를 mSort2의 파라미터로 전달 하였습니다.

스칼라는 똑똑하기(?) 때문에 아래와 같이 currying 형태로 전달되는 함수의 파라미터를 생략해도 정상적으로 동작 합니다. 

```scala
mSort2(strs)((x,y) => x.compareTo(y) < 0)
```

## MergeSort With Ordering

스칼라에서는 Ordering이라는 type을 제공합니다.(type에 대해서는 다음 기회에 자세히 알아보도록

하겠습니다.) Ordering을 사용하기 위해 merge sort 코드를 아래와 같이 수정합니다.

```scala
  def mSort3[T](l : List[T])(ord:Ordering[T]): List[T] ={
    val n = l.length / 2
    if(n == 0) l
    else{
      def merge(ll : List[T] , rl : List[T]) : List[T] = (ll , rl)  match{
        case (Nil , rl) => rl
        case(ll , Nil) => ll
        case(l :: ll1 , r :: rl1) =>
          if(ord.lt(l , r)) l :: merge(ll1 , rl)
          else r :: merge(ll , rl1)
      }
      val(fst , snd) = l splitAt n
      merge(mSort3(fst)(ord) , mSort3(snd)(ord))
    }
  }

mSort3(List("1" , "5" , "3" , "2"))(Ordering.String)  
```

String의 대/소를 비교하는 코드를 작성하지 않아도 위와 같이 정렬을 할 수 있게 되었습니다.

## Implicit Parameter

merge sort 함수의 파라미터에 implicit 키워드를 아래와 같이 추가해보도록 하겠습니다.

```scala
  def mSort4[T](l : List[T])(implicit ord:Ordering[T]): List[T] ={
    val n = l.length / 2
    if(n == 0) l
    else{
      def merge(ll : List[T] , rl : List[T]) : List[T] = (ll , rl)  match{
        case (Nil , rl) => rl
        case(ll , Nil) => ll
        case(l :: ll1 , r :: rl1) =>
          if(ord.lt(l , r)) l :: merge(ll1 , rl)
          else r :: merge(ll , rl1)
      }
      val(fst , snd) = l splitAt n
      merge(mSort4(fst) , mSort4(snd))
    }
  }


  mSort4(List("1" , "5" , "3" , "2"))
  mSort4(List(-1 , 10 , 2 , 4))
```

mSort4 호출시에는 Ordering type을 전달하지 않고 List만 파라미터로 사용 하였습니다.

mSort4 호출시 전달된 ord 객체는 아래 그림과 같습니다.

image::debug1.png[]

image::debug2.png[]

숫자를 입력받아 화면에 출력하는 간단한 예제를 만들어 보도록 하겠습니다.

```scala

def normalPrint(i:Int) : Unit = {
    print(i)
  }

def implicitPrint(implicit i:Int) : Unit = {
    print(i)
}

implicitPrint(30) // 30

implicitPrint //  error
```

implicitPrint 함수에 30을 파라미터로 넘겨주면 정상적으로 30이 출력 됩니다.

하지만 아무값도 넘겨주지 않으면 , 아래와 같은 오류가 발생합니다.

  error: could not find implicit value for parameter i: Int

이번엔 아래와같이 implicit 변수를 선언하고 implicitPrint 함수를 호출 해보겠습니다. 

```scala
implicit val a = 10
implicitPrint // 10
```    

정상적으로 10이 화면에 출력 되는 것을 확인 할 수 있습니다. 
