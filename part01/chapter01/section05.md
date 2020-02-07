## 1.5 例子： Eq
我们以另外一个类型类cats.Eq来结束本章节的内容。设计Eq用来支持类型安全相等，并且使用Scala内建的==操作符。
几乎所有Scala开发者写过如下的代码：

` List(1, 2, 3).map(Option(_)).filter(item => item == 1)
// res0: List[Option[Int]] = List()`

是的，很多时候原则上的正确并不会让你犯这种小错误。filter语句中的断言在遇到比较Int和Option[Int]的时候，总是返回错误。
这是一个程序员错误，我们应该比较与Some(1)比较而不是1。然而，这不是一种技术上的类型错误，因为==可以对任意的一对儿对象进行比较，而不管他们是什么类型。Eq就是设计用来增加某些类型安全来确保相等性检查，从而避免这个问题。
### 1.5.1 相等性，自由和博爱
我们可以使用Eq来定义任何给定类型的类型安全的相等性：

`package cats
trait Eq[A] {
def eqv(a: A, b: A): Boolean
// other concrete methods based on eqv...
}`

定义在cats.syntax.eq中的接口语法，提供了两个用来检查执行相等性的方法，这两个方法在实例Eq[A]的域中：
+  === 比较两个对象的相等性；
+  =!= 比较两个对象的不相等性。

### 1.5.2 比较整型
让我们来看看一些例子。首先，我们导入类型类：
import cats.Eq
现在，我们导入Int的实例：

`import cats.instances.int._ // for Eq val eqInt = Eq[Int]`

我们可以使用eqInt来测试相等性：

`eqInt.eqv(123, 123)
// res2: Boolean = true
eqInt.eqv(123, 234)
// res3: Boolean = false`

不像Scals的==方法，如果我们想要使用eqv来比较不同类型的对象，那么将会得到编译器错误：

`√\eqInt.eqv(123, "234")
// <console>:18: error: type mismatch; // found : String("234")
// required: Int
// eqInt.eqv(123, "234")
//`


  我们也可以导入接口语法cats.syntax.eq 来使用 === 和 =!= 方法:

  `import cats.syntax.eq._ // for === and =!= 123 === 123
// res5: Boolean = true
123 =!= 234
// res6: Boolean = true`

我们再次比较不同类型的对象，那么还是会产生编译器错误：

`123 === "123"
// <console>:20: error: type mismatch;
//  found   : String("123")
//  required: Int
//        123 === "123"
//                ^`

### 1.5.3 比较Option

我们来看看一个很有趣的例子—Option[Int]。为了比较类型Option[Int]的对象，我们需要导入作为Int的Option的Eq实例：
`import cats.instances.int._ // for Eq
import cats.instances.option._ // for Eq`

现在，我们尝试一些比较：

`Some(1) === None
// <console>:26: error: value === is not a member of Some[Int] // Some(1) === None
// ^`

我们接收到编译器的错误，因为类型并不匹配。我们在Eq实例的作用域内有Int和Option[Int]，但是比较的值的类型是Some[Int]。为了修复这个问题，我们将参数重塑为 Option[Int]:

` (Some(1) : Option[Int]) === (None : Option[Int])
// res9: Boolean = false`

我们可以以更友好的方式来使用标准库中的Option.apply和Option.empty方法:

`Option(1) === Option.empty[Int]
// res10: Boolean = false`

或者导入从 cats.syntax.option导入语法：

`import cats.syntax.option._ // for some and none 1.some === none[Int]
// res11: Boolean = false
1.some =!= none[Int]
// res12: Boolean = true`

###1.5.4 比较自定义类型

我们可以使用Eq.instance方法来定义我们自己的Eq实例，这个方法接受一个类型为(A, A) => Boolean的方法，并返回一个=Eq[A]:

`import java.util.Date
import cats.instances.long._ // for Eq
implicit val dateEq: Eq[Date] = Eq.instance[Date] { (date1, date2) =>
date1.getTime === date2.getTime }
val x = new Date() // now
val y = new Date() // a bit later than now
x === x
// res13: Boolean = true
x === y
// res14: Boolean = false`

### 1.5.5练习 相等性，自由和特性

实现一个我们上面的Cat的Eq实例：

`final case class Cat(name: String, age: Int, color: String)`

使用这个实例来对比下面的对象对的相等性和不相等性：

`val cat1 = Cat("Garfield",   38, "orange and black")
val cat2 = Cat("Heathcliff", 33, "orange and black")
val optionCat1 = Option(cat1)
val optionCat2 = Option.empty[Cat]`
