
## 2.1 Monoid的定义

我们在前面演示了几个加和的场景，每一个都有与幺元加和二元操作。这其实就是monoid。正式定义为，如果一个类型A：

+ 具有一个操作 **combine** 类型为 (A,A)=>A
+ 具有一个元素，即类型A的 **empty** 类型

这个定义可以非常容易的转换为Scala代码。下面是Cats中的简化定义：

`trait Monoid[A] {
  def combine(x: A, y: A): A
  def empty: A
}`

除了在加和中提供 **combine** 和 **empty** 操作， **monoid** 必须遵守几条规则。对于类型A的所有值x，y和z，组合必须是符合结合律的，并且空必须是一个幺元（单位元素）：

`def associativeLaw[A](x: A, y: A, z: A)
      (implicit m: Monoid[A]): Boolean = {
m.combine(x, m.combine(y, z)) == m.combine(m.combine(x, y), z)
}
def identityLaw[A](x: A)
      (implicit m: Monoid[A]): Boolean = {
(m.combine(x, m.empty) == x) && (m.combine(m.empty, x) == x)
}`

举个例子，整型减法并非是一个monoid，因为减法没有结合律：

`(1 - 2) - 3
// res15: Int = -4
1 - (2 - 3)
// res16: Int = `

在实践中，当我们写自己的 **Monoid** 实例的时候，我们仅仅需要考虑这些法则。不遵守这个法则的实例都是非常危险的，因为当我们在Cats的体系下运行的时候，可能会产生不可预见的结果。大部分时间，我们可以依靠Cats提供的实例，并假设库作者知道它们如何运作。
