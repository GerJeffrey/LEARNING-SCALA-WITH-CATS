
## 1.1、类型类的结构

类型类模式中有三个重要的组件：***类型类自身***，***特殊类型的实例***，以及我们暴露给用户的 ***接口方法***。

### 1.1.1、类型类

类型类是一个承载某项我们想要实现的功能的接口，或者API。在Cats中，类型类通过至少带有一个类型参数的特征（Trait）来表示。比如，我们可以通过如下代码表示一个通用目的的“JSON序列化”：

`// Define a very simple JSON AST
sealed trait Json
final case class JsObject(get: Map[String, Json]) extends Json final case class JsString(get: String) extends Json
final case class JsNumber(get: Double) extends Json
case object JsNull extends Json
// The "serialize to JSON" behaviour is encoded in this trait
trait JsonWriter[A] {
  def write(value: A): Json
}`

JsonWriter 是我们这个例子的类型类，通过Json和其子类提供支撑代码。

### 1.1.2 类型类实例

类型类实例提供了我们关心的类型的实现，包括Scala标准库和我们自己定义的类型。
在Scala中，我们通过带有implicit关键字标记的类型类来创建这个类型类的实在类：

`final case class Person(name: String, email: String)
object JsonWriterInstances {
  implicit val stringWriter: JsonWriter[String] =
    new JsonWriter[String] {
      def write(value: String): Json =
        JsString(value)
    }
  implicit val personWriter: JsonWriter[Person] =
    new JsonWriter[Person] {
def write(value: Person): Json = JsObject(Map(
"name" -> JsString(value.name),
"email" -> JsString(value.email) ))
}
// etc...
}
`
### 1.1.3 类型类接口

类型类接口是我们想要暴露给用户的任何功能。接口是可以接受一个类型类实例作为隐式参数的方法。有两种通用的方式来确定一个接口，接口对象和接口语法。

#### 接口对象（Interface Objects ）

创建一个接口的最简单的方法是将方法放入一个单例对象中：
`import JsonWriterInstances._
Json.toJson(Person("Dave", "dave@example.com"))
// res4: Json = JsObject(Map(name -> JsString(Dave), email -> JsString
(dave@example.com)))
`

编译器定位到我们调用的toJson方法没有提供隐式参数。它就开始尝试修复这个问题，搜索到这个相关类型的类型类实例并将其插入到调用的位置：

`son.toJson(Person("Dave", "dave@example.com"))(personWriter)
`

#### 接口语法（Interface Syntax）

我们可以通过接口方法来使用扩展方法来扩展存在的类型。Cats将这种方式称之这个类型类的“语法（syntax）”：

`object JsonSyntax {
  implicit class JsonWriterOps[A](value: A) {
def toJson(implicit w: JsonWriter[A]): Json = w.write(value)
} }
`
我们通过并排import我们需要的类型的实例来使用接口语法：

`import JsonWriterInstances._
 import JsonSyntax._
Person("Dave", "dave@example.com").toJson
// res6: Json = JsObject(Map(name -> JsString(Dave), email -> JsString
(dave@example.com)))
`
编译器再次为我们搜索隐式参数并且填充到相应的位置：

`Person("Dave", "dave@example.com").toJson(personWriter)
`
#### implicitly方法
Scala标准库提供了一个通用的类型类接口，叫做implicitly。这个接口定义的及其简单：
`def implicitly[A](implicit value: A): A =
  value
`
我们可以使用implicitly函数来召唤隐式域里面的任意值。我们先提供我们想要的类型，然后使用implicitly函数来完成剩下的工作：
`import JsonWriterInstances._ // import JsonWriterInstances._
implicitly[JsonWriter[String]]
// res8: JsonWriter[String] = JsonWriterInstances$$anon$1@a5a4735
`
Cats中大部分类型类提供了其他召唤实例的方式。然而， implicitly对于调试来讲是一个很好用的工具。我们可以在正常的代码流中插入implicitly 调用来确保编译器能够找到一个合适的类型类，并且确保没有模棱两可的隐式转换错误。
