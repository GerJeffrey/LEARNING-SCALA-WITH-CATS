
## 4.6 Eval Monad

catsEval是允许我们抽象不同模型的推断的monad。我们听说过这两种典型的模型：eager和lazy。Eval对于结果是否可被缓存有很大的区别。

### 4.6.1 Eager，Lazy和缓存

这些都是什么意思？

Eager立即计算，而lazy是在访问的时候才计算。缓存第一次访问的结果被缓存时计算。
举个例子， Scala的val是eager和缓存的。我们可以用一个明确的副作用计算来看到这一点。在下面的例子中，代码中执行计算x的值发生在其被定义的当场，而不是访问的时候才计算。访问x的时候直接调用保存的值，而不是重新运行这段代码：

`val x = {
  println("Computing X")
  math.random
  }
  x
  x`

为了对比，def是lazy并且并非缓存的。代码只有在我们访问它的时候(lazy)才运行，并且每次访问都会重新运行(非缓存);

`def y = {
  println("Computing Y")
  math.reandom
  }
y
y
`

至少，lazy vals是lazy和缓存的。z之下的代码知道我们访问的时候才会执行，并且只是第一次执行(lazy)。其计算结果会被缓存，并且在后续的时候可以访问(缓存):

`lazy val z={
  println("Computing Z")
  math.random
  }
z
z`

### 4.6.2 评估的Eval模型

Eval有三种子类型:Now, Later, 和Always。我们通过三种构造器方法来构建他们。这三种方法会传教这三种类的实例并返回Eval的类型：

`import cats.Eval

val now = Eval.now(math.random + 1000)
val later = Eval.later(math.random + 2000)

val always = Eval.always(math.random + 3000)
`
我们可以通过Eval的value方法解析结果：

`now.value
later.value
always.value`

Eval的每一种类型都会使用上述的一个评估模型来计算其结果。Eval.now会立即获取结果。其语义类似于val的eager和memoized：

`val x = Eval.now{
  println("Computing x")
  math.random
  }
x.value
x.value`

Eval.always是lay计算，类似于def:

`val y = Eval.always{
  println("Computing")
  math.random
  }

y.value
y.value`

Eval.later是lazy和memoized计算，类似于lazy val:

`val z = Eval.later{
  println("computing z")
  math.random
  }

z.value
v.value`

三种行为模式总结如下:

| Scala | Cats | Propertis |
| :----  | :---- | :---- |
| val | Now | eager, memoized |
| lazy val | Later | lazy, memoized |
| def | Always | lazy, not memoized |

### 作为Monad的Eval

和所有monad一样，Eval的map和flatMap方法将计算增加到链上。在这种情况，这个链被明确的作为方法的列表保存。方法在我们调用Eval的value方法的时候运行获取一个结果：

`val greeting = Eval
  .always {println("Step1"); "Hello"}
  .map{str => println("Step2"); s"$sr world")}

greeting.value`

请注意，
























#
