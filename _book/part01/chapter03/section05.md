## 2.5 Cats中的Monoids

现在，我们已经了解了什么是 **Monoid** 了, 我们再来看看他们如何再Cats中实现的。我们再次回顾一下实现的三个方面：类型类，实例和接口。

### 2.5.1 Monoid类型类


cats.kernel.Monoid是 **monoid** 的类型类，可以使用别名cats.Monoid。 **Monoid** 扩展了 cats.kernel.Semigroup, 别名是 cats.Semigroup。当我们要使用 **monoid** 类型类的时候，可以导入这些包：

 `import cats.Monoid
 import cats.Semigroup`

> #### Cats Kernel?

> Cats Kernel是Cats的一个子项目，其提供了一小组不需要整个cats工具箱的库的类型类。这些核心类型类被技术性的定义在 **cats.kernel** 包内，其别名就是 **cats**，因此我们也很少能意识到这些区别。

>  Cats Kernel 类型类覆盖了本书的 **Eq**, **Semigroup**, 以及 **Monoid** 。本书的涉及到的其他类型类都直接被定义在 **cats** 包内。

### 2.5.2 Monoid实例

**Monoid** 的用户接口遵从了标准的 Cats模式：即伴生对象有一个apply方法可以返回特定类型类实例。举个例子，如果我们想要一个String的 **Monoid** 实例，先修正隐式转换的域，我们可以这样来写：

`import cats.Monoid
import cats.instances.string._ // for Monoid
Monoid[String].combine("Hi ", "there") // res0: String = Hi there
Monoid[String].empty`

其相当于：

`Monoid.apply[String].combine("Hi ", "there") // res2: String = Hi there
Monoid.apply[String].empty
// res3: String = ""`

我们知道 **Monoid** 扩展自 **Semigroup**。如果我们不需要 **empty** ，我们相当于这样写：

`import cats.Semigroup
Semigroup[String].combine("Hi ", "there") // res4: String = Hi there`

在第一章中，我们已经了解到 **Monoid** 的实例的标准组织方式了。举个例子，我们想要 **Int** 的实例，那么我们导入 **cats.instances.int** 即可:

`import cats.Monoid
import cats.instances.int._ // for Monoid
Monoid[Int].combine(32, 10)`

类似的，我们可以通过 **cats.instances.int** 和 **cats.instances.option** 组合出=Monoid[Option[Int]]:

`import cats.Monoid
import cats.instances.int._ // for Monoid import cats.instances.option._ // for Monoid
val a = Option(22)
// a: Option[Int] = Some(22)
val b = Option(20)
// b: Option[Int] = Some(20)
Monoid[Option[Int]].combine(a, b)
// res6: Option[Int] = Some(42)`

参考第1章，可以对这些导入加深理解。

### 2.5.3 Monoid Syntax

Cats以 **|+|** 操作符为 **combine** 方法提供 **syntax** 支持。因为从技术溯源来讲，**combine** 来自于 **Semigroup** ，因此，我们通过导入 **cats.syntax.semigroup** 来获取 **syntax** 支持：

`import cats.instances.string._ // for Monoid import cats.syntax.semigroup._ // for |+|
val stringResult = "Hi " |+| "there" |+| Monoid[String].empty // stringResult: String = Hi there
import cats.instances.int._ // for Monoid val intResult = 1 |+| 2 |+| Monoid[Int].empty
// intResult: Int = 3`

### 2.5.4 练习：综合这一切

SuperAdder v3.5a-32是世界上第一个选择加和在一起的数字。 main函数中有def add(items: List[Int]): Int的签名。在一次意外中，这些代码被删除了。需要重写这个方法并保存起来。

#### B.3 综合这一切

我们可以用0和+将加和写为一个简单的foldLeft：

` def add(items: List[Int]): Int = items.foldLeft(0)(_ + _)`

尽管没有人强制我们这么做，我们同样可以用 **Monoid** 来重写这个fold：

`import cats.Monoid
import cats.instances.int._ // for Monoid import cats.syntax.semigroup._ // for |+|
def add(items: List[Int]): Int = items.foldLeft(Monoid[Int].empty)(_ |+| _)`
















































#
