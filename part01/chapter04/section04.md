
## 3.4 另一方面：高阶类型和类型构造器

高阶类型类似于类型的类型。其描述了类型中的“洞”的数量。我们通过可以通过填充这个“洞”产生类型来区分没有“洞”的规则类和带有洞的“类型构造器”。

举个例子，List是带有一个洞的类型构造器。我们通过指定一个参数来生产一个规则类型来填充这个洞，比如List[Int]或者List[A]。这个技巧并不会将类型构造器和泛型混淆一气。List是一个类型构造器，List[A]是一个类型：

`List //带有一个参数的类型构造器
List[A] //使用一个类型参数生产的类型`

还有一对概念比较类似，函数和值。函数是“值构造器”，当给函数提供参数的时候，会产生一个值：

`math.abs //带有一个参数的函数
math.abs(x) //使用一个值参数产生的一个值`

在Scala中，我们使用下划线声明类型构造器。一旦我们声明了类型构造器，我们可以将它们用做简单的标志性符：

`//使用下划线声明F
def myMethod[F[_]] = {
  // 引用F的时候不需要下划线
  val functor = Functor.apply[F]
  }`

类似于在定义函数的时候，我们需要指定函数的参数签名，但是在引用他们的时候就会忽略这些签名：

`// 指定参数声明f
val f = (x: Int) => x * 2
val f2 = f andThen f
`

武装了这些知识之后，我们就可以理解Cats中的Functor的定义允许我们创建任何单参数类型构造器，比如List，Option，Future或者类似于MyFunc等别名的类型构造器

> 语言特征导入

> 高阶类型类在Scala中是高级语言特性。当我们写下A[_ ]这样的语法的时候，需要高级类型类语言特性来压制编译器的警告，我也可以通过导入

>`import scala.language.higherKinds`

> 来实现，或者给build.sbt增加scalacOption来实现：

>`scalacOption += "-language:higherKinds"
`
> 本书采用导入的方式以确保其尽可能的显式。在实际应用中，采用sbt增加scalacOption来实现更好一些。

















#