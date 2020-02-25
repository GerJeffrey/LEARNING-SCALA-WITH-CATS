
## 3.3 Functor的定义

我们看了 **functor** 这些例子：一个封装了一些列计算的类。正式的来讲，**functor** 就是类型F[A]带有 **(A => B) => F[B]** 的 **map** 的操作。通用的类型图在图3.4中。

Cats将 **Functor** 编码为一个类型类，即 **cats.Functor** ，因此这个方法看起来有些不同。其会接受一个初始的F[A]作为参数。这里有一个简单版本的定义：

`package cats
import scala.language.higherKinds
trait Functor[F[_]] {
def map[A, B](fa: F[A])(f: A => B): F[B]
}`

如果你之前没有见过F[ _ ]， 那么我们就绕一点小圈子来讨论类型构造器和高级类类型。我们将会解释我们导入的scala.language。

> Functor规则

> **Functor** 确保无论我们序列化多少小型操作还是将其组合进一个大的功能都会是按顺序一个一个完成。为了确保这一点，必须遵守如下规则：
+ 恒等性：map中调用恒等函数等于没有做任何事情：

> `fa.map(a => a) == fa`

> 组合型：map嵌套两个函数f和g相当于先map f再map g:

> `fa.map(g(f(_))) == fa.map(f).map(g)`















































*
