---
title: Java 泛型
date: 2018-02-28T06:01:53.000Z
tags: ['Java']
categories: [Languages]
---
## 前言

我的上一篇博客 [Java泛型与擦除](https://s1mple.online/java-generic-type-erasure/)只是对于 Java 泛型和类型擦除做了一个简要的介绍，本文主要是针对 Java 泛型做一些更深入的补充说明。

## 泛型的引入

在没有泛型之前，我们如果想向一个 Box 类中装入一个不知道类型的对象，只能使用 **Object** 这个顶级父类来操作盒子中的对象。

```java
public class Box {
    private Object object;

    public void set(Object object) { this.object = object; }
    public Object get() { return object; }
}
```

而引入了泛型之后，代码修改如下：

```java
/**
 * Generic version of the Box class.
 * @param <T> the type of the value being boxed
 */
public class Box<T> {
    // T stands for "Type"
    private T t;

    public void set(T t) { this.t = t; }
    public T get() { return t; }
}
```

我们定义了一个装有 T 类型的 Box 类。在使用时可以随心所欲，既可创建一个专门装 String 的 Box ，也可创建一个装 Integer 的 Box。

```java
Box<String> bs = new Box<String>();
Box<Integer> bi = new Box<Integer>();
```

甚至于我们在定义 Box 时可以更加苛刻，只允许这个盒子装入一个数字类型的对象。

```java
public class Box<T extends Number> {
    // ...
}
```

那么，如果你想在创建一个装 String 的 Box 就得报错了。

```java
// Error: Type Parameter 'String' is not within its bound; should extend 'Number'
// Box<String> bs = new Box<String>();
```

**Tips**：在 Java SE 7之后，允许我们使用如下更简洁的语法 `Box<Integer> integerBox = new Box<>();`，因为编译器能够推断出上下文的类型参数，所以没必要做冗余的声明。

## 泛型方法

泛型方法是声明了自己的类型参数的方法。泛型方法的类型参数的**作用域**仅限于声明它的方法内部。允许静态和非静态的泛型方法，即可以用 **static** 修饰。泛型方法的语法是**在方法的返回类型之前包含一个尖括号列出的类型参数列表**。

示范代码如下：

```java
public class TestGenericsMethod {

    private <T> void printClassName(T x) {
        System.out.println(x.getClass().getSimpleName());
    }

    public static void main(String[] args) {
        TestGenericsMethod test = new TestGenericsMethod();
        test.printClassName(" ");
        test.printClassName(10);
        test.printClassName('a');
        test.printClassName(test);
    }

}

// result:
// String
// Integer
// Character
// TestGenerics
```

## 泛型的类型变量

**类型变量**( or **Type Variable**) ,即上述代码的 T，用于代表一类类型，声明之后可以在 class 中使用。

### 命名惯例

按照惯例，类型参数使用**单个大写字母**表示，虽然这个字母命名并不强求，比如上述代码中的`<T>`，你可以使用`<U>`等等代替，但是如下是**类型参数的命名惯例**：

|命名简写|含义|
|:--:|--------------|
| T | Type |
| S,U,V etc. | 2nd, 3rd, 4th types |
| E | Element (被广泛适用于 Java 的集合类中，表示集合中的元素类型) |
| K | Key |
| V | Value |
| N | Number |

### 多个类型参数

定义泛型类或接口时可以使用多个类型参数，一个典型案例就是 Map，它存储一对键值对，我截取了一段 Map.Entry 的源码：

```java
interface Entry<K,V> {

    K getKey();
    V getValue();
    V setValue(V value);
    ...
}
```

我们知道 HashMap 是继承了 Map 的，所以在使用时可以这样使用：

```java
HashMap<String, Integer> map = new HashMap<>();
```

### 通配符与 T 的区别

在接触泛型之初，我一直好奇 **类型参数**(**type parameters**，即 T, E, etc.) 和 **通配符**(**wildcard**) 之间有什么区别，因为它们的功能看上去有的重叠。以下是我总结自 Java 的官方文档以及 StackOverFlow 上面的解答。

**类型参数与通配符的区别：**

在定义泛型类或接口时，只能使用类型参数，并可以在类内部引用：

```java
public class Box<T> {
    private T t;
    ...
}
```

不可以使用通配符，因为`?`不能代表一种具体类型，而`T`代表一种具体类型，只是暂时未定。

```java
public class Box<?> { // Error!!!
    private ? t; // Error!!!
    ...
}
```

通配符可以使用 **super** 关键字定义**下界**，而类型参数不可以。

```java
public void bar(List<? super Number> list) {...}
```

如果需要在泛型类内部定义一个与当前参数类型无关的方法，可以使用`<?>`。

```java
public class Box<T> {

    private T t;// Used type 'T'

    public void printListSize(List<?> list){ // Not associated with type 'T'
        System.out.println(list.size());
    }

    public static void main(String[] args) {
        Box<String> box = new Box<>();
        box.printListSize(new ArrayList<>(Arrays.asList(1,2,3,"aa",'s'))); // 5
    }
}
```

实际上，查看 Java 的 **Class** 类的源码，可以看到`<?>`有被广泛的使用，是因为`Class<T>`中的大多数的方法都不依赖于 T。

有些人认为，如果你不需要使用声明的参数类型，那就可以使用`?`使得代码更简单更具可读性。所以，如果你需要定义一种可变的类型，并且在限定范围内使用到这种类型，请使用`<T>`，否则就可以使用`<?>`。

## 泛型示例之 LinkedStack

下面是一个使用 Java 泛型编写的 LinkedStack 的实例代码：

```java
public class LinkedStack<T> {
    private static class Node<T> {
        T item;
        Node<T> next; //下一个节点的引用，类似于C中的指针

        Node() { //没有参数的Constructor，用于初始化时生成栈顶
            item = null;
            next = null;
        }

        Node(T item, Node<T> next) {
            this.item = item;
            this.next = next;
        }

        boolean isEmpty() { //栈空标记
            return item == null && next == null;
        }
    }

    private Node<T> top = new Node<>(); //初始化栈顶

    public void push(T item) {
        top = new Node<>(item, top); //与上一节点链接
    }

    public T pop() {
        T result = top.item;
        if (!top.isEmpty()) {
            top = top.next; //指针后移
        }
        return result;
    }

}
```

## 参考

- [官方文档](https://docs.oracle.com/javase/tutorial/java/generics/types.html)
- [What is the difference between '?', 'E', and 'T' for Java generics?](https://stackoverflow.com/questions/6008241/what-is-the-difference-between-e-and-t-for-java-generics)