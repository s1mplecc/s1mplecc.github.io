---
title: Java8 Optional 源码阅读
date: 2018-11-27T13:26:41.000Z
tags: ['Java', 'FP']
categories: [Languages]
---
## 前言

Optional 是 Java8 引入的一个重要特性，它是一个容器，里面装着一个可能为空可能不为空的对象。在它出现之前，为避免空指针异常我们可能会这样编码：

```java
public String getLastFour(Employee employee) {
    if(employee != null) {
        Address address = employee.getPrimaryAddress();
        if(address != null) {
            ZipCode zip = address.getZipCode();
            if(zip != null) {
                return zip.getLastFour()
            }
        }
    }
    throw new FMLException("Missing data");
}
```

显然代码嵌套层次很深不够整洁。而用 Optional 将值包裹起来后，我们可以不再关注于值会不会为 `null`，会不会抛空指针，而将注意力集中在**对数据的操作**。并且 Optional 提供了 `map`、`flatMap`、`filter` 等方法让我们可以进行**函数式风格**的编码。那么上诉代码可以如此改写：

```java
public String getLastFour(Optional<Employee> employee) {
    return employee.flatMap(employee -> employee.getPrimaryAddress())
                   .flatMap(address -> address.getZipCode())
                   .flatMap(zip -> zip.getLastFour())
                   .orElseThrow(() -> new FMLException("Missing data"));
} 
```

更确切的说，Optional 是一个 **Monad** 容器。那什么是 Monad 呢？可以参考下面这段话：` Think of monads as an object that wraps a value and allows us to apply a set of transformations on that value and get it back out with all the transformations applied.` 简单的说，Monad 是一个包裹了一个值的容器（值可以是单个对象也可以是集合），允许我们对该值进行一系列的转换（函数操作）后返回给m我们期望的值。也就是说，Monad 封装了接收函数作为入参的一些方法（filter、map、flatMap 等）。

那么现在，让我们从源码开始一步一步揭开它的神秘面纱。

## 工厂方法

Optional 使用私有化的构造函数和单例模式提供了**值为空的单例**，暴露的静态工厂方法为 `Optional.empty()`。该种设计感觉和 Java 设计模式中的**空对象模式**有异曲同工之妙，也是避免空指针的核心所在，后面我们将会提到。

```java
/**
 * Common instance for {@code empty()}.
 */
private static final Optional<?> EMPTY = new Optional<>();

private Optional() {
    this.value = null;
}

public static<T> Optional<T> empty() {
    @SuppressWarnings("unchecked")
    Optional<T> t = (Optional<T>) EMPTY;
    return t;
}
```

同时提供 `Optional.of(value)` 和 `Optional.ofNullable(value)` 两个工厂方法用于将 value 装入 Optional 容器中。注意 `of()` 方法在值为空时会抛出**空指针异常**。所以在允许入参值为空时使用 `ofNullable()`，它在值为空时调用 `Optional.empty()` 返回 `EMPTY` 单例。

```java
public static <T> Optional<T> of(T value) {
    return new Optional<>(value);
}

/**
 * @throws NullPointerException if value is null
 */
private Optional(T value) {
    this.value = Objects.requireNonNull(value);
}

public static <T> Optional<T> ofNullable(T value) {
    return value == null ? empty() : of(value);
}
```

## isPresent 和 get

Optional 提供的 `isPresent()` 用于判断容器中的值是否为空，`get()` 方法用来获取容器中的值。

```java
public boolean isPresent() {
    return value != null;
}

public T get() {
    if (value == null) {
        throw new NoSuchElementException("No value present");
    }
    return value;
}
```

这两个方法通常会组合起来使用：

```java
Optional<String> optString = Optional.of("aaa");

if (optString.isPresent()) {
    System.out.println(optString.get());
}
```

但我想说的是，Optional 的初衷不是为了让我们仅仅使用这两个方法，事实上也应当尽量避免使用这两个方法，而是多使用 `ifPresent()` 和 `orElse()` 等方法代替。因为一旦你这样用，就和使用 `if(xxx != null){}` 没什么区别了。

Optional 真正的核心就在于下面这些方法。

## Optional 与函数接口

Optional 对于 Java8 提供的四种**函数接口**：`Function`、`Predicate`、`Consumer`、`Supplier` 都包装了相应的方法，以满足我们可以进行**函数式**风格的编码。本部分介绍这些方法的实现和使用，为了直观，部分代码使用匿名内部类的方式。

关于函数接口（**Functional Interfaces**），我的上一篇博客 [Java8 函数接口](https://s1mple.xyz/java-functional-interfaces/)中有详细介绍。

### ifPresent

`ifPresent(consumer)` 方法接受一个**消费者函数** `Consumer` 入参，如果 Optional 容器中的值不为空，则调用 `Consumer` 的 `accept(value)` 方法消费容器中的值。若值为空则什么都不做。

```java
public void ifPresent(Consumer<? super T> consumer) {
    if (value != null)
        consumer.accept(value);
}
```

如下代码会打印容器中的值 “abc”

```java
Optional.of("abc")
        .ifPresent(new Consumer<String>() {
            @Override
            public void accept(String s) {
                System.out.println(s);
            }
        }); // print abc

// Lambda
Optional.of("abc").ifPresent(s -> System.out.println(s));
```

### filter

`filter(predicate)` 方法接受一个**谓词函数** `Predicate` 入参，调用谓词的 `test(value)` 判断容器中的值是否满足谓词条件，**满足则返回该容器本身**，不满足或值为空时返回 `Optional.empty()` 单例。

```java
public Optional<T> filter(Predicate<? super T> predicate) {
    Objects.requireNonNull(predicate);
    if (!isPresent())
        return this;
    else
        return predicate.test(value) ? this : empty();
}
```

以下代码分别展示了 `filter` 通过和不通过的情况：

```java
Optional.of("abc")
        .filter(new Predicate<String>() {
            @Override
            public boolean test(String s) {
                return s.length() > 5;
            }
        })
        .ifPresent(System.out::println); // do nothing

// Lambda
Optional.of("abc")
        .filter(s -> s.length() == 3)
        .ifPresent(System.out::println); // print abc
```

### map

`map(mapper)` 方法接受一个**函数** `Function` 入参，将容器中的值执行 `apply(value)` 操作后返回，返回的依然是 `Optional` 包裹的值，值的类型为 `apply()`  方法的返回类型，即 `map` 可能**改变容器中值的类型**。同样，值为空或 `apply()` 为空则返回 `empty()`。

```java
public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
    Objects.requireNonNull(mapper);
    if (!isPresent())
        return empty();
    else {
        return Optional.ofNullable(mapper.apply(value));
    }
}
```

以下代码展示了 `map` 的用法，通过 `.map(s -> s.length())` 返回类型已经从 `Optional<String>` 变为 `Optional<Integer>`

```java
Optional.of("abc")
        .map(new Function<String, Integer>() {
            @Override
            public Integer apply(String s) {
                return s.length();
            }
        })
        .ifPresent(System.out::println); // 3

// Lambda
Optional.of("abc").map(s -> s.length()); // Optional<Integer>
```
	
### flatMap

`flatMap` 可以视作一种特殊的 `map` 操作，可以视为 map + flatten 操作。仔细观察，它的入参接受的 Function 函数的输出泛型为 `Optional<U>`，所以不需要像 `map` 方法中再用 `ofNullable()` 包装成 Optional 容器。

```java
public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper) {
    Objects.requireNonNull(mapper);
    if (!isPresent())
        return empty();
    else {
        return Objects.requireNonNull(mapper.apply(value));
    }
}
```

`map` 与 `flatMap` 的区别如下。自定义的 `optionalLength(string)` 返回 `Optional<Integer>` 类型，所以 `map` 会返回 `Optional<Optional<Integer>>` 类型，只有调用 `flatMap` 才返回 `Optional<Integer>` 类型，相当于多做了一步 flatten 操作将其拍平为一维纬度。

```java
static Optional<Integer> optionalLength(String s) {
    if (s == null) {
        return Optional.empty();
    }
    return Optional.of(s.length());
}

Optional.of("abc").map(s -> optionalLength(s)); // Optional<Optional<Integer>>
Optional.of("abc").flatMap(s -> optionalLength(s)); // Optional<Integer>
```

### orElseGet 和 orElseThrow

`orElseGet(other)` 接受**提供者函数** `Supplier` 作为入参，当容器中的值为空时调用 `Supplier` 的 `get()` 方法获取值。`orElseThrow(supplier)` 则是值为空时抛出异常。

```java
public T orElseGet(Supplier<? extends T> other) {
    return value != null ? value : other.get();
}

public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
    if (value != null) {
        return value;
    } else {
        throw exceptionSupplier.get();
    }
}
```

使用时如下，若当前容器中的值为空则返回 `anotherString()` 的值。注意这一步已经跳出了 Optional 容器，返回的类型为容器中值的类型。

```java
static String anotherString() {
    return "anotherString";
}

Optional.of("abc")
        .filter(s -> s.startsWith("zzz")) // Optional.empty()
        .orElseGet(new Supplier<String>() {
            @Override
            public String get() {
                return anotherString();
            }
        }); // "anotherString"
        
Optional.of("abc")
        .filter(s -> s.length() == 3) // Optional["abc"]
        .orElseGet(() -> anotherString()); // "abc"

Optional.of("abc")
        .filter(s -> s.startsWith("zzz")) // Optional.empty()
        .orElseThrow(() -> new RuntimeException("no eligible values"));
```

## 总结

Optional 提供的这些方法让我们不再关注于是否会抛出空指针异常，而是通过函数的自由组合专注于对数据的操作，比如如下代码：
    
```java
Optional.ofNullable(people)
        .flatMap(people -> people.getName())
        .map(name -> name.toUpperCase())
        .filter(name -> name.startsWith("T"))
        .orElse("TONY");
```

其实 Optional 的核心在于提供的 `empty()` 单例，使得如上代码整条链路不会在某一步中断抛出空指针异常，而是只要其中一步值为空后，`Optional.empty()` **将在这条链路上继续向下传播**，直到最后 `orElse()` 等方法指名默认处理并跳出容器。

Optional 源码时有诸多值得我们借鉴取经的地方，诸如：

- Monad 容器的思想。容器接收一个值，进行函数操作后依然返回该容器，比如 `map`、`flatMap`、`filter`。这就使得我们可以随意组合以上方法，编写函数式风格的代码。类似的还有 Java 8 的 `Stream` 接口。

- 提供 `Optional.empty()` 类似空对象模式的空值单例。

- 严谨的编码，源码中有许多 `Objects.requireNonNull()` 对极端情况进行验证，比如 `flatMap` 方法中的 `Objects.requireNonNull(mapper);` 和 `Optional.ofNullable(mapper.apply(value));` 对于入参函数不为空和调用 `apply()` 后返回值不为空的验证。

- API 的全面性，当容器中没有你期待的值时提供 `orElse`、`orElseGet`、`orElseThrow` 等其他处理手段，不管是设置默认值还是抛出异常，API 都考虑到了。

目前在一些 ORM 框架中也加入了 Optional 的支持，比如 Hibernate 从 5.2 版本后提供 `loadOptional()` 方法返回 `Optional<T>` 类型。有兴趣的可以自己去研究。

## 参考

- [Functors, Applicatives, And Monads In Pictures](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html)
- [Understanding the Optional Monad in Java 8](https://medium.com/coding-with-clarity/understanding-the-optional-monad-in-java-8-e3000d85ffd2)