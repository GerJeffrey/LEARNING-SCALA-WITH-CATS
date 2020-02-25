# 第4章 Monads

在Scala中，monad是最常见的抽象之一。许多Scala程序员都可以快速直觉的熟悉monad，尽管往往他们并不知道这就是monad。

非正式情况下，一个monad就是任何具备一个构造器和flatMap方法的东西。我们在上一章节看到的所有functor都是monad，包括Option，List和Future。我们甚至有专门支持monad的语法：为了更加容易理解。
然而，尽管这是无处不在的概念，Scala标准库却缺少包含"可以flatmapped的事物的"实体类。通过Cats库，我们可以获得这个便利。

本节内容我们回深入monad。我们将通过一些例子来导入他们。我们将学习到Cats中他们的正式定义和实现。最后，我们将看到你从没有看到的有趣的monad的推荐和他们如何使用的例子。










































#
  `
