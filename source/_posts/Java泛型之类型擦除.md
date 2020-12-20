---
title: Java 泛型之类型擦除
date: 2018-02-27T01:47:58.000Z
tags: ['Java']
categories: [Languages]
---
## Preface

> Java SE5 的重大变化之一就是加入了**泛型**，实现了参数化类型，使得代码可以适用于多种类型。但由于兼容旧版本的考虑，不得不作出了一些牺牲，比如**擦除**，使得泛型使用受到一定局限。本文简要介绍了 Java 泛型以及类型擦除的概念和使用
 
## 泛型

> **Java泛型** (**Generics**) 是 JDK5 引入的新特性，允许在定义类和接口的时候使用**类型参数** (**Type Parameter**)。声明的类型参数在使用时用具体的类型来替换。泛型最主要的应用是在 JDK5 中的新**集合类**框架中。

- 通过泛型，大大提升了集合类的**可复用性**，你可以定义一个装字符串的 ArrayList，也可以定义一个装整型的 ArrayList

    ```
    ArrayList<String> als = new ArrayList<>();
    als.add("aaa");

    ArrayList<Integer> ali = new ArrayList<>();
    ali.add(1); 
    ali.add("aaa");  // error
    ```
    
- 自定义泛型类可以参考我的上一篇博客，[Java泛型之LinkedStack](https://s2mple.xyz/java-generics-linkedstack/)

### 使用泛型需要注意
---

> 泛型的引入可以解决之前的集合类框架在使用过程中经常出现的运行时类型错误，因为编译器可以在编译时就发现很多明显的错误。但无奈的是，为了保证与旧版本的**兼容性**，Java 泛型的实现上存在着一些不够优雅的地方。

- 开发人员在使用泛型的时候，很容易根据自己的直觉而犯一些错误

    ```
    public void testGenerics(ArrayList<Object> list) {
        // ...
    }
    ```
    
    这个方法接收`ArrayList<Object>`作为形参，但是如果尝试传入一个`ArrayList<String>`对象，却发现无法通过编译
    
    ```
    public void test(){
        testGenerics(new ArrayList<String>()); // 编译错误!!!
    }
    ```
    
    虽然从直觉上来说，Object 是 String 的父类，这种类型转换应该是合理的。但是实际上这会产生**隐含的类型转换**问题，因此编译器直接就禁止这样的行为

## 类型擦除
 
> 使用泛型的时候加上的类型参数，会被编译器在**编译时**去掉(擦除)。这个过程就称为**类型擦除** (**Type Erasure**) 。如在代码中定义的`List<Object>`和`List<String>`等类型，在编译之后都会变成 List 。

> 可以说，类型擦除这个令人困惑的 Java 特性，完全是历史遗留问题导致的。因为 JDK5 之前没有泛型，为了旧代码能够正常运行，Java 泛型必须要在编译器层次将类型擦除，所导致的严酷事实就是：**在泛型代码内部，无法获得任何有关泛型参数类型的信息。**

很多泛型的特性都与类型擦除的存在有关，包括：

- **泛型类并没有自己独有的 Class 类对象。** 比如并不存在`List<String>.class`或是`List<Integer>.class`，而只有`List.class`

- **静态变量是被泛型类的所有实例所共享的。** 对于声明为`MyClass<T>`的类，访问其中的静态变量的方法仍然是`MyClass.myStaticVar`。不管是通过`new MyClass<String>`还是`new MyClass<Integer>`创建的对象，都是共享一个静态变量

- **泛型的参数类型不能用于 Java异常处理的`catch`语句中。** 因为异常处理是由 JVM 在运行时刻来进行的。由于类型信息被擦除，JVM 是无法区分两个异常类型`MyException<String>`和`MyException<Integer>`的。对于 JVM 来说，它们都是`MyException`类型的。也就无法执行与异常对应的`catch`语句

> 简单来说就是：**泛型的类型信息在编译阶段被擦除，运行时刻任何获取泛型参数类型信息的行为都是非法的。** 

### 编译器的类型检查
---

> 了解了类型擦除机制之后，就会明白**编译器承担了全部的类型检查工作**。编译器禁止某些泛型的使用方式，正是为了确保类型的安全性。
 
- 继续用上例代码来做演示

    ```
    public void testGenerics(ArrayList<Object> list) {
        list.add(1); // 合法
        list.add("aaa"); // 合法
    }

    public void test(){
        testGenerics(new ArrayList<String>()); // 编译错误!!!
    }
    ```
    不难理解，为了避免`ArrayList<String>`中添加一个 Integer 类型的对象，编译器干脆禁止了这样的行为。编译器会尽可能的检查可能存在的类型安全问题。对于确定是违反相关原则的地方，会给出编译错误。当编译器无法判断类型的使用是否正确的时候，会给出警告信息。
    
### 通配符与上下界
---

> 在使用泛型类的时候，既可以指定一个具体的类型，如`List<String>`就声明了具体的类型是 String，也可以用 **通配符`?`** 来表示未知类型，比如`List<?>`。通配符所代表的其实是**一组类型**，但具体的类型**暂时未知**，而是在**使用时再确定**。

- `List<?>`不同于`List<Object>`的地方是，`List<Object>`实际上确定了参数类型是 Object 及其子类。而`List<?>`则其中所包含的元素类型是不确定。使用时可以用`List<String>`或`List<Integer>`替换。如果它包含了 String 的话，往里面添加 Integer 类型的元素就是错误的

- 请看如下代码
 
    ```
    public void testGenerics(ArrayList<?> list) {
        list.add("aaa"); // 编译错误!!!
        
        Object obj = list.get(0); // 合法
    }

    public void test(){
        ArrayList<String> list = new ArrayList<>();
        list.add("aaa");
        testGenerics(list); // 合法
    }
    ```
    上述代码有如下几点要注意：
    1.  因为通配符`?`类型未知，所以为了避免运行时类型错误，`list.add("aaa")`编译不会通过
    2.  虽然类型未知，但肯定是 Object 及其子类，所以**通配符代表的元素可以用 Object 来引用。**
    3.  向`testGenerics()`方法传入的是`ArrayList<String>`类型的，并且编译通过。即使用时声明具体类型
    
> 由于`List<?>`中的元素只能用 Object 来引用，显然在需要更精细的类型时并不适用。在这些情况下，可以使用**上下界**来限制未知类型的范围。

- `List<? extends Number>`说明 List 中可能包含的元素类型是 Number 及其**子类**。

- `List<? super Number>`则说明 List 中包含的是 Number 及其**父类**。

- 当使用`extends`引入了上界之后，在使用类型的时候就可以**使用上界类中定义的方法**。查看如下的例子

    ```
    public static void testGenerics(ArrayList<? extends Number> list) {
        Number number = list.get(0);
        System.out.println(number.intValue());
    }

    public static void main(String[] args) {
        ArrayList<Double> doubles = new ArrayList<>();
        doubles.add(2.33333);
        testGenerics(doubles); // 运行输出结果：2 
    }
    ```
    形参改为`<? extends Number>`后，可以看到如下两个性质：
    1. 可以使用 Number 引用
    2. 在`testGenerics()`方法中可以调用 Number 类的`intValue()`方法

> 上述代码中，我们为了调用`intValue()`方法，必须给定泛型类的边界，以告诉编译器只能接受遵循这个边界的类型。这里重用了 **extends** 关键字。这样在编译时， **泛型类型参数将擦除到它的第一个边界**，即 Number，这样就可以调用`intValue()`方法了。

### 类型系统
---

> 引入泛型之后的类型系统增加了**两个维度**：一个是**类型参数自身**的继承体系结构，另外一个是**泛型类或接口自身**的继承体系结构。第一个指的是对于`List<String>`和`List<Object>`这样的情况，类型参数 String 继承自 Object 。而第二种指的是`List<String>`和`Collection<String>`这种情况。对于这个类型系统，有如下的一些规则：

- 相同类型参数的泛型类的关系取决于泛型类自身的继承体系结构。即`List<String>`是`Collection<String>`的子类型，`List<String>`可以替换`Collection<String>`。这种情况也适用于带有上下界的类型声明。

- 当泛型类的类型声明中使用了通配符的时候， 其子类型可以在两个维度上分别展开。如对`Collection<? extends Number>`来说，其子类型可以在 Collection 这个维度上展开，即`List<? extends Number>`和`Set<? extends Number>`等；也可以在 Number 这个层次上展开，即`Collection<Double>`和`Collection<Integer>`等。如此循环下去，`ArrayList<Long>`和`HashSet<Double>`等也都算是`Collection<? extends Number>`的子类型。

- 如果泛型类中包含多个类型参数，则对于每个类型参数分别应用上面的规则。

## Conclusion

> Java泛型确实有些让人困惑的特性，让它使用起来不如 C++ 这种最开始就囊括了泛型的语言舒服。这也是任何有历史的编程语言所需要承担的历史包袱，后续的版本更新会被早期的设计缺陷所拖累。正是这样，如果一门语言长久的停留在过去，必然会被新的语言取代。
 
## References

- 《Java 编程思想》
- [Java深度历险——Java泛型
](http://www.infoq.com/cn/articles/cf-java-generics)
