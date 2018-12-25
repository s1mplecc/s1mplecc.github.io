---
title: Java8 函数接口
date: 2018-11-16T12:33:17.000Z
tags: ['Java', 'FP']
categories: []
---
## 函数接口

在函数式编程中，**纯函数**的定义是：
1. 此函数在相同的输入值时，需产生相同的输出。函数的输出和输入值以外的其他隐藏信息或状态无关，也和由 I/O 设备产生的外部输出无关。
2. 该函数不能有语义上可观察的函数副作用，诸如“触发事件”，使输出设备输出，或更改输出值以外物件的内容等。

为使 Java 支持函数式编程，**函数接口**（**Functional Interfaces**）正是 Java 8 引入的一个核心概念。函数接口要求接口中只能定义个**唯一一个抽象方法**。同时，引入一个新的注解： `@FunctionalInterface`，作为函数接口的标记。

Java 8 提供了四大类函数接口：**谓词** `Predicate`，**函数** `Function`，**提供者** `Supplier`，**消费者** `Consumer`。以及衍生的函数接口，比如 `BiFunction<T, U, R>` 支持两个入参的函数等等，它们都定义在 `java.util.function` 包下。

### Lambda 表达式

函数式接口的重要属性是：**我们能够使用 Lambda 实例化它们**。Lambda 表达式的引入给开发者带来了不少优点：在 Java 8 之前，匿名内部类，监听器和事件处理器的使用都显得很冗长，代码可读性很差，Lambda 表达式的应用则使代码变得更加紧凑，可读性增强；Lambda 表达式使**并行操作大集合**变得很方便，可以充分发挥多核 CPU 的优势，更易于为多核处理器编写代码。

Lambda 表达式由三个部分组成：第一部分为一个括号内用逗号分隔的函数入参；第二部分为箭头符号 `->`；第三部分为方法体，可以是表达式和代码块。语法如下：

- `(parameters) ‑> expression` 方法体为表达式，该表达式的值作为返回值返回。

- `(parameters) ‑> { statements; }` 方法体为代码块，必须用 `{}` 包裹起来，若该函数需要返回值则需要 `return`。

用 Lambda 表达式替换匿名内部类的写法如下。再进一步，可以使用**方法引用**（**Method Reference**）简写

```java
// Anonymous inner class
Optional.of("abc").map(new Function<String, Integer>() {
    @Override
    public Integer apply(String s) {
        return s.length();
    }
});

// Lambda
Optional.of("abc").map(s -> s.length());

// Method reference
Optional.of("abc").map(String::length);
```

使用 `parallelStream()` 和 Lambda 表达式并行操作集合，默认线程数为 CPU 核数
```java
urls.parallelStream()
        .filter(url -> url.endsWith(".html"))
        .map(url -> htmlParser.contentFrom(url))
        .reduce((x, y) -> x + y)
        .orElse("");
```

### 谓词 Predicate

在计算机语言的环境下，谓词是指条件表达式的求值返回真或假的过程。在 Java 中就是进行逻辑判断后返回值为 `boolean` 的函数，对应函数接口为 `Predicate<T>`。在针对集合进行筛选时，通常会使用谓词对集合元素进行条件判断。

```java
@FunctionalInterface
public interface Predicate<T> {

    boolean test(T t);

    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }

    default Predicate<T> negate() {
        return (t) -> !test(t);
    }

    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }

    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
}
```

`Predicate<T>` 接口定义的唯一抽象方法 `test(T t)` 用以验证输入参数是否满足谓词条件。同时，它还提供了默认方法用以支持**组合的与、或、非逻辑运算**。

```java
Predicate<Apple> p1 = p ‑> p.getColor() == Color.RED;
Predicate<Apple> p2 = p ‑> p.getSize() == Size.Medium;

apples.stream().filter(p1.and(p2));
```

### 函数 Function

`Function<T, R>` 接口代表一个函数，准确地说，代表了具有输入输出的函数，这也是最常见的函数定义。具有两个入参的函数对应接口为 `BiFunction<T, U, R>` 。

```java
/**
 * @param <T> the type of the input to the function
 * @param <R> the type of the result of the function
 */
@FunctionalInterface
public interface Function<T, R> {

    R apply(T t);

    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }

    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }
    
    static <T> Function<T, T> identity() {
        return t -> t;
    }
}
```

函数也可以进行组合，组合后返回的结果也是一个函数，这就是所谓的**组合子**。组合子方法分别为：
- `compose(before)`，组合前一个函数，相当于 `V -> T -> R` 
- `andThen(after)`，组合后一个函数，相当于 `T -> R -> V`

### 提供者 Supplier

当一个函数仅仅用于返回结果时，我们可以将这种函数称为**提供者**，接口定义为 `Supplier<T>`。

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

只提供一个 `get()` 方法，我们可以将这种提供者代表的函数看做是一种 Query 操作。

### 消费者 Consumer

与提供者对应的接口则被称之为**消费者**，用以接收一个输入，但却并不关心返回结果，即返回值为 `void`。消费者的对应接口为 `Consumer<T>`。

```java
/**
 * Represents an operation that accepts a single input argument and returns no
 * result. Unlike most other functional interfaces, {@code Consumer} is expected
 * to operate via side-effects.
 */
@FunctionalInterface
public interface Consumer<T> {

    void accept(T t);

    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```

不像其他函数接口，`Consumer` 通常是**有副作用的**。这也不难理解，该函数只有输入没有输出，那数据通常是被外部系统消费了。同时 `Consumer` 支持 `andThen(after)` 组合

```java
Consumer<String> c1 = s -> System.out.println(s);
Consumer<String> c2 = s -> storeToDB(s);

Optional.of("abc").ifPresent(c1.andThen(c2));
```

这种消费者代表的函数相当于是一个 Command 操作。软件设计的一条原则就是 **CQS**（Command-Query Separation）命令-查询分离原则，即保证一个方法体内只有 Query 和 Command 的一种，就是因为查询无副作用而命令可能会改变数据状态。