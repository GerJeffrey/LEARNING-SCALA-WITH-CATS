
## 1.3 练习：可打印库

Scala提供了toString方法让我们将任意值转换为一个字符串。然而，这种方法引来了一些缺陷：它需要在预言中的每一种类型中实现这个方法，而这其中许多实现都会在实际应用中有限制，并且我们并没有针对特殊类型进行特化实现的选择。
为了解决这个问题，让我们定义个可打印类型类：
1. 定义个包含有format方法的类型类。format应该接受一个类型A的值并且返回一个字符串。
2. 创建一个包含有String和Int类型的Printable实例的PrintableInstances 对象。
2. 定义一个包含两个泛型接口方法的对象：
format接受一个类型A的值以及一个对应类型的Printable。这个方法使用对应的Printable来将A转化为字符串。
print接受的参数和format相同，但是返回值为Unit。这个方法用println将A转化的字符串打印到控制台。

#### 使用这个库

上面这种形式的通用目的打印库我们可以用在多种应用中。现在我们可以定一个应用来使用这个新库。
开始，我们将会定义个一个数据类型来代表众所周知的有毛动物：
`final case class Cat(name: String, age: Int, color: String)`
下一步，我们将会创建这个Cat的Printable实现以便于打印出如下形式的内容：
`NAME is a AGE year-old COLOR cat. `
最终，使用这个类型类在控制台或者一小段app中创建一个Cat对象，并将其打印到控制台：
`// Define a cat:
val cat = Cat(/* ... */) `

#### 更好的语法

让我们通过定义新的扩展方法提供更好的语法来是的这个打印库更加好用：
1. 创建一个叫做的PrintableSyntax对象；
2. 在对象Inside PrintableSyntax内部定义一个类PrintableOps[A]用来包裹一个类型A的值；
3. 在PrintableOps中定义下面方法：
    - format接受一个隐式Printable[A]并且返回表示包裹A的字符串；
    - print接受一个隐式Printable[A]并且返回Unit，其作用就是将A的字符串形式打印到控制台。
4. 使用这个扩展的方法来打印上述的练习；
