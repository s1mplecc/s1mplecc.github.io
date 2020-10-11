---
title: Clean Code -- 迪米特法则与 Paperboy 案例
date: 2018-05-04T05:36:05.000Z
tags: ['Clean Code', 'Refactoring']
categories: [Coding]
---
## 前言

该系列源于[张逸总监](http://zhangyi.xyz/)的 **Clean Code** 培训，包涵了如何写出高质量代码的思想、代码案例以及重构代码的实际演练。本篇为该系列的第一篇，主要介绍**迪米特法则**、**信息专家模式**，并结合案例实操在 **IntelliJ IDEA** 中如何重构代码。本文大部分内容转载自张总博客: [迪米特法则与重构](http://zhangyi.xyz/demeter-law-and-refactoring/#more)。

## 准备工作

- **IntelliJ IDEA**，相比 Eclipse，IDEA 在重构方面十分优秀，结合快捷键，使用起来让人赏心悦目。
- **示例代码**地址 https://github.com/agiledon/cleancode.git ，进行实际操作有助于加深理解和记住快捷键。代码有两个分支，master 分支为重构前的代码，**after-refactoring** 分支为重构后代码，可以使用快捷键 `Cmd + D` 查看**代码差异**。
- **活用快捷键**，本人使用的 IDEA 快捷键为 Mac OS X 10.5+ 。Windows 用户将 `Cmd` 替换成 `Ctrl`，或者在 IDEA 的 **Refactor** 菜单中查看快捷键。
```
Shift + F6 // 重命名方法、属性、文件
Cmd + Alt + M // 提取方法（extract method）
Cmd + Alt + N // 内联（Inline），与 extract 相反
Cmd + Shift + ⬆️/ ⬇️ // 上下移动声明体（statement）  

Ctrl + T // 重构菜单
```

## 迪米特法则

在面向对象设计的世界里，有一个寻常却又常常为人所忽略的原则——**迪米特法则（Law of Demeter）**。这个原则认为，任何一个对象或者方法，它应该只能调用下列对象：    

- 该对象本身
- 作为参数传进来的对象（也可以是该对象的字段）
- 在方法内创建的对象

这个原则用以指导正确的**对象协作**，分清楚哪些对象应该产生协作，哪些对象则对于该对象而言，又应该是无知的。

## 代码案例及分析

如何理解这个原则？我们可以看看 David Bock 就该原则给出的一个[绝佳案例](https://www2.ccs.neu.edu/research/demeter/demeter-method/LawOfDemeter/paper-boy/demeter.pdf)。假设一个超市购物的场景，顾客（Customer）到收银台结账，收银员（Paperboy）负责收钱。我们来看看代码：

```java
public class Customer {
    private String firstName;
    private String lastName;
    private Wallet myWallet;
    public String getFirstName(){
        return firstName;
    }
    public String getLastName(){
        return lastName;
    }
    public Wallet getWallet(){
        return myWallet;
    }
}

public class Wallet {
    private float value;
    public float getTotalMoney() {
        return value;
    }
    public void setTotalMoney(float newValue) {
        value = newValue;
    }
    public void addMoney(float deposit) {
        value += deposit;
    }
    public void subtractMoney(float debit) {
        value -= debit;
    }
}

public class Paperboy {
    public void charge(Customer myCustomer, float payment) {
        Wallet theWallet = myCustomer.getWallet();
        if (theWallet.getTotalMoney() > payment) {
            theWallet.subtractMoney(payment);
        } else {
            //money not enough
        }
    }
}
```
    
让我们将 Paperboy 中`charge()`方法的代码翻译成这幕小话剧的对白。

```txt
“把钱包交出来!”收银员算出顾客要买的商品总价后，“温柔”地对顾客说道。
顾客言听计从，赶紧将钱包掏出来，恭恭敬敬地递给收银员。
接过钱包，收银员毫不客气地打开，检查里面的钱够不够。噢，不错，钱够了。收银员从钱包取出钱，心满意足地笑了。
```
如果你是顾客，你敢去这样的超市 shopping 吗？

对于 Paperboy 而言，Wallet 不满足迪米特法则三个条件中的任何一个，因此，**不应该让 Paperboy 与 Wallet 对象进行直接交互**。若从**拟人化**的角度思考，则 Wallet 其实属于 Customer 的**隐私**。如此重要的隐私，怎么能直接交给收银员这个陌生人呢？这里所谓的“隐私”，可以视为是“数据”，是“信息”，是“知识”，因此我们往往又将迪米特法则称之为“**最小知识法则**”。

当我们理解“最小知识法则”时，又可以从**职责**的角度去思考以上代码。对于收银员角色，他的职责应该是负责收钱，而不用去管钱包里的钱够不够，如果够了怎么办，如果不够又该怎么办，这些统统都不属于他的职责。设想一下，当超市里人流如织，大家都在购买商品时，如果每一个收银员都要承担这般的职责时，会出现什么样的景象？所以“最小知识法则”乃善法，在对象社区中，我们就应该刻意减少对象之间彼此深入的了解。了解最小的知识，就意味着依赖最小，彼此产生的影响就会最小。这实际上是 **KISS（keep it simple and stupid）** 原则的体现。

**信息专家模式**告诉我们：“信息的持有者即为操作该信息的专家”。对于对象，所谓**信息就是该对象内部的字段**。在上述的例子中，Wallet 是 Customer 的字段，那么操作 Wallet 的行为自然就应该分配给 Customer 了。这是题中应有之义。“信息专家模式”其实是面向对象最重要原则“**数据与行为应该封装在一起**”的别名。若在领域建模时能遵循该原则，则可以规避我们设计出贫血模型。

## 重构

如何修改以上代码？注意，`charge()`行为仍然属于 Paperboy 的职责，因此我们不应该将该方法整体搬迁到 Customer 中，而应该先进行**方法的提取**，选中代码块，使用 **Cmd + Alt + M** 快捷键提取方法。这块代码执行的是支付的步骤，所以命名为`pay()`。

![781D2706-3961-4C77-ACF8-8BDDCE21B17E](/0.png)

提取后的`charge()`在接收方法请求后，转而将请求**委派**给了`pay()`方法。我们可以这样理解：在抽象层面，收款是收银员的职责；在实现层面，是`pay()`方法支持了收款行为，该实现归属于顾客。

![1AF2B42F-7B13-4284-AF03-9C1D5C5FE0A1](/1.png)

观察`pay()`方法，我们发现该方法操作的数据皆来自 Customer。我们嗅到了一种坏味道，即 Martin Fowler 所谓的“**特性依恋（Feature Envy）**”。对于该坏味道，老马是这样阐释的：“函数对某个类的兴趣高过对自己所处类的兴趣”。不要再嫉妒了，桥归桥，路归路，让方法回到自己最喜欢的地方吧。运用“**Move Method**”重构手法，将`pay()`方法移动到 Customer 中。

光标在方法签名或方法体中都可，按下 **F6**，IDEA 会根据方法的入参，智能的提示将方法移动到哪个类中，这里只能是 Customer。还要注意方法的访问权限，默认是 Escalate 往上提升一级，这里选择 Public。

![2C6EA6D7-A330-4E90-8B2A-54223072A47A](/2.png)

操作之后，`pay()`方法移动到 Customer 中。

![F92C14F0-6693-4154-8A6D-F71AB14BF0E2](/3.png)

在将方法移到正确的位置后，我们发现暴露的`getWallet()`方法根本就没有意义。更何况，将钱包裸露出去，难道是想要炫富吗？还是低调一点为好，隐藏自己的“隐私”，总好过被人觊觎而招来飞来横祸之险。于是，**内联（inline）** 之。将光标放在`getWallet()`上，按下 **Cmd + Alt + N**。

![563C9626-A71E-4DC7-A8DC-BA4DA530EF64](/4.png)

`getWallet()`方法被删除，并被替换为 Customer 的字段`myWallet`。那现在声明的`theWallet`就是多余的，继续内联。
![28E463A5-F36A-4250-8611-36BDACF80FCE](/5.png)

最后，重构过的代码应该是这样的。阅读代码一目了然，Paperboy 进行收银，并委派给顾客进行实际的支付，即掏钱包，数钱，付钱。
![3CA41F80-0556-42F3-8E16-5AEEC39332A3](/6.png)![E66E7919-9391-4046-8CB4-C46555E561A8](/7.png)

## 总结

判断一段代码是否违背了迪米特法则，有一个小窍门，就是看调用代码是否出现形如`a.m1().m2().m3().m4()`之类的代码。这种代码在 Martin Fowler《重构》一书中，被名为“**消息链条(Message Chain)**”，有人更加夸张地名其为“火车残骸”。车祸现场啊，真是惨不忍睹。

实际上未重构前的代码即是一种消息链条，在`charge()`方法中实际执行的是`myCustomer.getWallet().subtractMoney(payment)`，先获取 Customer 的 Wallet，再执行 Wallet 的扣钱方法。

那么，如下代码是否这样的残骸呢？
    
```java
str.split("&")
    .stream()
    .map(str -> str.contains(elementName) ? str.replace(elementName + "=", "") : "")
    .filter(str -> !str.isEmpty())
    .reduce("", (a, b) -> a + "," + b);
```

答案是：不是。这样的代码我们一般称之为“流畅接口或连贯接口（Fluent Interface）”。二者的区别在于**观察形成链条的每个方法返回的是别的对象，还是对象自身。如果返回的是别的对象，就是消息链条**。所谓`m1().m2().m3().m4()`的调用，其实是调用者不需要也不想知道的“知识”，把这些中间过程的细节暴露出来没有意义，调用者关心的是最终结果；而上述代码中的`map()`与`filter()`等方法其实返回的还是 Stream 类。这一调用方式其初衷并非告知中间过程的细节，而是一种声明式的 DSL 表达，调用者可以自由地组合它们。

## 参考

- [迪米特法则与重构 -- 张逸](http://zhangyi.xyz/demeter-law-and-refactoring/#more)
- [The Paperboy, The Wallet, and The Law Of Demeter --  David Bock](https://www2.ccs.neu.edu/research/demeter/demeter-method/LawOfDemeter/paper-boy/demeter.pdf)