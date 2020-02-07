
## 1.4  遇见Cats
在前面的部分，我们看到如何在Scala中实现类型类；在本节，我们将会看到Cats中是如何实现类型类的。Cats采用模块化结构写成，这样就允许我们选择我们想要使用的那一种类型类、实例、方法。让我们来使用cats.Show来看看。
Cats中的Show等价于我们在上一个最后部分定义的Printable类型类。它提供了一种对开发者提供更友好的控制台显示方式的机制，而不是使用toString方法。这里是简化的定义：
`package cats
trait Show[A] {
  def show(value: A): String
}`
### 1.4.1 导入类型类
Cats中的类型类定义在cats包中。我们直接从这个包导入Show即可：
`import cats.Show`
Cats类型类的所有伴生类都有一个apply方法可以定位到任何我们制定的一个实例：
`val showInt = Show.apply[Int]
// <console>:13: error: could not find implicit value for parameter
instance: cats.Show[Int]
// val showInt = Show.apply[Int] // ^ `
糟糕，这段代码并不能工作。这个apply方法使用隐式转换来查找单个实例因此我们不得不把这些实例引入到隐式转换域里面。
### 1.4.2 导入默认实例
cats.instances 包提供了一大堆类型的默认实例。我们直接像下面的例子一样导入这些实例。每一个倒入提供了Cats的类型实例对应的指定参数类型：
cats.instances 包提供了一大堆类型的默认实例。我们直接像下面的例子一样导入这些实例。每一个倒入提供了Cats的类型实例对应的指定参数类型：
+ cats.instances.int 提供了Int类型实例
+ cats.instances.string  提供String类型实例
+ cats.instances.list  提供List类型实例
+ cats.instances.option  提供了Option类型实例
+ cats.instances.all 提供了所有Cats中的类型实例

可以通过查看 cats.instances 包来看看里面有些什么类型实例。
我们倒入Show的Int和String实例：
`import cats.instances.int._ // for Show import cats.instances.string._ // for Show
val showInt: Show[Int] = Show.apply[Int] val showString: Show[String] = Show.apply[String] `
### 1.4.3 导入接口语法
我们可以通过导入 cats.syntax.show接口语法使得Show更加容易使用。导入 cats.syntax.show之后会让我们在Show的隐式转换范围内的任何实例扩展show方法：
`import cats.syntax.show._ // for show val shownInt = 123.show
// shownInt: String = 123
val shownString = "abc".show // shownString: String = abc `
Cats可以分别倒入每一种类型类；我们在后面的章节遇到则会进行介绍和讲解。

#### 导入所有
在这本书里面，我们将会在每一个例子中给你确定的指出那个你要使用的导入实例及语法。然而，这种方式适合很多种场景。采用如下的方式之一可以简化导入：
+ 导入cats._ 将会导入所有的cats的所有实例;
+ 导入cats.instances.all._ 将会导入标准库中的所有类型类；
+ 导入cats.syntax.all._ 将会导入所有的语法;
+ 导入cats.implicits._ 将会导入所有标准类型实例和所有的语法。
很多人通过下面的导入开始他们的文件，只有遇到名称冲突和隐式类型混淆的时候才会指定特定的导入：

`import cats._
import cats.implicits._ `
### 1.4.5 定义定制实例
我们可以简单的通过定义一个给定类型的trait来定义Show的实例：
`import java.util.Date
implicit val dateShow: Show[Date] =
  new Show[Date] {
def show(date: Date): String = s"${date.getTime}ms since the epoch."
} `
然而，Cats也提供了一组简易方法来简化这个过程。这里有Show的伴生对象里面的两个构建方法，我们可以用来定义我们自己的类型：
`object Show {
// Convert a function to a `Show` instance: def show[A](f: A => String): Show[A] =
???
// Create a `Show` instance from a `toString` method:
  def fromToString[A]: Show[A] =

??? } `
这些方法可以允许我们通过下面的方式快速构建实例：

`implicit val dateShow: Show[Date] =
Show.show(date => s"${date.getTime}ms since the epoch.") `
正如你所见，这些使用了构建方法之后比之前没有使用的代码简化了很多。Cats中许多类型来提供了helper函数可以像这样来构建实例，而不是形变已存在的实例。
#### 练习: Cat Show 
我们通过Show来实现Cat应用来替换Printable
