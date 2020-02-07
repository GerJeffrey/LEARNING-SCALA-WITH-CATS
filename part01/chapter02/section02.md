## 2.2 Semigroup的定义

**semigroup** 仅仅是 **monoid** 的 **combine** 部分。 很多数据类型我们没有办法定义 **empty** 元素，但不排除很多 **semigroup** 就是 **monoid**。比如，我们前面看到的序列拼接和整数加和就是 **monoid** 。如果我们严格控制到非空序列和负整数，那么我们依然不能定义一个精确的 **empty** 。 Cats有一个NonEmptyList数据类型，这个数据类型实现了 **semigroup**，但是没有实现 **monoid**。通过简单的例子，我们可以更加精确的定义Cats的 **monoid**：

`trait Semigroup[A] {
  def combine(x: A, y: A): A
}
trait Monoid[A] extends Semigroup[A] {
  def empty: A
}`

当我们开始讨论类型类的时候，我们将会经常看到这种继承。这会给我们的代码更加模块化并且容易被重用。如果我们定义了类型A的 **monoid** ，我们就会免费的获得一个 **semigroup** 。类似的，如果一个方法需要类型 **Semigroup[B]** 作为参数，我们也可以传递一个 **Monoid[B]** 来替代它。











































#
