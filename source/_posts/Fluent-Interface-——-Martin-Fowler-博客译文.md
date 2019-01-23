---
title: Fluent Interface —— Martin Fowler 博客译文
tags: [DSL, API Design]
categories: [Translations]
date: 2019-01-23 14:15:45
---
## 译文

几个月前我和 Eric Evans 进行了一次讨论会，他谈到一种接口的设计风格，我们称之为流畅接口（Fluent Interface）。它不是一种常见的风格，但我们觉得应该广为人知。描述它的最直观的方式就是例子。

最简单的例子可能就来自 Eric 编写的 [TimeAndMoney Library](http://timeandmoney.sourceforge.net/)。为了指定一段时间间隔，我们通常这么做：
    
```java
TimePoint fiveOClock, sixOClock;
...
TimeInterval meetingTime = new TimeInterval(fiveOClock, sixOClock);
```

但是 TimeAndMoney 库的使用者会这样做：

```java
TimeInterval meetingTime = fiveOClock.until(sixOClock);
```

下面我继续演示“客户下订单”这个例子。一个订单包含多个订单项，每个订单项包含商品项和购买的数量。在提交订单时一个订单项应当是可跳过的，这意味着我更希望在没有此订单项（缺货）的情况下提交订单，而不是推迟提交整个订单。所以这里可以给整个订单一个“急促（rush）”的状态标识。

要实现上述功能，最常见的编码如下：

```java
private void makeNormal(Customer customer) {
    Order o1 = new Order();
    customer.addOrder(o1);
    OrderLine line1 = new OrderLine(6, Product.find("TAL"));
    o1.addLine(line1);
    OrderLine line2 = new OrderLine(5, Product.find("HPK"));
    o1.addLine(line2);
    OrderLine line3 = new OrderLine(3, Product.find("LGV"));
    o1.addLine(line3);
    line2.setSkippable(true);
    o1.setRush(true);
}
```

本质上我们创建了多个对象然后将它们组装在一起。如果无法在构造函数中设置所有内容，那么就需要创建临时变量来帮助我们完成组装 —— 尤其是将集合项添加到集合中。

下面是使用流畅接口实现相同的组装：

```java
private void makeFluent(Customer customer) {
    customer.newOrder()
            .with(6, "TAL")
            .with(5, "HPK").skippable()
            .with(3, "LGV")
            .priorityRush();
}
```

关于这种风格最重要的一点就是，基于 Internal DomainSpecificLanguage 将要做的事沿着一条线进行编码（译者注：Domain Specific Language，DSL，领域专用语言）。实际上这也是为什么我们选择用 “Fluent” 一词来描述它，在很多方面这两个术语是同义词。**这种 API 被设计为可读的和流式的，这种流畅性的代价是在设计和构建 API 时需要花更多的功夫**。构造函数、setter 和 add 方法的 API 简单且容易编写，但要想提供一个漂亮的流畅接口则需要更多的思考。

事实上，刚才我想用在 Calgary 咖啡店吃早餐的时间完成这个小例子的编码，但是我搞砸了，看来好的流畅接口需要花费一些时间去实现。如果你想找一个比较成熟的例子，可以看看 [JMock](http://jmock.org/)。与任何 mocking 库一样，JMock 需要创建复杂的行为规范。在过去几年中已经构建了许多 mocking 库，而 JMock 的这个则包含了非常漂亮的流畅接口，使用体验非常好。这是它的一个例子：

```java
mock.expects(once())
    .method("m")
    .with(or(stringContains("hello"), stringContains("howdy")));
```

我看到 Steve Freeman 和 Nat Price 在 [JAOO2005](https://www.martinfowler.com/bliki/JAOO2005.html) 上就 JMock API 的演变发表了精彩的演讲，演讲相关的内容他们已经发表到一篇 [OOPSLA论文](http://www.mockobjects.com/files/evolving_an_edsl.ooplsa2006.pdf) 上。

到目前为止，我们看到用于创建对象配置的流畅接口通常会涉及到 [值对象](https://en.wikipedia.org/wiki/Value_object)。我不确定这是否属于流畅接口的一个定义特征，虽然我怀疑它们出现在声明性上下文中有某种关联。对我们而言，**流畅性的关键考验在于领域特定语言的质量**。API 使用起来越像流式的语言，它就越流畅。

像这样构建一个流畅接口会导致一些不符合使用习惯的 API。其中最明显的一个就是 setter 会有返回值（在订单示例中，`with` 方法为订单添加一个订单项并返回整个订单），而惯例是修改性质的方法返回 `void`，因为这样遵循 [CommandQuerySeparation](https://martinfowler.com/bliki/CommandQuerySeparation.html) 原则（译者注：CQS，命令查询分离原则）。这个约定确实妨碍了流畅接口，所以我倾向于暂不遵循这个惯例。

你应该**根据流畅接口的下一个行为（fluent action）去选择返回类型**。JMock 提出了一个重点：根据接下来的需要改变其返回类型。**这种风格的一个很好的优点是方法补完后（intellisense）有助于告诉你接下来要键入什么 —— 有点像 IDE 中的智能提示**。总的来说，我发现动态语言对于 DSL 来说效果更好，因为它们的语法往往更简洁。但是，使用方法补完是静态语言的一个优点。

流畅接口定义的方法的一个问题是它们可能名不符实。举个例子，你去查看 `with` 方法的文档可能并没有什么意义，因为这个方法的实现和 `with` 并没有什么关联。我承认光就方法的命名来说这不是一个好的命名，因为它根本不能表达该方法实际做了什么。只有在流畅行为的上下文中这种命名才能显示出它的优势（译者注：这点我在编码时也深有体会，流畅接口的方法实现与方法命名常常做的是两回事，比如说把值对象传递下去）。解决此问题的一种可能的方法是只在此上下文中使用 builder 对象（译者注：可以参考 Builder Pattern，比如 `new BankAccount.Builder(4567L).withOwner("Homer").atBranch("Springfield").build();`，只在最后一步 `build()` 中进行构建）。

Eric 提到的一点是，到目前为止，他使用并看到了流畅的接口大多是关于值对象的配置。值对象不具有领域意义的标识（Identity），因此你可以轻松创建并丢弃它们。所以接口的流畅度取决于使用旧值构造新值。从这个意义上讲，订单案例并不典型，因为它属于 [EvansClassification](https://www.martinfowler.com/bliki/EvansClassification.html) 中的实体对象（Entity）。（译者注：Evans Classification，Evans 关于领域对象的分类 Entity、Value Object 和 Service）

目前为止我还没有看到很多的流畅接口，可以得出结论，我们对它们的优缺点了解还不够。所以任何使用它们的劝告都只能是初步的 —— 但我认为它们已经成熟，可以进行更多的尝试。

[Piers Cawley](https://bofh.org.uk/2005/12/21/fluent-interfaces/) 对本文有一个很好的跟进。

**更新**（2008年6月23日）。自从我写这篇文章以来，这个术语被广泛使用，这给了我一种令人愉快的满足感。在我一直在研究的书中，我已经提炼了关于流畅接口和内部 DSLs 的想法。我也注意到了一个常见的误解 —— 很多人似乎将流畅接口与方法链（Method Chaining）等同起来。当然链式接口是使用了流畅接口的一种常用的技术，但真正的流畅接口远不止于此。

我上面展示的 JMock 示例使用了方法链，但同时也使用嵌套函数和对象作用域。

## 原文链接

[Fluent Interface](https://www.martinfowler.com/bliki/FluentInterface.html)，Matrin Fowler 博客，发表于 2005 年 12 月 20 日。