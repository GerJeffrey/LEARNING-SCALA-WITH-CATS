
## 1.6 控制实例选择
当我们使用类型类的时候，就需要考虑如何控制实例部分的两个议题：
+ 定义在类型和子类型中的实例之间的关系是什么？
 举个例子，如果我们定义了一个JsonWriter[Option[Int]], 表达式Json.toJson(Some(1))会选择这个实例吗？ (记住，Some是Option的子类型)。
+ 我们如何在很多可能性之间选择类型类实例？
  如果我们定义了两个Person的JsonWriters，当我们写下Json.toJson(aPerson)时，会选择哪一个实例？

### 1.6.1 型变

当我们定义类型类的时候，我们可以给类型参数加上型变标注来影响类型类的型变，编译器就有能力在隐式解析过程中选择实例。
Essential Scala这本书的概述里面，型变与子类型相关。我们讲B是A的一个子类型，那么我们就可以在A类型的值出现的地方用B来替换。
当类型构造器工作的时候，可能会引发协变和逆变的过程。举个例子，我们用+符号来表示协变：

`trait F[+A] // the "+" means "covariant"`

#### 协变

协变意味着如果B是A的子类型，类型F[B] 是类型 F[A] 的子类型。包括像List和Option这样的集合类型：

`trait List[+A]
trait Option[+A]`

Scala集合类的协变允许我们在我们的代码中用一个类型的集合替换另外一个类型的集合。举个例子，因为Circle是Shape的子类型，因此，我们可以将List[Circle]用在任何List[Shape]的地方：

`sealed trait Shape
case class Circle(radius: Double) extends Shape
val circles: List[Circle] = ???
val shapes: List[Shape] = circles`

那么，什么是逆变？我们协类型构造器的时候使用-符号，像这样：

`trait F[-A]`

#### 逆变

可能有点让人困惑，逆变的意思是，如果A是B的子类型，那么类型F[B]就是F[A]的子类型。这种情况下，对于表示处理过程的模型类型是有用的，比如上面提到的JsonWriter类型类:

`trait JsonWriter[-A] {
def write(value: A): Json
}
// defined trait JsonWriter`

让我们分析一下。记住型变的意思是一个值替代另外一个值的能力。考虑一种场景，我们有两个值，一个是类型Shape的值，一个是类型Circle的值，和两个JsonWriter对象，一个是为Shape准备的，另外一个为Circle准备：

`val shape: Shape = ???
val circle: Circle = ???
val shapeWriter: JsonWriter[Shape] = ???
val circleWriter: JsonWriter[Circle] = ???
def format[A](value: A, writer: JsonWriter[A]): Json = writer.write(value)`

现在，问自己一个问题，哪一个value和writer的组合可以传递给format？我们可以将circle与任何一个writer进行组合，因为所有的Circle都是Shape。反之，我们不能将shape与circleWriter组合在一起，因为并非所有的Shape都是Circle。

我们采用逆变来形成正式的模型来规范这种关系。因为Circle是Shape的子类型，因此JsonWriter[Shape]是 JsonWriter[Circle]的子类型。这就意味着我们可以将shapeWriter应用在任何JsonWriter[circle]的地方。

#### 不变

不变其实是最简单的情况。我们在类型构造器中不加+或者-符号：

`trait F[A]`

这就意味着，不论类型A和类型B是什么关系，类型F[A]和F[B]之间不存在子类型的关系。
这是Scala类型构造器的默认语法。当编译器搜索隐式转换的时候，它将会查找一个匹配的类型或者子类型。这样我们可能会通过采用型变标注来控制类型类实例选择的某些扩展。
在这里势必会引发两个问题。让我们先想象一下我们有一个像这样的代数类型：

`sealed trait A
final case object B extends A
final case object C extends `

问题是：

1. 如果有一个对象是可能的，是否被定义为超类的那个实例应该被选择？举个例子，我们能否定义A的一个实例，而让他适配类型B和C呢？
2. 应该选择一个子类型的实例还是更倾向于选择超类的实例？举个例子，如果我们定义了A和B的实例，并且我们有了B类型的一个值，那么这个实例应该是选择B呢还是更应该选择A？

其实，我们不能同时满足这两个条件。下面的表格给出了三种不同的行为：

| 类型类型变 | 不变 | 协变 | 逆变 |
| :----:   | :----: | :----: | :----: |
| 使用超类型实例？ | 不 | 不 | 是 |
| 更倾向于特定类？ | 不 | 是 | 不 |


世界上没有完美的系统。Cats更倾向于采用逆变类型类；这就允许我们为我们所需的子类型指定特定的实例。其实也就意味着只要我们需要，举个例子，Some[Int]这个类型的值，类型实例Option不能使用。但是可以采用类型标记像Some(1):Option[Int]这样来解决，或者使用像Option.apply,Option.empty,some或者我们在1.53节里面的none方法这种“聪明的构造器”来解决。
