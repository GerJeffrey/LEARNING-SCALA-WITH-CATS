
## 2.6 Monoid的应用

我们知道了 **monoid** 其实是加和或者组合概念的抽象，但是它应该应用在什么地方呢？在一些巨大影响力的思想方面起到了主导作用。这些内容将陆续在本书的后续章节中逐步揭示出来。

### 2.6.1 大数据

在Spark和Hadoop这样的大数据应用中，我们将数据分析分布于许多机器上进行运算，并提供容错性和扩展性。这就意味着每一个机器都会返回结果数据的一部分，并且我们必须将这些结果组合起来形成最终的结果。在绝大多数情况下，这可以被认为是一个 **monoid** 。
如果我们想要计算一个web站点已经接受了多少个访问者，这就意味着对每个数据部分做为一个Int来计算。我们知道这其实就是Int的 **monoid** 实例的加和，这才是对部分结果进行组合的正确方法。
如果我们想要找出一个网站已经接受了多少个独立访问者，这就相当于在每一个部分数据上建立一个Set[User]。我们知道Set的union **monoid** 实例是对部分结果组合为最终结果的正确的方法。
如果我们想要计算我们服务器日志的99%和95%的应答时间，我们以使用一种叫做QTree的数据结构，也有相应的 **monoid** 与之对应。
一旦我们我抱有殷切的希望。大部分我们想要对大数据的分析几乎都是一个 **monoid** ，因此，我们可以以此来构建一个表达能力超强，功能强大的分析系统。这其实就是Twitter的 **Algebird** 和 **Summingbird** 项目所做的事情。 我们可以在mapreduce案例研究中深入探索。

#### 2.6.2 分布系统

在一个分布式系统里，不同的机器都以运算出不同数据视图结束。举个例子，一台机器可能会收到其他机器没有收到的更新。我们可能需要协调这些不同的视图，因此每台机器在没有更多更新到达的时候需要使得他们保持一直的数据。这种情况被称之为最终一致性。
有一种特殊的数据类型类支持这种协调。这些数据类型被称之为交换复制数据类型(CRDTs)。其关键的操作在于可以将两个实例中的所有信息捕获并合并起来。这种操作依赖于具备 **monoid** 实例。我们将会在CRDT实例学习时深入探索这种思路。

### 2.6.3 Monoids in the Small

上面的两个例子 **monoid** 影响到了整个系统架构。还有更多的情况是 **monoid** 使得代码段更加的简化。我们将会在本书中学习到很多这种例子。





































#
