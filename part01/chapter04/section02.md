## 4.2 Cats中的monad

是展示标准Cats协议中的monad的时候了。像正常情况一样，我们以类型类，实例和语法的步骤来看。

### 4.2.1 Monad类型类

monad类型类是 **cats.Monad**. Monad 扩展自两个类型类， FlatMap，其提供了flatMap方法， 和Applicative，其提供了pure。Applicative也扩展自Functor，其给每一个Monad提供了一个map函数，我们在上述的练习中可以看到。我们会在第6章讨论Applicative。

这里有直接使用了pure、flatMap和map的例子：

`import cats.Monad
import cats.instances.option._
import cats.instances.list._

val opt1 = Monad[Option].pure(3)
val opt2 = Monad[Option].flatMap(opt1)(a => Some(a + 2))
val opt3 = Monad[Option].map(opt2)(a => 100 * a)

val list1 = Monad[List].pure(3)
val list2 = Monad[List].flatMap(List(1,2,3))(a=>List(a, a * 10)
val list3 = Monad[List].map(list2)(a => a + 123)`

Monad提供了许多其他方法，包括functor的所有的方法。可以通过scaladoc获取更多信息。

### 4.2.2 默认实例




























#
