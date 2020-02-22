
## 3.7 Cats中的逆变和不变

我们看一下Cat中的逆变和不变functor的实现，其由cats.Contravariant和cats.Invariant类型类所提供，下面是简化版本的代码：

`trait Contravariant[F[_]]{
  def contramap[A, B](fa: F[A])(f: B=>A):F[B]
  }
  def Invariant[F[_]]{
  def imap[A, B](fa:F[A])(f:A=>B)(g:B=>A):F[B]
    }`

### 3.7.1 Cats中的逆变

我们可以通过使用Contravariant.apply方法来调用一个Contravariant实例。Cats为包括Eq，Show和Function1这些类型在内的消耗参数类型提供了实例。这里有一个例子:

`import cats.Contravariant
import cats.Show
import cats.instances.string_

val showString = Show[String]
val showSymbol = Contravariant[Show].contramap(showString)((sym:Symbol))=>s"'${sym.name}"
showSymbol.show('dave)`

为了更方便，我们看可以使用cats.syntax.contravariant，其提供了contramap扩展方法：

`import cats.syntax.contravariant._
showString.contramap[Symbol](_.name).show('dave)`

### 3.7.2 cats中的不变

在其他类型中，Cats提供了Monoid的不变实例。这和我们在3.6.2节中讨论的Condec的例子稍微有些区别。我们回忆一下，这是Monoid看起来的样子：

`package cats
trait Monoid[A]{
  def empty: A
  def combine(x: A, y:A):A
}
`

假设我们想要对Scala的Symbol类型生产一个Monoid。Cats并没有提供Symbol的Monoid，但是却提供了蕾西类型的Monoid，即String。我们可以协一个依赖另外一个empty String的empty和一个combine方法，如下：
1. 将两个Symbol作为接收参数；
2. 将Symbol转换为String；
3. 使用Monoid[String]组合String
4. 将结果转换回到Symbol

我们可以通过imap传入类型为String=>Symbol和Symbol=>String的方法来实现组合。这里有代码，使用了cats.syntax.invariant中提供的imap扩展方法：

`import cats.Monoid
import cats.instances.string._
import cats.syntax.invariant._
import cats.syntax.semigroup._

implicit val symbolMonoid: Monoid[Symbol]=Monoid[String].imap(Symbol.apply)(_.name)

Monoid[Symbol].empty
'a |+| 'few |+| 'words`


























#







































##
