---
title: Clean Code -- 第三节：PartDB 案例与模版方法模式
date: 2018-05-10T01:24:34.000Z
tags: ['Code Refactoring']
categories: [Coding]
---
## 前言

该系列源于[张逸总监](http://zhangyi.xyz/)的 **Clean Code** 培训，包涵了如何写出高质量代码的思想、代码案例以及重构代码的实际演练。[上一篇](https://s2mple.xyz/2018/05/07/Clean%20Code%20--%20%E6%8A%A5%E8%A1%A8%E5%8F%82%E6%95%B0%E5%A1%AB%E5%85%85%E6%A1%88%E4%BE%8B/)我们通过报表参数填充案例复习了**迪米特法则（最小知识法则）**、**信息专家模式：数据和行为应该封装在一起**。本篇我们将通过 PartDB 案例来见识 IDEA 中 **Extract Superclass** 等强大的重构功能，以及与案例相关的设计模式：**模版方法模式**。

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

## 案例代码

我们定义好了一个汽车零件类 Part，现在要通过 PartDB 去访问数据库执行`select * from part`，并将返回结果填充成一个 PartList。

```java
public class PartDB {
    private static final String DRIVER_CLASS = "";
    private static final String DB_URL = "";
    private static final String USER = "";
    private static final String PASSWORD = "";
    private static final String SQL_SELECT_PARTS = "select * from part";
    private List<Part> partList = new ArrayList<Part>();

    public void populate() throws Exception {
        Connection c = null;
        try {
            Class.forName(DRIVER_CLASS);
            c = DriverManager.getConnection(DB_URL, USER, PASSWORD);
            Statement stmt = c.createStatement();
            ResultSet rs = stmt.executeQuery(SQL_SELECT_PARTS);
            while (rs.next()) {
                Part p = new Part();
                p.setName(rs.getString("name"));
                p.setBrand(rs.getString("brand"));
                p.setRetailPrice(rs.getDouble("retail_price"));
                partList.add(p);
            }
        } finally {
            c.close();
        }
    }
}
```

上述代码大概是每一个 Java 程序员都会接触到的：使用 JDBC 访问数据库。加载驱动、创立连接、执行 SQL 返回结果集并且填充到 Java 类中。显然，如果每一步都需要我们手动编写，那若是新增了一个类，比如 CustomerDB 是不是还要去 getConnection 等等，这必然会造成许多**冗余代码**。而现在许多的持久层框架，例如 MyBatis、Hibernate 都是基于 JDBC 进行再封装，避免了重复繁琐的工作。下面我们看看如何重构上述代码。

## 重构

首先一段整洁易读的代码应该层次分明，如果没想到重构，我们可能会使用注释和换行将代码分段。

```java
// get connection
Class.forName(DRIVER_CLASS);
c = DriverManager.getConnection(DB_URL, USER, PASSWORD);

// get result set
Statement stmt = c.createStatement();
ResultSet rs = stmt.executeQuery(SQL_SELECT_PARTS);

// populate parts entity
while (rs.next()) {
    ...
}
```

然而，在学会重构之后，我们要习惯使用方法名产生注释的效果，**注释不是越多越好，而要少而精，不必要的注释就不要**。使用 **提取方法（Cmd + Alt + M）** 重构代码，并使用 **Cmd + Shift + ⬆️/ ⬇️** 移动方法顺序。

![25C7F4CD-951E-4B05-8A98-5A0433520EDA](/0.png)

注意我这里将`c`重命名（**Shift + F6**）为`connection`，**命名要遵循在表达清楚业务含义的同时尽可能的短**，`rs`、`err`甚至`e`这种因为频繁使用，基本上可以望文生义，所以可采用业界统一认可的缩写。但`c`这种命名，一旦代码量增多后，就可能带来阅读上的困扰，应当避免。顺便说一下，应该使用`populateParts()`而不是`populatePartList()`这种命名，因为**命名应该站在功能层面，不应该暴露技术实现**，万一哪一天，你改成 Set 了呢？

我们来观察提取出的三个方法。`getConnection()`创建数据库连接，在抽象层面与业务无关的，实现层面我们暂不考虑用户自定义情况（实际上应该由配置文件读入），所以实现层面也与业务无关。`getResultSet()`执行 SQL 获取结果集，在抽象层面与业务无关。实现层面上由于`SQL_SELECT_PARTS`所以与业务强耦合。我们可以使用 **Cmd + Alt + N** ，将`select * from part` SQL 语句**内联**，再提取`getSql()`方法（**Cmd + Alt + M**）。

![953ECD52-3430-4BFD-A662-24E78BCAF755](/1.png)

如此一来，`getResultSet()`在实现层面也与业务无关了。而`getSql()`方法在抽象层面与业务无关，在实现层面就与业务有关了。`populateParts()`这个名字容易让人产生误解，我们将其重命名为`populateEntities()`，这样，它在抽象层面也与业务无关，而在实现层面与业务有关。为什么我一再强调**抽象层面**和**实现层面**与**业务**的关系，就是为了让方法变得纯粹。这也是软件设计中一种重要的设计思维：**关注点分离**，目的是将**解决特定领域问题的代码从业务逻辑中独立出来，业务逻辑的代码中不再含有针对特定领域问题代码的调用，业务逻辑同特定领域问题的关系通过侧面来封装、维护，这样原本分散在在整个应用程序中的变动就可以很好的管理起来**。比方有一个函数叫做`CreateNewCustomer()`，那么该函数只与客户的数据（属性）打交道，而给新客户自动发优惠券的动作就不能放到这个函数里面。

现在，我们所有的方法都在抽象层面与业务无关，只有`getSql()`和`populateEntities()`在实现层面与业务有关。至此，我们拨开重重迷雾，冰山的一角轰然显现！既然大家都在抽象层面与业务无关，我们就应该**抽象出更高阶的抽象类**，这里就用到了 **Extract Superclass** 这个神奇的重构手法。

使用 **Ctrl + T** 调出重构菜单，输入 super 就可以看到 Extract Superclass 的提示，敲击回车
![F0675E41-8516-4FC3-96BB-928AB1060D40](/2.png)注意需要勾选组成父类的成员，这里除了最后一个`partList`是业务独有的，其他全勾选上，而`getSql()`和`populateEntities()`方法因为具体实现与业务相关，就应该是子类去重写，所以应当勾选上 Make abstract。

**重构后的 JdbcTemplate 类**：

![68B462DA-CB1F-41FD-A519-6576CF7BD2FE](/3.png)

**重构后的 PartDB 类**：

![CC64100D-FC25-4825-85DD-83AECC318E86](/4.png)

为什么我在提取父类时直接将父类命名为 JdbcTemplate，大家应该不难想到，我们重构完成的抽象父类，就是一个最简化的 JdbcTemplate。许多框架亦是如此诞生的，使用者使用时让自己的业务实体类去继承这个模版类，重写业务相关方法。这也是软件设计模式中非常著名的**模版方法模式**。

## 模版方法模式

模板方法模式说白了就是：**定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤**。代表具体逻辑步骤的方法称做**基本方法(primitive method)**；而将基本方法汇总穿成一条线的方法叫做**模板方法(template method)**，这个设计模式的名字也是由此而来。 

在上述的抽象类 JdbcTemplate 中，我们的模版方法就应该是`populate()`方法

```java
public void populate() throws Exception {
    Connection connection = null;
    try {
        connection = getConnection();
        ResultSet rs = getResultSet(connection);
        populateEntities(rs);
    } finally {
        connection.close();
    }
}
```

类似于写信的模版，类似“尊敬的xxx”、“此致，敬礼”这种千篇一律的套话我们可以定义一个模版，而需要差异化书写的只是信的内容。在`populate()`中，骨架已经搭好：先创建数据库连接，再执行 SQL 返回结果集，最后填充实体类。所以这里能看出，**模版方法展现了框架的生命周期**。为防止恶意操作，一般模板方法都会加上`final`关键字，不允许用户重写模版方法。

基本方法又可以分为三种：抽象方法、具体方法和钩子方法：

- **抽象方法(Abstract Method)**：一个抽象方法由抽象类声明，由具体子类实现。在 Java 语言里抽象方法以`abstract`关键字标识。
- **具体方法(Concrete Method)**：一个具体方法由抽象类声明并实现，而子类并不实现或置换。可以添加`final`关键字做强约束不可重写，比如`getResultSet()`
- **钩子方法(Hook Method)**：一个钩子方法由抽象类声明并实现，而子类会加以扩展。通常抽象类给出的实现是一个**空实现**，作为方法的默认实现。譬如有些框架的`init()`方法，用户可以实现也可以不实现，定义成`abstract`就不合适，所以框架会给个空方法

## 总结

我在工作当中也使用过 MongoTemplate、RedisTemplate 诸如此类，还从未仔细想过这些类为什么这样命名。直到这次张总通过 IDEA 强大的 **Extract Superclass** 重构手法，给我们展示一个 JdbcTemplate 如何浮出水面的过程，实在是醍醐灌顶，豁然开朗。有兴趣的同学可以看看这些框架的源代码，加深对模版方法模式的理解。

## 参考

- [模版方法模式](http://www.runoob.com/design-pattern/template-pattern.html)
- [《JAVA与模式》之模板方法模式](http://www.cnblogs.com/java-my-life/archive/2012/05/14/2495235.html)
- [Separation of concerns - Wiki](https://en.wikipedia.org/wiki/Separation_of_concerns#Internet_protocol_stack)
- [架构漫谈系列之关注点分离](http://www.cnblogs.com/asis/p/architecture-Soc.html)