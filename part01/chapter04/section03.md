
## 4.3 幺元Monad

在前面的章节我们通过写了一个抽象的不同monad展示了Cats的flatMap和map syntax：

`import scala.language.higherKinds
import cats.Monad
import cats.syntax.functor._
import cats.syntax.flatMap._

def sumSquare[F[_]:Monad](a: F[Int], b:F[Int]):F[Int]=
for{
  x <-a
  y <- b
  }yield x*x + y*y`

  这个方法在Option和List上工作的很好，但是却不能传入一个普通值：

  `sumSquare(3,4)`

  如果我们可以让sumSquare支持monad或者完全不是monad的参数，那么这个方法将会非常有用。这需要我们在monad和非monad之上进行高级抽象。幸运的是，Cats提供了Id类型来跨越这个裂隙：

   `import cats.Id
   sumSquare(3: Id[Int], 4:Id[Int])`

Id允许我们使用普通值调用monad方法。然而，精确的语义理解起来去不是那么容易。我们传递给sumSquare的参数转换为Id[Int]传入，并且接收其返回的Id[Int].

这期间发上了什么？这里是Id定义的解释：

`package cats
type Id[A] = A`

Id实际上是一个类型别名，将一个原子类型转换为单个参数赖姓构造器。我们可以将任何类型值转换到对应的Id：

`"Dave":Id[String]

123: Id[Int]

List(1,2,3):Id[List[Int]]`

cats为Id提供了不同类型类的实例，包括Functor和Monad。这些可以让我们用普通值来调用map、flatMap和pure：

`val a = Monad[Id].pure(3)
val b = Monad[Id].flatMap(a)(_ + 1)
import cats.syntax.functor._
import cats.syntax.flatMap._

for{
  x <- a
  y <- b
  }yield x + y`

这种在monad和非monad代码之上的抽象能力是非常有力的。举个例子，我们可以在生产中使用Future允许异步代码，同步测试使用Id。我们可以在第8章中学习到这个例子。

### 4.3.1 练习：单子保密单元

为Id类型实现pure,map和flatMap.我们通过这个实现发现了什么？

#### D2 单子保密单元

让我们通过定义方法签名开始：

`import cats.Id

def pure[A](value: A):Id[A]=???
def map[A, B](initial: Id[A])(func: A => B):Id[B] = ???

def flatMap[A, B](initial: Id[A])(func: A=>Id[B]): Id[B] = ???
`

现在让我们看一下每一个方法。pure操作从A产生了ID[A]。但是A和Id[A]是相同类型！我们所做的所有事情都是初始值：

`def pure[A](value: A): Id[A] = value

pure(123)`

map方法带一个类型为Id[A]的参数，应用类型为A=>B的函数，并返回一个Id[B]。但是Id[A]是A，Id[B]为B。我们所做的就是调用这个函数，并且没有打包和拆包的需要:

`def map[A, B](initial: Id[A])(func: A=>b):Id[B] = func(initial)

map(123)(_ * 2)`

flatMap和map实际上是相同的：

`def flatMap[A, B](initial: Id[A])(func: A => Id[B]):Id[B] = func(initial)

flatMap(123)(_ * 2)
`
这个节点在我们对functor和monad理解为序列化类型类。每一个类型类都允许我们序列化操作而忽略某中复杂性。在Id的这种例子中，并不具有复杂性，并且是的map和flatMap做的事情相同。

主要到我们没有不得不在上述的方法体中写类型注解。编译器能够根据他们使用的上下文将类型A翻译为Id[A]或者相反。

我们看到的唯一的限制是Scala在搜索隐式转换的时候不能统一类型和类型构造器。因此，我们需要将Int在调用sumSquare的开始的部分重定义为Id[Int]:

`sumSquare(3: Id[Int], 4: Id[Int])`









































*
