---
layout: post
comments: true
title:  "Functional Programming in Scala week 4"
categories: programming
date:   2016-07-26 23:27:50 +0900
tags:
- scala
- coursera
---
## 4.1 Objects Everywhere

> 퓨어 object-oriented 언어란 모든 value가 object라는 말인데, 그렇다면 스칼라가 퓨어 object-oriented language인가?

스칼라의 모든 값은 object로 표현되기 때문에 퓨어하다 할 수 있다. 예로 scala.Boolean 대신 커스텀으로 Boolean 클래스를 정의한다(자바의 래핑클래스(Integer 등)처럼)

Boolean 클래스에서는 실제 스칼라 Boolean으로 사용할 수 있었던 연산을 모두 재정의해준다. 
ifThenElse는 if(cond) f1 else f2과 같다(여기서 f1, f2는 ifThenElse의 파라미터)
아래는 '<' 함수를 정의한 예제이다.

```
claass Boolean {
  ...
  def < (x: Boolean): Boolean = ifThenElse(false, x)
 }
```


## 4.2 Functions as Objects
스칼라에서는 function values는 오브젝트로 취급된다. 사실 function type A => B는 scala.Function1[A, B]의 축약 형태와 같다고 할수 있다.

```
package scala
trait Function1[A, B] {
  def aaply(x: A): B
}
```
즉, 함수는 apply 메소드를 가진 오브젝트와 같다.
익명함수의 경우에는 다음과 같이 확장될 수 있다.

```
(x: Int) => x * x

// is expanded to
{ class AnonFun extends Function1[Int, Int] {
  def apply(x: Int) = x * x
  }
  new AnonFun
}

// shorter
new FUnctino1[Int, Int] {
  def apply(x: Int) = x * x
}
```
그러니까 실제로 f(a, b) 라는 함수를 call 했을 때, f.apply(a, b)가 불리는 것과 같다는 말이다.
예를 들면,

```
val f = (x: Int) => x * x
f(7)

val f = new Function[Int. Int] {
  def apply(x: Int) = x * x
}
f.apply(7)
```
위에서 본것처럼 apply 메소드는 오브젝트 안에 있을 때 오브젝트 이름 그대로 호출할 수 있다. 지난번에 봤던 List를 예로 들어보면 아래와 같다.

```
trait List[T] {
  def isEmpty: Boolean
  def head: T
  def tail: List[T]
}

class Cons[T](val head: T, val tail: List[T]) extends List[T] {
  def isEmpty = false
}

class Nil[T] extends List[T] {
  def isEmpty: Boolean = true
  def head: Nothing = throw new NoSuchElementException("Nil.head")
  def tail: Nothing = throw new NoSuchElementException("Nil.tail")
}

// List()
object List {
  def apply[T]: List[T] = new Nil
  def apply[T](x: T): List[T] = new Cons(x, new Nil)
  def apply[T](x1: T, x2: T): List[T] = new Cons(x1, new Cons(x2, new Nil))
  
  // objectd 이름 그대로 호출 가능, 파라미터가 맞는 apply 메소드를 알아서 찾아감
  val a = List()
  val b = List(1)
  val c = List(2, 3)
} 
```

## 4.3 Subtyping and Generics

스칼라 언어에서 다형성을 표현하는 두가지 방법은 subtyping과 generic 이다.

### Type Bounds
> takes an IntSet
> returns the IntSet itself if all this elements are positive
> throws an exception otherwise

위의 세가지 조건을 충족시킬 수 있는 함수를 생각해보자.

```
def assertAllPos(s: IntSet): IntSet
```
대부분의 경우는 위의 함수로 충분하지만 정확히 하자면 다음과 같이 쓸수 있다.

```
def assertAllPos[S <: IntSet](r: S): S = ...
```
"S <: IntSet"을 type parameter S의 upper bound라고 한다. 이것은 S가 반드시 IntSet의 subType(또는 자신)이어야 한다는 말과 같다. 
반대로 "S :> T"는 S가 T의 superType이거나 T가 S의 subType이라는 말이다. 이를 lower Bounds 라고 한다.

```
[S >: NonEmpty]
```
위에서 말했듯이 위의 의미는 S가 NonEmpty 클래스의 supertype인데, S는 NonEmpty의 모든 base 클래스(자신 포함)가 해당된다. 여기서 S는 NonEmpty, IntSet, AnyRef, Any가 될 수 있다.

마지막은 Mixed Bound

```
[S >: NonEmpty <: IntSet]
```
이것의 의미는 S가 NonEmtpy와 IntSet 타입 사이의 모든 타입이 될 수 있다는 말과 같다.

### Covariance
서브클래스의 인스턴스 컬렉션을 상위클래스의 컬렉션으로 보내는 것을 Covariance(공변성)라고 한다. 왜냐하면 subtyping 관계가 컬렉션에서도 그대로 적용되었기 때문이다.

```
NonEmpty <: IntSet
// 위가 성립된다면 아래도 성립
List[NonEmpty] <: List[IntSet]
```

### Arrays in Scala

```
T[]     // Java
Array[T]  // Scala

// Covariance에 의해 아래가 성립
NonEmpty[] <: IntSet[]        // Java
Array[NonEmpty] <: Array[IntSet]  // Scala
```

자바의 Array Typing에는 타입과 관련된 아래의 문제가 있다.

```
NonEmpty[] a = new NonEmpty[]{new NonEmpty(1, Empty, Empty)}
IntSet[] b = a
b[0] = Empty
NonEmpty s = a[0]
```

a는 NonEmpty 타입의 Array를 가리키는 포인터이다. 두번째 줄에서 IntSet Array b 에 a를 대입하였다. b가 실제로 카리키는 대상은 NonEmpty List지만, covariance 규칙에 의해 상위 타입의 컬렉션이 하위 타입의 컬렉션을 대신할 수 있다. 세번째 줄에서 b의 첫번째 item에 Empty 클래스를 대입하였다. 마지막으로 a의 첫번째 item을 NonEmpty 타입의 s에 대입하였다. b와 a 는 실제로 가리키는 대상이 같기 때문에 세번째 줄에서 b[0]에 들어간 Empty는 a[0]에서도 동일하게 작동한다. 그런데 마지막 줄에서 Empty 타입의 item 을 NonEmpty 타입에 할당하기 때문에 런타임 에러가 발생한다.

### Liskov Substitution Principle
> If A <: B, then everything one can to do with a value of type B one should also be able to do with a value of type A
리스코프 치환원칙은 타입 A와 B가 있을때 하나의 타입이 다른 하나의 서브타입이 될 수 있는 조건에 대해 말해준다.

```
// in scala
val a: Array[NonEmpty] = Array(new NonEmpty(1, Empty, Empty))
val b: Array[IntSet] = a
b(0) = Empty
val s: NonEmpty = a(0)
```

스칼라의 경우에는 두번째 줄에서 컴파일 에러가 난다. 그 이유는 스칼라의 Array는 covariant하지 않기 때문이다. (NonEmpty )

```
NonEmpty <: IntSet
not Array[NonEmpty] <: Array[IntSet]
```

## 4.4 Variance

스칼라에서 List는 covariant, Array는 성립하지 않는다. 그 이유는 list의 경우에는 immutable한 컬렉션이고, Array는 mutable 하기 때문이다. 보통 mutation을 허용하는 타입은 covariant하지 않다.

C[T]에서 A <: B인 경우 다음이 성립한다.
B가 A의 수퍼타입이면서 C[B]가 C[A]의 수퍼타입인 경우에는 covariant, C[A]가 C[B]의 수퍼타입이면 contravariant

* C[A] <: C[B] 이면 C는 covariant (class C[+A])
* C[A] >: C[B] 이면 C는 contravariant (class C[-A])
* C[A]와 C[B] 둘다 다른것의 서브타입이 아니면 C는 nonvariant (class C[A])

다음의 두 타입중 어떤 타입이 수퍼타입이고, 어떤 타입이 서브타입인가?
함수의 파라미터가 더 구체적인(서브타입) 타입이 들어 갔을때는 반드시 그 타입으로 인자가 넘어와야한다. type B를 보면 파라미터 타입이 IntSet의 서브타입인 NonEmpty이므로 인자가 반드시 NonEmpty 타입이어야 한다. 반면에 type A를 보면, 파라미터 타입이 IntSet이라 NonEmpty 포함 IntSet의 모든 서브타입이 들어 올 수 있다. 리턴타입은 NonEmpty이므로 IntSet이라 할 수 있다. 즉, A는 B의 규칙을 만족시킨다. 게다가 A는 파라미터에 추가로 Empty 같은 타입이 들어 올 수 있으므로, A가 B보다 더 확장된 형태이다. 
그러므로, B가 A의 수퍼타입이다. 함수의 파라미터는 contravariant하고 함수의 리턴값은 covariant 하기 때문에 A <: B가 참이다.

```
type A = IntSet => NonEmpty
type B = NonEmpty => IntSet
```

위의 내용을 요약하면 아래와 같다. 

```
If A2 <: A1 and B1 <: B2, then
  A1 => B1  <:  A2 => B2
```

> Functions are contravariant in their argument type(s) and covariant in their result type.

### Variance Checks
위에서 Array는 mutable한 속성 때문에 covariant하지 못하다는 문제를 살펴봤었다. mutable한 속성이라는 것은 update 가능하다는 말과 같은데, Array 클래스에서 update 함수의 파라미터의 타입이 어떤 문제를 가지고 있는지 살펴보자. 앞서서 covariant 타입은 함수의 result 타입에만 나타날 수 있다고 말했다.

```
class Array[+T] {
  def update(x: T) ...
}
```

그런데 위의 Array 클래스의 update 함수를 보면, covariant 타입 T가 파라미터에 쓰여졌기 때문에 Array는 covariant 하지 못
한 컨테이너라 할 수 있겠다.

그래서 앞서서 보았던(4.2) Function1은 사실 아래와 같은 형태이다.

```
package scala
trait Function1[-T, +U] {
  def apply(x: T): U
} 
```

그렇다면 List의 경우는 어떨까?
Nil, Cons 클래스의 경우로 살펴보자.

```
package week4

trait List[+T] {
  def isEmpty: Boolean
  def head: T
  def tail: List[T]
}

class Cons[T](val head: T, val tail: List[T]) extends List[T] {
  def isEmpty = false
}

object Nil extends List[Nothing] {
  def isEmpty: Boolean = true
  def head: Nothing = throw new NoSuchElementException("Nil.head")
  def tail: Nothing = throw new NoSuchElementException("Nil.tail")
}

// val x의 return 타입이 List[Nothing]을 상속받는 Nil object 이므로,
// covariant 규칙에 의해 List[String]으로 리턴 타입을 지정할 수 있다.
// List[Nothing] <: List[String]
object test {
  val x: List[String] = Nil
}
```

Nil이 List[Nothing]을 상속하게 만들면 모든 리스트의 서브타입이 된다. 그리고 trait List[T]를 trait List[+T]로 바꿔서 covariant하게 만들어 준다. val x: List[String] = Nil을 입력하게 되면, Nil이 List[Nothing]을 상속받으므로 covariant하게 바뀐 List 속성에 의해서 Nothing 보다 상위 클래스인 String 타입으로 리턴 할 수 있게 되었다.

List 클래스에 다음과 같은 prepend 메서드를 추가해보자.

```
def prepend(elem: T): List[T] = new Cons(eleml, this)
```

컴파일 에러가 난다. 그 이유는 타입 T가 covariant하기 때문에 파라미터에 사용하면 안된다. prepend 메서드가 새로운 리스트를 생성함해도 불구하고 문제가 생기는 이유는 prepend 메서드에 elem의 타입이 T이기 때문이다. 타입 T가 covariant하다면 반드시 result type에만 사용해야 한다.


### Prepend Violates LSP
prepend 메서드가 왜 Liskov Substitution Principle을 위반했는지 알아보자
xs의 타입이 List[IntSet]인 경우에는 문제가 없다.

```
xs.prepend(Empty)
```

하지만 ys의 타입이 List[NonEmpty]라고 했을 때는 문제가 있다.

```
ys.prepend(Empty)
```

NonEmpty 타입이 들어와야 할 자리에 Empty 타입이 들어왔으므로 타입에러가 발생한다. 그래서 이 경우에는 List[NonEmpty]는 List[IntSet]의 서브타입이 될 수 없다.

하지만 prepend 메서드는 immutable list에 실제로 존재한다. 어떻게 이게 가능할까? 답은 lower bound에 있다. U >: T는 U가 T의 부모 타입이라는 말이다. 이렇게 되면, elem이 T보다 상위 타입이 오더라도 문제가 되지 않는다.

```
def prepend[U >: T](elem: U): List[U] = new Cons(elem, list)
```


## 4.5 Decomposition

다음과 같은 class 구조가 있다고 하자

```
trait Expr {
  // classification
  def isNumber: Boolean
  def isSum: Boolean
  // accessor
  def numValue: Int
  def leftOp: Expr
  def rightOp: Expr
}

class Number(n: Int) extends Expr {
  def isNumber: Boolean = true
  def isSum: Boolean = false
  def numValue: Int = n
  def leftOp: Expr = throw new Error("Number.leftOp")
  def rightOp: Expr = throw new Error("Number.rightOp")
}

class Sum(e1: Expr, e2: Expr) extends Expr {
  def isNumber: Boolean = false
  def isSum: Boolean = true
  def numValue: Int = throw new Error("Sum.numValue")
  def rightOp: Expr = e1
  def leftOp: Expr = e2
}
```

무척 쓸모 없어 보이는 메서드들이 여럿 보인다. 일단은 더 나은 코드를 설명하기 위한 단계이므로 참고 살펴보자.
그리고 위의 클래스 구조를 evaluation하는 간단한 인터프리터 함수인 eval이 다음과 같다

```
def eval(e: Expr): Int = {
  if (e.isNumber) e.numValue
  else if (e.isSum) eval(e.leftOp) + eval(e.rightOp)
  else throw new Error("Unknown expression " + e)
}
```
이때 다음과 같은 코드가 있다면, 우선 eval 함수가 실행되면서 e가 어떤 타입인지 찾기 위해 classification method인 isSum으로 Sum 타입인지 찾을 것이다. 그리고 그 안의 두 인자가 각각 Number 이므로 또다시 eval 함수 내에서 isNumber에 의해 Number 타입인지 찾을 수 있을 것이다. 뭔가 비효율적으로 보인다.

```
eval(Sum(Number(1), Number(2))) = 3
```

여기서 만약에 아래와 같은 두개의 클래스가 추가 된다면 어떨까?

```
class Prod(e1: Expr, e2: Expr) extends Expr   // e1 * e2
class Var(x: String) extends Expr         // Variable 'x'
```

위의 두 클래스는 Number나 Sum과 마찬가지로 Expr을 상속받으므로 trait Expr의 메서드를 모두 구현해야한다. 그리고 isNum, isSum 과 같은 클래스 타입을 찾기 위한 메서드를 2개(isVar, isProd)더 추가해야 할 것이다. 또 var 값을 가져오기 위한 name 메서드도 추가되서 총 3개가 추가된다. 위의 구조에서만 15개의 메서드가 있는데, 단 2개의 클래스만 추가하더라도 더 필요한 메서드가 25개(Expr에 3개, Number에 3개, Sum에 3개, 그리고 새로운 클래스에 각각 8개)나 된다. 이건좀 아닌거 같다.

메서드를 좀 줄여보자
자바에서 사용하는 type test, type cast 메서드를 이용한다.

```
Scala           Java
x.isInstanceOf[T]     x instanceof T    // type test
x.asInstanceOf[T]     (T) x       // type cast
```

평가함수인 eval을 조금 고쳐보자.

```
def eval(e: Expr): Int = {
  if (e.isInstanceOf[Number])
    e.asInstanceOf[Number].numValue
  else if (e.isInstanceOf[Sum])
    eval(e.asInstanceOf[Sum].leftOp) + eval(e.asInstanceOf[Sum].rightOp)
  else throw new Error("Unknown expression " + e)
}
```

자바에서 사용하는 타입 test 함수인 instanceof와 타입 캐스팅 하는 방법을 적용하였다. 스칼라에서는 각각의 방법을 함수로 만들어 두었다. 이 방법을 사용하면 위에서 보았던 classification 메서드(isNum, inSum)를 사용할 필요가 없다. 대신에 타입 체크 및 캐스팅 함수가 low-level 함수이기 때문에 불안정한다는 단점이 있다. 

Object-Oriented Decomposition을 이용한 또다른 해법을 살펴보자

```
trait Expr {
  def eval: Int
}

class Number(n: Int) extends Expr {
  def eval: Int = n
}

class Sum(e1: Expr, e2: Expr) extends Expr {
  def eval: Int = e1.eval + e2.eval
}
```

각각의 클래스에 eval 함수를 구현하였다. 각 클래스에 맞게 구현되기 때문에 accessor 함수들도 불필요하다. 이제 많이 깔끔해졌다. 하지만 문제는 여전히 있다. rait에 하나의 메서드가 추가된다면, 나머지 클래스에 모두 구현해야한다는 점이다. 또다른 문제가 있다.

```
a * b + a * c = a * (b + 3)
```

위와 같이 축약하기 어렵다. 왜냐하면 이것은 non-local simplification이기 때문이다. 이것은 single object의 메서드로 캡슐화 할 수 없다. sub-tree를 모두 테스트하고 접근해야하는 문제가 있다. 


## 4.6 Pattern Matching
이전챕터에서 Decomposition을 시도한 몇가지 방법은 아래와 같다.

* Classification and access methods: quadratic explosion
* Type tests and casts: unsafe, low-level
* Object-oriented decomposition: does not always work, need to touch all classes to add a new method.

classification과 accessor 의 주 목적은 아래와 같다.
* Which subclass was used?
* What were the arguments of the constructor?

보통 사용되는 new Sum(e1, e2)와 같은 형태의 생성자를 스칼라는 case class라는 문법을 통해서 자동으로 Pattern Matching 시켜준다. 

```
// 두개의 case class
trait Expr
case class Number(n: Int) extends Expr
case class Sum(e1: 
Expr, e2: Expr) extends Expr

// 실제 apply 메서드의 형태
// Number(1), Sum(2, 3)과 같이 호출될꺼다
object Number {
  def apply(n: Int) = new Number(n)
}
object Sum {
  def apply(e1: Expr, e2: Expr) = new Sum(e1, e2)
} 

// eval 함수를 이용해서 패턴매칭, 
// 파라미터 e가 Number냐 Sum이냐에 따라서 자동으로 선택되어 처리
def eval(e: Expr): Int = e match {
  case Number(n) => n
  case Sum(e1, e2) => eval(e1) + eval(e2)
}
```

### Match Syntax rules
* match is followed by a sequence of cases, pat => expr.
* Each case associates an expression expr with a pattern pat.
* A matchError exception is thrown if no pattern matches the value of the selector.

패턴은 Number, Sum과 같은 contructor로 만들어지며, 인자(variables)는 반드시 소문자로 시작해야한다. 그리고 한 pattern 안에 같은 파라미터 문자를 쓰면 안된다. 상수는 null, true, false를 제외하고는 반드시 대문자로 시작해야한다. 마지막으로 wildcard pattern인 '_'은 해당 파라미터를 신경쓰지 않겠다는 것이다. 대체로 해당 case에서 사용되지 않는 파라미터에 '_'를 사용한다.

eval 함수를 trait Expr에 넣어보자.

```
trait Expr {
  def eval: Int = this match {
    case Number(n) => n
    case Sum(e1, e2) => e1.eval + e2.eval
  }
}
```





## 4.7 Lists

가장 기본적인 리스트 형태는 아래와 같이 정의할 수 있다.

```
// List(X1, ..., Xn)
val fruit: List[String] = List("Apples", "oranges", "pears")
val nums: List[Int] = List(1, 2, 3, 4)
val diag3: List[List[Int]] = List(List(1, 0, 0), List(0, 1, 0), List(0, 0, 1))
val empty: List[Nothing] = List()
```

스칼라에서 List와 Array는 중요한 두가지 차이가 있다.
* List are immutable - the elements of a list cannot be changed
* Lists are recursive, while arrays are flat

또한 스칼라에서는 construction operation인 ::(cons라 부름, 지난주의 prepend 함수랑 동일하다)를 이용하여 좀더 간단하게 리스트를 만들 수 있다. cons는 right-associative 연산이기 때문에 우측에서부터 왼쪽으로 하나씩 붙여 나간다는 생각으로 사용하면 된다. 위의 리스트 들을 cons를 이용해서 작성해보면 다음과 같다.

```
fruit = "apples" :: ("oranges" :: ("pears" :: Nil))
fruit = "apples" :: "oranges" :: "pears" :: Nil

nums = 1 :: (2 :: (3 :: (4 :: Nil)))
nums = 1 :: 2 :: 3 :: 4 :: Nil

empty = Nil
```

right-associative 연산이기 때문에 실제 컴파일러는 위의 연산(nums)을 다음과 같이 해석한다.

```
nums = 1 :: 2 :: 3 :: 4 :: Nil
Nils.::(4).::(3).::(2).::(1)
```

### sorting Lists

재귀를 이용한 Insertion Sort

```
def isort(xs: List[Int]): List[Int] = {
  xs match {
    case Nil => List()
    case y :: ys => insert(y, isort(ys))
  }
}

def insert(x: Int, xs: List[Int]): List[Int] = {
  xs match {
    case Nil => List(x)
    case y :: ys => {
      if (x < y)  x :: xs
      else y :: insert(x, ys)
    }
  }
}
```
