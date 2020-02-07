
## 2.4 练习: 集合的Monoid

对于集合的 **monoid** 和 **semigroup**

## B.2 All 集合的Monoid

集合的并集，操作符为union，幺元为空集合：

`implicit def setUnionMonoid[A]: Monoid[Set[A]] =
  new Monoid[Set[A]] {
    def combine(a: Set[A], b: Set[A]) = a union b
def empty = Set.empty[A] }`

我们需要将setUnionMonoid定义为一个方法，而不是作为类型A的参数的值。类型参数允许我们使用相同的定义来调用任何数据类型的Set的Monoid:

`val intSetMonoid = Monoid[Set[Int]]
val strSetMonoid = Monoid[Set[String]]
intSetMonoid.combine(Set(1, 2), Set(2, 3))
// res2: Set[Int] = Set(1, 2, 3)
strSetMonoid.combine(Set("A", "B"), Set("B", "C")) // res3: Set[String] = Set(A, B, C)`

Set差集是一个 **semigroup**，而不是一个 **monoid** ，因为没有幺元：

`implicit def setIntersectionSemigroup[A]: Semigroup[Set[A]] = new Semigroup[Set[A]] {
    def combine(a: Set[A], b: Set[A]) =
      a intersect b
}`

Set补集和差集不满足结合律，因为它们既不能作为 **monoid** 也不能作为 **semigroup** 。然而，对称差却可以作为一个monoid：

`implicit def symDiffMonoid[A]: Monoid[Set[A]] =
  new Monoid[Set[A]] {
    def combine(a: Set[A], b: Set[A]): Set[A] =
      (a diff b) union (b diff a)
def empty: Set[A] = Set.empty }`
































#
