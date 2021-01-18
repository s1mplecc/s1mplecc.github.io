---
title: Clean Code -- 第二节：报表参数填充案例
date: 2018-05-07T13:39:03.000Z
tags: ['Code Refactoring']
categories: [Coding]
---
## 前言

该系列源于[张逸总监](http://zhangyi.xyz/)的 **Clean Code** 培训，包涵了如何写出高质量代码的思想、代码案例以及重构代码的实际演练。[上一篇](https://s2mple.xyz/2018/05/04/Clean%20Code%20--%20%E7%AC%AC%E4%B8%80%E8%8A%82%EF%BC%9A%E8%BF%AA%E7%B1%B3%E7%89%B9%E6%B3%95%E5%88%99%E4%B8%8E%20Paperboy%20%E6%A1%88%E4%BE%8B/)我们介绍了迪米特法则（最小知识法则）、信息专家模式：数据和行为应该封装在一起，并且通过 Paperboy 案例实操在 IntelliJ IDEA 中如何重构代码。本篇我们将通过报表参数填充案例来加深理解，并见识 IDEA 中更为强大的重构功能。

## 准备工作

- **IntelliJ IDEA**，相比 Eclipse，IDEA 在重构方面十分优秀，结合快捷键，使用起来让人赏心悦目。
- **示例代码**地址 https://github.com/agiledon/cleancode.git ，进行实际操作有助于加深理解和记住快捷键。代码有两个分支，master 分支为重构前的代码，**after-refactoring** 分支为重构后代码，可以使用快捷键 `Cmd + D` 查看**代码差异**。
- **活用快捷键**，本人使用的 IDEA 快捷键为 Mac OS X 10.5+ 。Windows 用户将 `Cmd` 替换成 `Ctrl`，或者在 IDEA 的 **Refactor** 菜单中查看快捷键。

| 快捷键 | 作用 |
|------|------|
|Ctrl + T | 重构菜单 |
|Shift + F6 | 重命名方法、属性、文件 |
|Cmd + Alt + M | 提取方法（extract method） |
|Cmd + Alt + N | 内联（Inline），与 extract 相反 |
|Cmd + Shift + 上下箭头 | 上下移动声明体（statement） |

## 案例：报表系统之参数处理

在示例项目中，需要对客户发出的 Web 请求进行处理，获得需要的参数。参数的值放在 Request 中，实现根据配置文件获得了参数的类型信息。根据项目需求，将参数划分为三种：

- 单一参数（SimpleParameter）
- 元素项参数（ItemParameter）
- 表参数（TableParameter）

因为参数的属性是在配置文件中已经配好，所以定义了 ParameterGraph 对象。它能够读取参数的配置信息，并根据参数的类型创建不同的参数类，这些参数类共同实现了 Parameter 接口。

最初的实现代码为：

```java
public class ParameterCollector {
    public void fillParameters(ServletHttpRequest request, ParameterGraph parameterGraph) {
        for (Parameter para : parameterGraph.getParmaeters()) {
            if (para instanceof SimpleParameter) {
                SimpleParameter simplePara = (SimpleParameter) para;
                String[] values = request.getParameterValues(simplePara.getName());
                simplePara.setValue(values);
            } else {
                if (para instanceof ItemParameter) {
                    ItemParameter itemPara = (ItemParameter) para;
                    for (Item item : itemPara.getItems()) {
                        String[] values = request.getParameterValues(item.getName());
                        item.setValues(values);
                    }
                } else {
                    TableParameter tablePara = (TableParameter) para;
                    String[] rows =
                            request.getParameterValues(tablePara.getRowName());
                    String[] columns =
                            request.getParameterValues(tablePara.getColumnName());
                    String[] dataCells =
                            request.getParameterValues(tablePara.getDataCellName());

                    int columnSize = columns.length;
                    for (int i = 0; i < rows.length; i++) {
                        for (int j = 0; j < columns.length; j++) {
                            TableParameterElement element = new TableParameterElement();
                            element.setRow(rows[i]);
                            element.setColumn(columns[j]);
                            element.setDataCell(dataCells[columnSize * i + j]);
                            tablePara.addElement(element);
                        }
                    }
                }
            }
        }
    }
}
```

观察代码，`if statement`中的三段代码块其实做的都是**填充参数**的事，只不过用`instanceof`判断了参数类型再执行对应的填充操作。结合之前的 Paperboy 案例，与 Wallet 打交道的应该是 Customer 而不是 Paperboy。上述代码明显也违反了迪米特法则，**参数填充的行为应该交由参数类完成而不是 ParameterCollector**。
    
## 重构

基于代码重用的思想，先将三段代码 **提取方法（Cmd + Alt + M）**，并分别取名为`fillSimplePara()`、`fillItemPara()`、`fillTablePara()`，重构后的方法顺序可能有所差异，可以在方法签名上使用 **Cmd + Shift + ⬆️/ ⬇️** 快捷键移动方法顺序，使之符合正常的阅读习惯。

![A6C5B629-ACC6-458A-8379-852021771E83](/0.png)

提取了方法后，这一幕是不是似曾相识？之前的 Paperboy 案例中，我们从`charge()`方法中提取了`pay()`方法，意识到`pay()`方法应该归属于 Customer 而不是 Paperboy。这里就要再次强调：**数据和行为应该封装在一起**。显然，ParameterCollector 这个类不应该关注如何填充参数，而是应该收到`fillParameters()`请求后**委派**给各个 Parameter 类完成填充操作。所以我们采取“**Move Method**”重构手法（**F6**）。

![17C89C7E-DA4F-4914-AA2A-5F6C8DE8845E](/1.png)

注意，IDEA 通过入参智能提示可以将方法移动到两个类中，显然应该选择 SimpleParameter，同时，方法的访问权限应为 public。接下来分别将`fillItemPara()`和`fillTablePara()`方法移动至 ItemParameter 和 TableParameter 中。

![8FB216EF-394B-42F3-B556-6EC576224FE7](/2.png)

观察上图你会发现，原先的方法有两个参数，但由于我们移动了方法到对应的 Parameter 类中，第二个参数直接变成了用`this`引用当前对象。`ItemParameter itemPara = this;`这一步是多余的，直接使用**内联（Cmd + Alt + N**）消除。然后呢，别着急，我们想一想，既然 SimpleParameter、ItemParameter、TableParameter 三个类都有接收一个 request 参数的填充方法，而且他们都实现了 Parameter 接口，那我们为什么不将方法提到接口中，让其他三个类去重写呢？所以接下来先把方法都重命名为`fill()`，使用 **Shift + F6**进行重命名。

![74F483BA-13C8-4769-9E94-290369E2B41B-1](/3.png)

接着，不要傻乎乎的去 Parameter 接口中手写`fill()`抽象方法，还是有快捷键的！那就是 “**Pull Members Up**”重构手法，说真的，当时看张总演示我就震惊了！选中方法，使用 **Ctrl + T** 调出重构菜单，输入一个 pull 你就能看到提示了。
![C477A0C4-7E53-4C85-BE02-4A37697680B5](/4.png)

IDEA 会提示你将`pull()`方法提升至 Parameter 接口中并 make abstract。
![9E0BC21E-0DFB-47AF-8B9E-C99900755E7F](/5.png)

我们看看重构后的效果，甚至于 ItemParameter 的`fill()`方法还已经帮你添加了`@Override`注解，但是剩下的两个 Parameter 类中的`fill()`方法还需要你手动的添加注解。

![0007121B-DCAA-4629-B1EB-911959D12455](/6.png) ![FE508CEB-BABF-4D6F-830E-A6C126BCD693](/7.png)

最后我们回到 ParameterCollector 类，使用 IDEA 的**智能辅助**快捷键（**Alt + Enter**）去 Cleanup code 删除多余的类型转换。

![5A6909D5-21DC-45BB-B3F0-02ECE946EC10](/8.png)

然后，你就会发现，`if statement`也是多余的，删除后，**Cmd + Alt + L** 格式化代码。

![91715C62-8438-4C16-BE77-0C821007877A](/9.png)

## 总结

如此，重构过的代码，可谓是层次清晰、一目了然，可比原来一坨堆在 ParameterCollector 中要美观多了。在重构的过程中，我们意识到，**填充参数的行为应该交由它的专家，即各个 Parameter 类管理**，移动方法后，又意识到既然都实现了 Parameter 接口，就应当使用 Java 接口的特性将`fill()`提升到 Parameter 接口中成为一个抽象方法。至此，`if`语句和`instanceof`也都不需要了，交由 JVM 在运行时识别类类型即可。所以说，**重构的过程还可以帮助程序员梳理编码过程，进行合理的抽象和封装**。当然，没有实践的空想理论是不行的，牢记 IDEA 的快捷键，将重构养成习惯，不时来一波亮瞎狗眼的骚操作，岂不美滋滋？