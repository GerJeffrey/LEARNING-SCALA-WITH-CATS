## 1.2 与隐式转换一起工作

在Scala中与类型类工作意味着与隐式值和隐式参数一起工作。我们需要了解更加有效率的使用这一特征的规则。

### 1.2.1 打包隐式转换
Scala语言中令人惊讶的神奇之处在于，任何被标记为implicit的定义必须放在在一个object或者trait中，而不是顶层(top level)。在上面这个例子中，我们将类型类实例打包进叫做JsonWriterInstances的对象中。我们也可以将其放在JsonWriter的伴生对象中。在Scala中将实力放入一个类型类的半生对象中，因为起了隐式转化域的作用。
### 1.2.2  隐式转换域
正如我们上面看到的，编译器通过类型来搜索类型类的候选者。举个例子，对于西门的表达式，编译器将会查找一个类型为JsonWriter[String]的实例:
`Json.toJson("A string!")
`
编译器在调用的地方从隐式转换域里面查找候选实例，这个过程大概由如下几个步骤组成：
* 本地继承定义；
* 被导入(import)的定义；
* 类型类或者参数类(这个例子中是case JsonWriter或者String)的半生对象里面的定义；
如果标记为implicit，定义仅仅会被包含在隐式转换域中。此外，如果编译器查看到多个候选定义，那么将会以“含混不清隐式值错误”而失败：

`implicit val writer1: JsonWriter[String] = JsonWriterInstances.stringWriter
implicit val writer2: JsonWriter[String] = JsonWriterInstances.stringWriter
Json.toJson("A string")
// <console>:23: error: ambiguous implicit values:
// both value stringWriter in object JsonWriterInstances of type =>
     JsonWriter[String]
//  and value writer1 of type => JsonWriter[String]`
隐式转换解析的准确规则比这个更加复杂，但是这种复杂度和这本书没有大的相关性。为了说明问题，我们将类型类实例大致的以四种方式打包：
1. 将他们放在类似于JsonWriterInstances这种对象中；
2. 将他们放在trait中；
3. 将他们放在类型类的伴生对象中；
4. 将他们放在参数类型的半生对象中。
选项1中通过导入类型类使它进入隐式转换域中；选项2通过继承来引入到隐式转换域中；选项3和4，无论我们如何使用类型类实例总是在隐式转换域中。

### 1.2.3 递归隐式转换解析过程
类型类和隐式转换的威力在于编译器在寻找候选定义时组合隐式转换定义的能力。早期，我们暗示过，所有的类型类实例都是隐式值。这一点是非常简单易懂和明确的。我们实际上可以通过如下两种方式来定义实例：
1. 定义一个实体实例作为被请求类型的隐式值；
2. 通过其它类型实例的隐式转换方法来构建实例；
我们为什么要通过其它实例来构建类型类实例？作为讲解实例，可以考虑定一个JsonWriter的Option。我们需要每一个类A的JsonWriter[Option[A]]来对应。我可以尝试通过隐式值来暴力解决：
`implicit val optionIntWriter: JsonWriter[Option[Int]] = ???
implicit val optionPersonWriter: JsonWriter[Option[Person]] = ???
// and so on...
`

但是，很明显，这种方法行不通，因为不具有认可可扩展性。这需要我们提供两种隐式值，一种是A，一种是Option[A]。
幸运的是，我们可以以A的实例为基础抽象出处理Option[A]的通用构造器：
+ 如果option是Some(aValue)，那么aValue就使用A的writer；
+ 如果option是None，你们就返回一个JsNull。
如下是这种方式的实现：

`implicit def optionWriter[A]
(implicit writer: JsonWriter[A]): JsonWriter[Option[A]] =
  new JsonWriter[Option[A]] {
    def write(option: Option[A]): Json =
option match {
case Some(aValue) => writer.write(aValue) case None => JsNull
} } `

这个方法构造了Option[A]的JsonWriter，它通过填充A的特定功能来达成这一目的。当编译器看到如下表达式的时候：

`Json.toJson(Option("A string")) `

首先搜索隐式的JsonWriter[Option[String]]，并且搜索到了JsonWriter[Option[A]]的隐式方法：

`Json.toJson(Option("A string"))(optionWriter[String]) `
递归搜索将JsonWriter[String]作为optionWriter的参数：
`Json.toJson(Option("A string"))(optionWriter(stringWriter)) `
通过这种方式，隐式转换解析就变成了隐式定义可能组合的空间的搜索，用来发现调用正确类型类实例的组合。

> 隐式转换

> 当你采用一种隐式定义来创建一个类型类构造器的时候，需要确认将方法的参数标记为隐式转换的参数。没有这个关键字，编译器将不能干在隐式转换解析的过程中正确的填充这个参数。带有非隐式参数形式的隐式函数是Scala中被称之为隐式转换的模式。这是一种有别于前面讲述过的接口语法( Interface Syntax) ，因为在这种情况下，JsonWriter是一个具备扩展方法的隐式类。隐式转换是一种旧式的编程模式，在现代scala代码中并不多见，也不建议使用。幸运的是，如果你这么干，编译器将会给你以警告。你不得不手动导入scala.language.implicitConversions来打开隐式转换：

> `implicit def optionWriter[A]
(writer: JsonWriter[A]): JsonWriter[Option[A]] =
???
	 //  <console>:18: warning: implicit conversion method optionWriter should be enabled
	 //  by making the implicit value scala.language. implicitConversions visible.
	 //  This can be achieved by adding the import clause 'import scala.language.implicitConversions'
	 //  or by setting the compiler option -language: implicitConversions.
	 //  See the Scaladoc for value scala.language.implicitConversions for a discussion
	 //  why the feature should be explicitly enabled.
	 //         implicit def optionWriter[A]
	 //  ^
	 //  error: No warnings can be incurred under -Xfatal-warnings.
