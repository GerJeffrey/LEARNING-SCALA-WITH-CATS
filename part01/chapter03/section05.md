## 3.5 Cats中的Functor

让我们学习一下Cats中的 **functor** 的实现。我们会像学习monoid一样学习到三个方面：类型类，实例和syntax

### 3.5.1 Functor 类型类

**functor** 的类型类是 cats.Functor 。我们通过标准的伴生对象里面的 Functor.apply 获得实例。通常，默认的实例被安排在 cats.instances包内：

`import scala.language.higherKinds
import cats.Functor
import cats.instances.list._
import cats.instances.option._

val list1 = List(1,2,3)
val list2 = Functor[List].map(list1)(_ * 2)
val option1 = Option(123)
val option2 = Functor[Option].map(option1)(_.toString)`

**Functor** 也提供 **lift** 函数，其将一个类型为A=>B的函数提升为类型为F[A] => F[B]:

`val func = (x:Int) => x +1
val liftedFunc = Functor[Option].lift(func)
liftedFunc(Option(1))`

### 3.5.2 Functor syntax

Functor syntax提供的最主要的方法就是map。使用Options和Lists来演示这个函数比较困难，因为scala都给他们内建了map函数。我们用两个例子来说明。

第一，我们先来看一边函数。Scala的Function1类型没有map方法(有一个andThen方法)，因为就没有map方法的冲突问题：

`import cats.instances.function._
import cats.syntax.functor._

val func1 = (a: Int) => a + 1
val func2 = (a: Int) => a * 2
val func3 = (a: Int) => a + "!"
val func4 = func1.map(func2).map(func3)
func4(123)`

我们再来看另外一个例子。这次我们会将functor进行抽象，因为我们不会看到任何实体类型。我们可以写一个方法实现无论functor的上下文是什么的等式：

`def doMath[F[_]](start:F[Int])
 (implicit functor:Functor[F]):F[Int] = start.map(n => n + 1 * 2)

 import cats.instances.option._
 import cats.instances.list._

 doMath(Option(20))
 doMath(List(1,2,3))

 import cats.instances.option._
 import cats.instances.list._

 doMath(Option(20))
 doMath(List(1,2,3))`

 为了演示这一点如何工作，我们需要看一下cats.syntax.functor中map的定义。这里的代码简单的代码版本：

  `implicit class FunctorOps[F[_], A](src: F[A]){
    def map[B](func: A => B)(implicit functor:Functor[F]):F[B] = {
      functor.map(src)(func)
    }
  }`

编译器不论有没有内建map都会插入一个map方法:

`foo.map(value => value + 1)`

FunctorOps的map方法需要一个隐式的Functor作为参数。这就意味着只有F的Functor在作用域内才能编译。如果没有，就会出现编译错误：

 `final case class Box[A](value: A)
 val box = Box[Int](123)
 box.map(value => value + 1)`

### 3.5.3 自定义类型实例















































#
