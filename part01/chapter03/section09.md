
## 1.7 小结

我们本章已经用第一个具备精致函数式编程的类型类完成了一个大的里程碑：
• **Semigroup** 代表加和或者组合操作；
• **Monoid** 通过增加一个幺元或者“0”元素来扩展了 **Semigroup** 。
我们可以通过导入这三种事物来使用 **Semigroup** 和 **Monoid**，即类型类本身，我们关注的类型类实例，以及给我提供 **|+|** 操作的 **semigroup** 语法：

`import cats.Monoid
import cats.instances.string._ // for Monoid import cats.syntax.semigroup._ // for |+|
"Scala" |+| " with " |+| "Cats"
// res0: String = Scala with Cats`

只要作用域里面有正确的实例，我们就可以对任何我们需要的事物进行叠加操作：

`import cats.instances.int._ // for Monoid import cats.instances.option._ // for Monoid
Option(1) |+| Option(2)
// res1: Option[Int] = Some(3)
import cats.instances.map._ // for Monoid
val map1 = Map("a" -> 1, "b" -> 2) val map2 = Map("b" -> 3, "d" -> 4)
map1 |+| map2
// res3: Map[String,Int] = Map(b -> 5, d -> 4, a -> 1) import cats.instances.tuple._ // for Monoid
val tuple1 = ("hello", 123)
val tuple2 = ("world", 321)
tuple1 |+| tuple2
// res6: (String, Int) = (helloworld,444)`

我们同样也可以通过自定义的 **Monoid** 实例来写更加泛化的代码：

`def addAll[A](values: List[A])
      (implicit monoid: Monoid[A]): A =
values.foldRight(monoid.empty)(_ |+| _) addAll(List(1, 2, 3))
// res7: Int = 6
addAll(List(None, Some(1), Some(2)))
// res8: Option[Int] = Some(3)`

**Monoid** 是通向Cats的大门。容易理解还好用。然而，这只是Cats给我带来的抽象能力的冰山一角。在下一章节，我们将会学习到 **functor** ，这是一种具有map方法特点的类型类。那就从这里开启有趣之旅吧！














#







































##
