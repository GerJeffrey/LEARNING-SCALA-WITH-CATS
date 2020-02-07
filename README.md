# 前言

编写这部书有双重目的，第一是介绍monads，functors以及其它架构程序设计的函数式编程模式，并且解释在Cats中，这些概念是如何实现的。
Monads和相关概念与面向对象中设计模式的概念类似，都是在抽象分层组织代码构建抽象，与面向对象模式有两点不同：

* 1、这些概念是经过正式、准确的定义的；
* 2、这些概念具有极端的普遍性；

这种普遍性意味着可以通过应用monads这类函数式概念来解决那些难以被理解并且难以被抽象的广泛问题。
在这部书中，我们的目标就是通过不同的方式向你展示这些概念，帮助你构建这些概念如何正确工作的思考模型。我们已经扩展了学习的用例，一个小的图形化标记，许多小的例子，并且还有数学定义。我们希望您通过学习可以找到为你所用的内容。
好吧，那就让我们开始吧。


## 版本

本书采用的Scala版本是2.12.3，Cats的版本是1.0.0。下面是基于sbt的最小构建，包括相关依赖和设置：


`scalaVersion := "2.12.3" libraryDependencies +=
"org.typelevel" %% "cats-core" % "1.0.0"
scalacOptions ++= Seq( "-Xfatal-warnings", "-Ypartial-unification"
)
`

## 模版项目

方便起见，我们创建了一个Giter8模版来作为开始项目。为了复制这个模版，我们键入如下内容：

`$ sbt new underscoreio/cats-seed.g8
`
这条命令会产生一个以Cats作为依赖的沙箱项目。需要得到如何运行样例代码或者通过交互式Scala控制台，请查看生成的README.md文件。
Cats-seed模版以最小化存在。如果你需要功能齐备的内容，你们请签出Typelevel的 sbt-catalysts模版：

`$ sbt new typelevel/sbt-catalysts.g8
`
这条命令将产生一套依赖库和编译器插件以及连带单元测试的模版和培训文档。详情可以查看catalysts 和 sbt-catalysts 了解详情。
# LEARNING-SCALA-WITH-CATS
