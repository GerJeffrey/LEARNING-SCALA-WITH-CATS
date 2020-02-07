
## 2.3 练习: Monoids的真相

我们已经看到了几个 **monoid** 的例子，这里还有更多的例子可以看到。我们先考虑 **Boolean** 。这个类型可以定义多少 **monoid**？ 对于每一个 **monoid**，定义 **combine** 和 **empty** 操作符合 **monoid** 的法则。我们以如下的定义开始：

`trait Semigroup[A] {
  def combine(x: A, y: A): A
}
trait Monoid[A] extends Semigroup[A] {
  def empty: A
}
object Monoid {
  def apply[A](implicit monoid: Monoid[A]) =
monoid }`


## B.1  Monoids的真相

Boolean有四个 **monoid** ！第一个， 是逻辑与(&&), 操作符为 &&，幺元是true：

`implicit val booleanAndMonoid: Monoid[Boolean] =
  new Monoid[Boolean] {
    def combine(a: Boolean, b: Boolean) = a && b
    def empty = true
  }`

第二个，是逻辑(or), 操作符为||,幺元为false：

`implicit val booleanOrMonoid: Monoid[Boolean] =
  new Monoid[Boolean] {
    def combine(a: Boolean, b: Boolean) = a || b
    def empty = false
  }`


第三，操作符为异或(xor),幺元为false：

`implicit val booleanEitherMonoid: Monoid[Boolean] = new Monoid[Boolean] {
    def combine(a: Boolean, b: Boolean) =
      (a && !b) || (!a && b)
    def empty = false
  }`

最后一种，操作符为同或(xnor),幺元为true：

`implicit val booleanXnorMonoid: Monoid[Boolean] =
  new Monoid[Boolean] {
    def combine(a: Boolean, b: Boolean) =
      (!a || b) && (a || !b)
    def empty = true
  }`

  每一个例子中，幺元法则非常清晰。类似的，**combine** 的结合律也在例子中列举出来。














































*
