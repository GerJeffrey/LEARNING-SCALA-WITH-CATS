## 4.5 另外一面:错误处理和MonadError

Cats提供了附加的类型类MonadError用来抽象类似Either的数据类型来处理错误。MonadError提供了引发和错误处理的额外操作。

>这部分是可选的

>你除非你需要在错误处理的monad之上进行抽象，你无需使用MonadError。举个例子，你可以使用MonadError来对Future和Try进行抽象，或者对Either和EitherT进行抽象(我们将会在第5章遇到)。

>如果你不需要这种抽象，可以直接跳到4.6节。

###4.5.1 MonadError类型类

这是MonadError的简单定义版本:

`package cats
trait MonadError[F[_], E] extends Monad[F]{
  def raiseError[A](e: E): F[A]
  def handleError[A](fa: F[A])(f: E => A):F[A]
  def ensure[A](fa: F[A])(e: E)(f: A => Boolean): F[A]
  }`

MonadError 定义了两个类型参数:

+ F是monad的类型；
+ E是F内包含的错误的类型

为了演示这些参数如何匹配，这里有一个例子实例化Either的类型类:

`import cats.MonadError
import cats.instances.either._

type ErrorOr[A] = Either[String ,A]
val monadError = MonadError[ErrorOr, String]`

>ApplicativeError

>实际上，MonadError扩展自称之为ApplicativeError的类型类。然而我们在第6章之前不会遇到Applicative。语法实际上与我们之前遇到的都比较类似，所以我们现在直接忽略到细节即可。

###4.5.2 引发和处理错误

MonadError有两个最重要的方法，分别是raiseError和handleError。raiseError类似于Monad的pure函数，其创建一个代表失败的实例：

`val success = monadError.pure(42)

val failure = monadError.raiseError("Badness")`

handleError是raiseError的补充。允许我们消费一个错误，并且可能将其转换为成功，类似于Future的恢复方:

`monadError.handleError(failure){
  case "Badness" => monadError.pure("It is ok")
  case other => monadError.raiseError("It's not ok")
  }`

还有第三个有用的方法叫做ensure，实现了类似于filter的行为。我们通过断言来预测成功的monad的值，并且如果断言返回错误，那么就引发指定的一个错误：

`import cats.syntax.either._
monadError.ensure(success)("Number too low")(_ > 1000)`

cats通过cats.syntax.applicativeError提供了raiseError和handlerError的语法，ensure通过cats.syntax.monadError来提供:

`import cats.syntax.applicative._
import cats.syntax.applicativeError._
import cats.syntax.monadError._

val success = 42.pure[ErrorOr]

val failure = "Badness".raiseError[ErrorOr, Int]

success.ensure("Number to low")(_ > 1000)`

还有这些方法的其他有用变体。可以查看cats.MonadError和cats.ApplicativeError获取更多信息。

### 4.5.3 MonadError的实例

cats提供了包括Either，Future，Try等多种数据类型的MonadError的实例。Either的实例可以对任何错误类型进行定制，而FUture和Try的实例却经常表示可抛出的错误：

`import scala.util.Try
import cats.instances.try._

val exn: Throwable = new RuntimeException("It' all gone wrong")

exn.raiseError[Try, Int]`

4.5.4 练习：抽象

































#
