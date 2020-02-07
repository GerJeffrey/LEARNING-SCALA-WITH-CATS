
## 1.7 小结

这一章节中，我们对类型类有了粗浅的认识。并且用scala原生的方式自己实现了Printable类型类，之后，举了Cats的Show和Eq两个例子。
现在，我们可以发现Cats类型类的通用模式：

+ cats包里面的类型类实际上就是通用的trait；
+ 每一个类型类都有一个伴生对象，这个半生对象有一个apply方法用来实现实例，有一个或者多个构造器方法用来创建实例，以及一组其他相关联的帮助方法。
+ cats.instances提供了默认的实例，其组织方式通过类型参数来构造而非类型类。
+ cats.syntax提供了很多类型类的语法。

第一部分剩下的部分，我们将会看到更多类型和强大的类型类，比如 **半群(Semigroup)**，**幺半群(Monoid)**，**函子(Functor)**，**单子(Monad)**，**Semigroupal**,**Applicative**，**Traverse** 以及更多类类型。每一种类类型，我们都会学习到它们所具有的功能性以及正式的规则，并且这些内容是如何在cats中实现的。许多这些类类型都比Show或者Eq更具有抽象，尽管会让这些概念更加难以理解和学习，但是它们远比Show或者Eq解决通用问题更加有用。