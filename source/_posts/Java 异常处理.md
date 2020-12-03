---
title: Java 异常处理
date: 2018-07-16T05:21:06.000Z
tags: [Java]
categories: [Languages]
---
## 前言

一直以来，Java 的异常处理可能是每个 Java 程序员都需要面对和经历的难题之一。何时该抛出异常，该抛出何种异常，需不需要自定义异常类，何时又该捕获异常并处理，相信每一个 Java 程序员或多或少都有这样的疑问。最近也是觉得项目需要统一规范的异常处理，所以花了些时间研读 *Thinking in Java*、《阿里巴巴 Java 开发手册》以及前辈们总结的优质文章，归纳了一些自认为比较关键的部分，希望有助于大家更好的设计如何处理异常。关键还是需要大量优秀源码的阅读经验。

> 异常的设计初衷是将**运行时产生的错误信息通过某种方式传递给某个接收者 —— 该接收者将知道如何正确的处理这个问题**。Java 使用异常来提供一致的错误报告模型，使得构件能够与客户端代码可靠地沟通问题。实际上，**异常处理的一个重要目标就是把错误处理的代码同错误发生的地点分离**，使得你在某处专注于要完成的事，而在另一处处理错误。既分离了主干代码和错误处理逻辑，又可以重用错误处理代码。—— *Thinking in Java*

上面所说的接受者其实分为两类，一类是开发人员，另一类是使用用户。所以，实际上异常设计出来：

1. 帮助开发人员定位错误，修复代码漏洞。
2. 反馈给客户端用户，比如表单的输入值非法，让他得以更正错误。

## 异常分类

Java 将异常分为两个大类：**非检查型异常（Unchecked Exception）**，也称非受控异常。以及**检查型异常（Checked Exception）**，也称受控异常。这两者的区别在于：
1. Check Exception 编译器会做强制检查，必须使用`try...catch`捕获或者`throws`在方法签名处申明，否则编译不通过。
2. Unchecked Exception 发生在运行期，具有不确定性，通常是由于程序的**逻辑问题**所引起的，所以在程序设计中我们需要考虑周全，尽量通过**提前预检**避免这类异常。

Java 中异常的继承结构如下图所示：

![v2-2cb1558f17876f329804fcced62661ef_1200x500](0.jpg)

### Error

Error 表示**系统级别的错误**，应用程序本身无法克服和恢复的一种严重问题，例如 JVM 栈溢出`StackOverFlowError`、内存溢出`OutOfMemoryError`等。**在编码中不应该主动抛出 Error 类型的异常**。

### 运行时异常

运行时异常`RuntimeException`是`Exception`衍生子类中的非检查型异常，即编译器不会强制要求捕获或者声明抛出。`RuntimeException`发生的时候，表示程序中出现了编程错误（即 Bug），所以应该找出错误修改程序，而不是去捕获。比如空指针异常`NullPointerException`、数组下标越界`ArrayIndexOutOfBoundsException`，程序员可以通过提前预检去避免该类错误。

### 受控异常

绝大多数受控异常（Checked Exception）位于`java.io`包内，这是合乎情理的，因为在你请求了不存在的系统资源的时候，一段健壮的程序必须能够优雅的处理这种情况，而不应该挂掉。比如`FileNotFoundException`可能是用户删除了文件，或者`ConnectException`网络中断导致的连接异常，这类问题可以通过用户恢复文件或检查网络及时矫正。

## 异常处理的最佳实践

异常堆栈信息提供了导致异常出现的方法调用链的精确顺序，包括每个方法调用的类名，方法名，代码文件名以及行数，以此来精确定位异常出现的现场。在有效使用异常的情况下，异常类型回答了**什么**异常被抛出，异常堆栈跟踪回答了**在哪**抛出，异常信息回答了**为什么**会抛出，如果你的异常没有回答以上全部问题，那么可能你没有很好地使用它们。本文主要介绍异常使用的两大原则：**提早抛出**和**延迟捕获**。更多请参考博客最后的参考文章，推荐研读《阿里巴巴 Java 开发手册》。

### 提早抛出

来看以下代码

```java
private static void readFile(String filename) throws IOException {

    InputStream in = new FileInputStream(filename);

    // read file...
}

readFile(null); // throw exception

/*
Exception in thread "main" java.lang.NullPointerException
at java.io.FileInputStream.<init>(FileInputStream.java:130)
at java.io.FileInputStream.<init>(FileInputStream.java:93)
at info.s1mple.exceptiondemo.ExceptionDemo.readFile(ExceptionDemo.java:15)
at info.s1mple.exceptiondemo.ExceptionDemo.main(ExceptionDemo.java:10) */
```

以上代码展示了`FileInputStream`类的构造函数传入`null`值导致的`NullPointerException`异常。不幸的是，`NullPointerException`是 Java 中信息量最少的（却也是最常遭遇且让人崩溃的）异常。它压根不提我们最关心的事情：到底哪里是`null`。所以我们不得不回退几步去找哪里出了错。
    
通过跟踪堆栈打印信息，我们可以确定错误原因是向`readFile()`传入了一个空文件名参数。既然`readFile()`不能处理空文件名，所以马上检查该条件，如果文件名为空则抛出非法参数异常。

```java
private static void readFile(String filename) throws IOException {
    if (filename == null){
        throw new IllegalArgumentException("filename is null");
    }

    InputStream in = new FileInputStream(filename);

    // read file...
}

readFile(null); // throw exception

/*
Exception in thread "main" java.lang.IllegalArgumentException: filename is null
    at info.s1mple.exceptiondemo.ExceptionDemo.readFile(ExceptionDemo.java:15)
    at info.s1mple.exceptiondemo.ExceptionDemo.main(ExceptionDemo.java:10) */
```

这里使用`IllegalArgumentException`继承自`RuntimeException`的非受控异常，所以不强制要求在方法签名处声明。实际开发中根据业务要求，假如文件名是由用户输入，那可以自定义类似`RequestArgsIllegalException`的受控异常类将错误信息返回给前端并呈现给用户。在命名自定义异常类时要清晰准确。

类似上述代码中的`if statement`被称为**卫语句（guard clauses）**，原则是将可能出错的每个分支做**单独检查**，要么抛出异常要么立即返回，将正常的实现代码放在卫语句之后保证运行到此处时所有条件都已通过。如此既避免了嵌套层次过多的`if...else`条件分支，又使得实现代码被剥离出来放在最后集中处理。
    
通过提早抛出异常（又称**迅速失败**），异常得以清晰又准确。堆栈信息立即反映出什么出了错（提供了非法参数值），为什么出错（文件名不能为空值），以及哪里出的错。另外，其中包含的异常信息`"filename is null"`使得异常提供的信息更加丰富，而这是我们之前代码中抛出的`NullPointerException`所无法提供的。

### 延迟捕获

编写异常的程序员最可能犯的一个错误是，在程序有能力处理异常之前就捕获它。问题在于，捕获之后该拿异常怎么办？最不该的就是什么都不做。空的`catch`块等于把整个异常丢进黑洞，能够说明何时何处为何出错的所有信息都会丢失。把异常写到日志中还稍微好点，至少有迹可循。但我们总不能指望用户去阅读或者理解日志文件和异常信息。阿里开发手册中说道：**捕获异常是为了处理它，不要捕获了却什么都不处理而抛弃之，如果不想处理它，请将该异常抛给它的调用者。最外层的业务使用者，必须处理异常，将其转化为用户可以理解的内容**。

在有条件处理异常之前过早捕获它，通常会导致更严重的错误和其他异常。例如，如果上文的`readFile()`方法在调用`FileInputStream`构造方法时立即捕获和记录可能抛出的`FileNotFoundException`，代码会变成下面这样：

```java
private static void readFile(String filename) throws IOException {
    InputStream in = null;

    try {
        in = new FileInputStream(filename);
    } catch (FileNotFoundException e) {
        logger.error("file not found: {}", filename);
    }

    in.read(); // may produce NullPointerException
}

readFile("xxx"); // throw exception

/*
13:10:27.439 [main] ERROR info.s1mple.exceptiondemo.ExceptionDemo - file not found: xxx
Exception in thread "main" java.lang.NullPointerException
    at info.s1mple.exceptiondemo.ExceptionDemo.readFile(ExceptionDemo.java:32)
    at info.s1mple.exceptiondemo.ExceptionDemo.main(ExceptionDemo.java:16) */
```
    
上面的代码在完全没有能力从`FileNotFoundException`中恢复过来的情况下就捕获了它。如果文件无法找到，那么`in.read()`就不应该执行。尽管我们捕获了`FileNotFoundException`异常并打印了日志，但是一个更让人头疼的`NullPointerException`将被抛出。错误信息不仅误导了我们出了什么错（真正的错误是文件找不到而不是空指针），还误导了错误的出处，不应该在`in.read()`而应该在`new FileInputStream(filename)`。

所以`readFile()`真正应该做的事情不是捕获这些异常，那应该是什么？看起来有点有悖常理，通常最合适的做法其实是什么都不做，不要马上捕获异常。把责任交给`readFile()`的调用者，让它来决定文件缺失的处理方法，有可能会提示用户指定其他文件，或者使用默认值，实在不行的话警告用户并退出程序。

当然，最终你的程序需要捕获异常，否则会意外终止。但这里的技巧是**在合适的层面捕获异常**，以便你的程序要么可以从异常中有意义地恢复并继续下去，而不导致更深入的错误；要么能够为用户提供明确的信息，引导他们修正错误。如果当前所在方法无法胜任，那么就不要处理异常，而将它抛出直至具备处理这个异常的能力的作用域来处理。

## 该抛出何种异常

那么抛出异常时该抛出何种异常，是 Checked Exception 还是 Unchecked Exception ？我在网上查阅资料时发现不同的人对此有不同的观点，不管是开发人员还是项目的 Leader。有的认为该全部抛出 Unchecked Exception 类型的异常，并在某一层面做统一捕获处理，这样做的优点是保证了方法的纯粹性。另一种观点则不支持抛出`RuntimeException`。通过阅读《Thinking in Java》和[关于异常的争论：要检查，还是不要检查？-- IBM developerWorks](https://www.ibm.com/developerworks/cn/java/j-jtp05254/)（这是一篇非常优质的文章）发现很多资深的专家甚至是语言的设计者对此持有不同观点。
 
### 传统的观点

> “If a client can reasonably be expected to recover from an exception, make it a checked exception. If a client cannot do anything to recover from the exception, make it an unchecked exception.” —— *The Java Tutorial*

Sun 公司的 *The Java Tutorial* 摘录中，总结了关于将一个异常声明为检查型还是非检查型的传统观点。

因为 Java 语言并不要求方法捕获或者指定运行时异常，因此编写只抛出运行时异常的代码或者使得他们的所有异常子类都继承自`RuntimeException`，对于程序员来说是有吸引力的。这些编程捷径允许程序员编写 Java 代码不会受到来自编译器的所有挑剔性错误的干扰。尽管对于程序员来说这似乎比较方便，但是它回避了 Java 的捕获或者指定要求的意图，并且对于那些使用您提供的类的程序员可能会导致问题。 

如果您仅仅是因此而抛出一个`RuntimeException`或者它的子类，那么您换取到了什么呢？您只是获得了抛出一个异常而不用您指定这样做的能力。换句话说，这是一种用于避免文档化方法所能抛出的异常的方式。在什么时候这是有益的？也就是说，在什么时候避免注明一个方法的行为是有益的？答案是“几乎从不”。

**换句话说，Sun 告诉我们检查型异常应该是准则。** 该教程通过多种方式继续说明，通常应该抛出检查型异常，而不是`RuntimeException` —— 除非您是 JVM。

在 *Effective Java: Programming Language Guide* 一书中，Josh Bloch 提供了下列关于检查型和非检查型异常的知识点，这些与 *The Java Tutorial* 中的建议相一致（但是并不完全严格一致）：

- **第 39 条：只为异常条件使用异常。** 也就是说，不要为控制流程而使用异常，比如应当使用`Iterator.hasNext()`去判断迭代边界，而不是在调用`Iterator.next()`捕获`NoSuchElementException`。
- **第 40 条：为可恢复的条件使用检查型异常，为编程错误使用运行时异常。** 这里，Bloch 回应传统的 Sun 观点 —— 运行时异常应该只是用于指示编程错误，例如违反前置条件。
- **第 41 条：避免不必要的使用检查型异常。** 换句话说，对于调用者不可能从其中恢复的情形，或者惟一可以预见的响应将是程序退出，则不要使用检查型异常。
- **第 43 条：抛出与抽象相适应的异常。** 换句话说，一个方法所抛出的异常应该在一个抽象层次上定义，该抽象层次与该方法做什么相一致，而不一定与方法的底层实现细节相一致。例如，一个从文件、数据库装载资源的方法在不能找到资源时，应该抛出某种`ResourceNotFound`异常（通常使用异常链来保存隐含的原因），而不是更底层的`IOException`、`SQLException`。

### 重新考察检查型异常

最近，几位受尊敬的专家，包括 *Thinking in Java* 作者 Bruce Eckel 和 *J2EE Design and Development* 作者 Rod Johnson，已经公开声明尽管他们最初完全同意检查型异常的正统观点，但是他们已经认定**排他性**使用检查型异常的想法并没有最初看起来那样好，并且**对于许多大型项目，检查型异常已经成为一个重要的问题来源**。Eckel 提出了一个更为极端的观点，建议所有的异常应该是非检查型的；Johnson 的观点要保守一些，但是仍然暗示传统的优先选择检查型异常是过分的。Martin Fowler 也说过：“总的来说，我觉得异常很不错，但是 Java 的‘被检查的异常’带来的麻烦比好处要多。”

不恰当的使用检查型异常会导致许多问题：

- **检查型异常不适当地暴露实现细节**。如果选择不处理而将异常逐层向上抛出，势必会导致方法与底层实现所抛异常相耦合。举个例子，一个`loadUserProfile()`方法可能会去调用加载数据库或者加载文件系统的代码，而在该方法层面抛出`SQLException`或是`FileNotFoundException`，这实际上违反了 Bloch 的 第 43 条 —— 被抛出的异常所位于的抽象层次与抛出它们的方法不一致。所以你可能需要重新包装一个`NoSuchUserException`并抛出。
- **不稳定的方法签名**。你可能对因为实现的改变而修改方法签名声明的异常感到厌烦，事实上这也是由于业务方法与具体实现相耦合所导致的。本质上还是没遵循第 43 条，方法在遇到失败时应该抛出一个异常，并且该异常应该反映该方法做什么（就像`NoSuchUserException`反应出该方法是去查询用户），而不是它具体如何做（像`SQLException`反应出需要连接数据库）。
- **难以理解的代码**。因为许多方法都抛出一定数目的不同异常，错误处理的代码相对于实际的功能代码的比率可能会偏高，使得难以找到一个方法中实际完成功能的代码。异常通过集中错误处理来设想减少代码的，但是一个具有三行代码和六个 catch 块（其中每个块只是记录异常或者包装并重新抛出异常）的方法看起来比较膨胀并且会使得本来简单的代码变得模糊。这与异常设计的初衷相违背。

基于以上种种原因，Bruce Eckel 声称在使用 Java 语言多年后，他已经得出这样的结论，认为检查型异常是一个失败的发明。Eckel 提倡将所有的异常都作为非检查型的。如果查看 Eckel 的 Web 站点上的讨论，您将会发现回应者是严重分裂的。一些人认为他的提议是荒谬的；一些人认为这是一个重要的思想。

Rod Johnson 采取一个不太激进的方法。他提出，**一些异常本质上是次要的返回代码**（它通常指示违反业务规则），该类异常的作用通常在于中断违反业务规则的操作，抛出至处理错误的层面被捕获（指示如何修正）。Johnson 提倡对于该类异常使用检查型异常。

### 该使用何种异常

关于是否使用检查型异常的决定是复杂的，并且很显然没有明显的答案。Sun 的建议是对于任何情况使用它们，而 C# 方法（也就是 Eckel 和其他人所赞同的）是对于任何情况都不使用它们。其他人表示可以折中考虑。个人认为并没有绝对的对与错，**选择何种异常在于团队的统一规范**。非检查型异常的最大风险之一就是它并没有按照检查型异常采用的方式那样自我**文档化**。阅读 Java 源码发现，不管是检查型异常还是非检查型异常，都在文档中标明了该方法会抛出何种异常。在决定使用非检查型异常更需如此，不管是提示自己还是使用你提供的类库的其他人员。

## 参考

- 《Thinking in Java》
- [阿里巴巴 Java 开发手册](https://dir.s1mple.online/alibaba-java-dev-manual.pdf)
- [关于异常的争论：要检查，还是不要检查？-- IBM developerWorks](https://www.ibm.com/developerworks/cn/java/j-jtp05254/)
- [Java异常的深入研究与分析 -- 知乎](https://zhuanlan.zhihu.com/p/28302269)
- [如何优雅的处理异常？-- 知乎](https://www.zhihu.com/question/28254987)
- [《Effective Java》中关于异常处理的几条建议](https://www.cnblogs.com/skywang12345/p/3544287.html)
