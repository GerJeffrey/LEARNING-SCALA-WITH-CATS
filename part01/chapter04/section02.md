## 4.2 Cats中的monad

是展示标准Cats协议中的monad的时候了。像正常情况一样，我们以类型类，实例和语法的步骤来看。

### 4.2.1 Monad类型类

monad类型类是 **cats.Monad**. Monad 扩展自两个类型类， FlatMap，其提供了flatMap方法， 和Applicative，其提供了pure。Applicative也扩展自Functor，其给每一个Monad提供了一个map函数，我们在上述的练习中可以看到。我们会在第6章讨论Applicative。

这里有直接使用了pure、flatMap和map的例子：

`import cats.Monad
import cats.instances.option._
import cats.instances.list._

val opt1 = Monad[Option].pure(3)
val opt2 = Monad[Option].flatMap(opt1)(a => Some(a + 2))
val opt3 = Monad[Option].map(opt2)(a => 100 * a)

val list1 = Monad[List].pure(3)
val list2 = Monad[List].flatMap(List(1,2,3))(a=>List(a, a * 10)
val list3 = Monad[List].map(list2)(a => a + 123)`

Monad提供了许多其他方法，包括functor的所有的方法。可以通过scaladoc获取更多信息。

### 4.2.2 默认实例

Cats通过cats.instances为标准库中所有monad都提供了实例(Option,List,Vector等等)：

`import cats.instances.option._

Monad[Option].flatMap(OPtion(1))(a => Option(a * 2))

import cats.instances.list._

Monad[List].flatMap(List(1,2,3))(a => List(a, a * 10))

import cats.instances.vector._

Monad[Vector].flatMap(Vector(1,2,3))(a => Vector(a, a * 10))`

Cats也为Future提供了Monad。不想Future类型自身的方法，monad的pure和flatMap方法不能接收隐式ExcecutionContext参数(因为参数并非Monad特质的参数的一部分)。为了变通，Cats需要我们在调用Future的Monad的时候，ExecutionContext需要在作用域内：

`import cats.instances.future._
import scala.concurrent._
import scala.concurrent.duration._

val fm = Monad[Future]`

将ExcutionContext引入作用域可以修复调用实例时的隐式转换：

`import scala.concurrent.ExecutionContext.Implicits.global

val fm = Monoad[Future]`

Monad实例使用被捕获的ExecutionContext替换调用pure和flatMap：

`val future = fm.flatMap(fm.pure(1))(x => fm.pure(x + 1))

Await.result(future, 1.second)`

另外，Cats提供了我们没有在标准库中提供的新monad。后面我们会熟悉这些内容。

### 4.2.3 Monad syntax

monad的syntax可以通过三个方面来讲：

+ cats.syntax.flatMap提供了flatMap的syntax；
+ cats.syntax.functor提供了map的syntax；
+ cats.syntax.applicative提供了pure的syntax;

实际上，我们可以直接导入cats.implicits将所需要的所有东西全部导入。然而，为了清晰的说明，我们分开来导入。

我们使用pure来构建monad实例。我们经常需要指定类型参数来区分我们需要的特殊实例：

`import cats.instances.option._
import cats.instances.list._
import cats.syntax.applicative._

1.pure[Option]
1.pure[List]`

像Option和List一样来直接在Scala monad上展示flatMap和map方法是比较困难的， 因为他们定义了自己的这些方法的显式版本。我们会写一个泛化函数来执行包裹在用户选择的monad中的参数计算：

`import cats.Monad
import cats.syntax.functor._
import cats.syntax.flatMap._
import scala.language.higherKinds

def sumSquare[F[_]: Monad](a: F[Int], b:F[Int]): F[Int]  = a.flatMap(x => b.map(y => x*x + y*y))

import cats.instances.option._
import cats.instances.list._

sumSquare(Option(3),Option(4))

sumSquare(List(1,2,3), List(4, 5))
`

我们可以使用for comprehension来重写这些代码。编译器会在重新flatMap和map的时候使用我们的Monad来插入正确的隐式转换

`def sumSquare[F[_]: Monad](a: F[Int], b:F[Int]):F[Int] =

for{
  x <- a
  y <- b
  } yield x*x + y*y

sumSquare(Option(3), Option(4))
sumSquare(List(1,2,3),List(4,5))`

这或多或少让我们了解关于Cats中的monad。现在让我们看一下某些在标准库中不存在的有用的monad实例。






























#
