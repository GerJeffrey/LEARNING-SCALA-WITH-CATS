
## 4.4 Either

让我们学习一下另外一个有用的monad：Scala标准库中的Either类。在Scala2.11中以及更早版本，许多人不认为Either是一个monad，因为它没有map和flatMap方法。在Scala2.12版本中，Either变成了right biased(右偏差)。

###4.4.1左偏差和右偏差

在Scala2.11中，Either没有默认的map或者flatMap方法。这就使得2.11版本中的Either在for comprehension中用起来不方便。我不得不在每一个生成器语句中插入调用.right：

`val either1: Either[String, Int] = Right(10)
val either2: Either[String, Int] =
Right(32)

for{
  a <- either1.right
  a <- either2.right
  } yield a + b`

在Scala2.12中，Either被重新设计了。现代的Either确定right表示成功的结果并且直接支持map和flatMap调用。这是的for comprehension用起来更加令人陶醉：

`for{
  a <- either1
  b <- either2
  } yield a + b`

Cats通过cats.syntax.either导入来兼容Scala2.11，允许我们在所有的Scala中使用右偏置的版本。在scala2.12中，我们即可以忽略这个导入，也可以直接使用：

`import cats.syntax.either._

for{
  a <- either1
  b <- either2
  } yield a + b`

### 4.4.2 创建实例

另外，直接创建Left和Right的实例，我们也可以从cats.syntax.either导入asRigh和asLeft扩展方法：

`import cats.syntax.either._

val a = 3.asRight[String]
val b = 4.asRight[String]

for{
  x <- a
  y <- b
  } yield x*x + y*y`

这种"只能构造器"具备Left.apply和Right.apply优势，因为无论是Left还是Right他们总是返回Either类型。这就会帮助我们避免因为类似于下面例子中过窄的问题而导致的类型引用错误：

`def countPositive(nums: List[Int]) =
  nums.foldLeft(Right(0)){(accumulator, num) =>
    if(num > 0 ){
      accumulator.map(_ + 1)
    }else{
      Left("Negative. Stopping")
    }
    }`

这段代码由于两个原因而编译出错失败：

1. 编译器引用的accumulator的类型是Right来替代Either；
2. 我们不能对Right.apply指定类型参数，因此编译器将左参数直接作为Nothing来引用。

切换为asRight就可以避免这些问题。asRight有一个Eigher类型的返回值，并且允许我们使用一个类型参数来指定类型：

`def countPositive(nums: List[Int]) =
  nums.foldLeft(0.asRight[String]){(accumulator, numb) => if(num > 0){
    accumulator.map(_ + 1)
    }else{
      Left("Negative. Stopping")
  }}

  countPositive(List(1,2,3))
  countPositive(List(1,-2,3))
`
cats.syntax.either为Either伴生对象增加了一些有用的扩展方法。catchOnly和catchNonFatal方法捕获Either的实例上的异常是非常有用的：

`Either.catchOnly[NumberFormatException]("foo".toInt)

Either.catchNonFatal(sys.error("Badness"))`
还有从其他数据创建Either的方法：

`Either.fromTry(scala.util.Try("foo".toInt))

Either.fromOption[String, Int](None, "Badness")`

### 4.4.3转换Either

cats.syntax.either也给Either实例增了一下有用的方法。我们可以使用orElse和getOrElse从right或者返回的默认值中抽取值：

`import cats.syntax.either._

"Error".asLeft[Int].getOrElse(0)
"Error".asLeft[Int].orElse(2.asRight[String])`

ensure方法允许我们检查左值是否满足预设：

`-1.asRight[String].ensure("Must be non-negative!")( _ > 0)`

recover和recoverWith方法提供了处理Future的名称缺失问题的类似方法：

`"error".asLeft[Int].recover{
  case str: String => -1
  }
"error".asLeft[Int].recoverWith{
  case str: String => Right(-1)
}
  `

还有leftMap和bimap方法实现map：

`"foo".asLeft[Int].leftMap(_.reverse)

6.asRight[String].bimap(_.reverse, _ * 7)

"bar".asLeft[Int].bimap(_.reverse, _ * 7)`

swap方法可以让我们将左值和右值交换：

`123.asRight[String]
123.asRight[String.swap]`

最后，cats增加了一些便携方法:toOption, toList, toTry, toValidated等等。

### 4.4.4错误处理

Either最典型的应用是实现快速失败处理。我们在进行序列计算的时候经常食用flatMap。如果其中一个计算失败，那么剩余的计算就不会再运行：

`for{
 a <- 1.asRight[String]
 b <- 0.asRight[String]
 c <- if(b == 0) "DIV0".asLeft[Int] else (a / b).asRight[String]

  }yield c * 100`

当使用Either作为错误处理的时候，我们需要确定我们想要用来表示错误的类型是什么。我们可以使用Throwable来做这件事情：

`type Result[A] = Either[Throwable, A]`

这给我来提供的语法与scala.util.Try有点类似。然而问题是，Throwable是一个极端宽泛的类型。我们几乎并不知道错误发生的类型。

另外一个方法就是定义代数数据类型来表示发生在程序中的错误：

`sealed trait LoginError extends Product with Serializable

final case class UserNotFound(username: String) extends LoginError

final case class PasswordIncorrent(username:String) extends LoginError

case object UnexpendtedError extends LoginError

case class User(Username: String, password: String)

type LoginResult = Either[LoginError, User]`

这种方法解决了我们在Throwable中看到的问题。其给我们了一组固定集合的错误类型，并且能够在catch-all中对我们不能预知的错误做处理。我们也获得了任何穷经检查的任何模式匹配：

`def handleError(error: LoginError): Unit = error match{

  case UserNotFound(u) => println(s"User not found: $u")
  case PasswordIncorrent(u) => println(s"Password incorrent: $u")
  case UnexpectedError =>
    println(s"Unexpended error")
  }
val result1: LoginResult = User("dev", "passw0rd").asRight

val result2: LoginResult = UserNotFound("dave").asLeft

val result1.fold(handleError, println)
val result2.fold(handleError, println)`

### 4.4.5 练习什么是最好的？

在前面的错误处理策略例子中是否符合更所有的目的？我们是否想从错误处理得到另外的特性？

#### D.3 什么是最好的？

这是一个开放性问题。也是一种具有技巧的问题，其答案依赖于我们正在查找的语法。我们琢磨出这些点：

+ 在处理大型工作时错误恢复非常重要。我们不希望在需要运行一天的工作最终发现在最后的元素中却出错了。

+ 错误报告也有同等的重要性。我们需要知道什么错误，而不是仅仅是某些事情错了。

+ 在很多情况下，我们想要收集所有的错误，而不是我们遇到的第一个。一个典型的例子就是验证网页表单。当提交表单的时候，给用户提供所有的错误是一种更好的体验。
























#
