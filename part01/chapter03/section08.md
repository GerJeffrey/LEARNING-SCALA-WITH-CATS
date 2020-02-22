
## 3.8 另外一面:部分统一

在3.2节，我们看到了奇怪的编译器错误。下面的代码如果我们增加 **-Ypartial-unification** 编译器标志那么就可以完美的编译通过：

`import cats.Functor
import cats.instances.function._
import cats.syntax.functor._

val func1 = (x: Int) => x.toDouble
val func2 = (x: Double) => y * 2
val func3 = func1.map(func2)`

如果取消  **-Ypartial-unification** 编译器标志， 那么就会编译出错。

很明显，"部分统一"是一种可选的编译器行为，不做标记我们的代码将不能编译通过。我们花费一点时间来学习一下这种行为，并且相关的方法。

### 3.8.1 统一类型构造器

为了编译类似于 func1.map(func2)的表达式，编译器将会为Function1查找Functor。然而，Functor需要接收一个参数的类型构造器：
`trait Functor[F[_]]{
  def map[A, B](fa: F[A])(func: A => B):F[B]
  }`

并且Function1有两个类型参数(函数参数和结果类型)：

`trait Function1[-A, +B]{
  def appli(arg: A):B
  }`

编译器不得不修复Function1的两个参数，并且给Functor传入正确类型的类型构造器。它有两个选择：

`type F[A] = Int => A
type F[A] = A => Double`

我们知道前面的一个是正确选择。然而，Scala编译器的早期版本并不能推断出这一点。这种"非著名"限制，编号为SI-2712，可以组织不同类型构造的"统一"。这个编译器限制目前已经被修复，尽管我们还是需要通过给build.sbt增加一个标记来完成：

`scalacOptions += "-Ypartial-unification"`

### 3.8.2 从左到右的消除













#







































##
