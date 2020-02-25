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

我可以简单的通过定义其map方法来定一个一个 **functor** 。尽管cats.instances中有这么个实例，这里我们还是举一个Option的Functor例子。这个实现虽然看起来比较琐碎，我们只是简单的对Option的map调用了而已：

`implicit val optionFunctor: Functor[Option] =
new Functor[Option] {
    def map[A](value: Option[A])(func:A => B):Option[B] = value.map(func)
  }`

有时候，我们需要给我们的实例注入依赖，举个自理，如果我们不得不为Future自定义一个Functor，我们需要考虑future.map参数里面的隐式转换ExcecutionContxt参数。我们不能给 **functor.map** 增加额外参数，因此我们不得不考虑在创建实例的时候注入依赖：

`import scala.concurrent.{Future, ExecutionContext}

implicit def futureFunctor
(implicit ec: ExecutionContext):Functor[Future] =

new Functor[Future]{
  def map[A, B](value:Future[A])(func: A => B):Future[B] = value.map(func)
}
`

当我们调用Future的Functor，或者直接使用Functor.apply，或者通过map的扩展方法，编译器将会在调用栈点通过隐式解析和递归搜索一个ExucutionContext作为定位到的隐式值。这个过程扩展起来看起来就像：

`//我们这样来写
Functor[Future]

//The compiler expands to this first:

Functor[Future](futureFunctor)

//然后就是
Functor[Future](futurFunctor(executionContext))`

### 3.5.4 练习：使用Functor来拓展

为下面的二叉树类型写一个Functor。可以核实Branch和Leaf的实例与预想的工作一致：

`sealed trait Tree[+A]

final case class Branch[A](left:Tree[A],right:Tree[A]) extends Tree[A]

final case class leaf[A](value:A) extends Tree[A]`

语法与写一个List的Functor类型。我们先递归数据结构，应用到每一个我们找到Leaf。functor的规则从直觉上需要我们对Branch和Leaf的节点保持相同的模式和结构：

`import cats.Functors
implicit val treeFunctor:Functor[Tree]=
  new Functor[Tree]{
    def map[A, B](tree:Tree[A])(func:A=>B):Tree[B] =
      tree match{
        case Branch(left,right) =>
          Branch(map(left)(func),map(right)(func))
        case Leaf(value) => Leaf(func(value))
      }
    }

    Branch(Leaf(10),Leaf(20).map(_ * 2))`

  上面的代码会编译出错，其原因在于我们在1.6.1中讨论的不变性问题。编译器能够找到Tree的一个Functor，但是却不能找到Branch和Leaf的Functor。我们加上只能构造器来补充：

  `object Tree{
    def branch[A](left:Tree[A],right:Tree[A]):Tree[A] = Branch(left,right)
    }
    def leaf[A](value:A):Tree[A]=Left(value)
    }`

  现在我们就可以正确的使用Functor了：

  `Tree.leaf(100).map( _ * 2)

  Tree.branch(Tree.leaf(10),Tree.leaf(20)).map(_ * 2)`

















































#
