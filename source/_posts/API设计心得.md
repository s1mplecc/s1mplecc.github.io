---
title: API 设计心得
date: 2018-10-10T06:19:11.000Z
tags: [API Design]
categories: [Design]
---
## Preface

> 最近我在项目中单独编码了加载配置的模块，并提供给团队里的其他开发者使用。在设计 API 时有些心得体会，遂有了此博客记录下来。

## 项目结构

截取了部分项目结构，大致如下：

```java
// config module

├── ConfigFactory.java
├── YamlConverter.java
└── core
    ├── AbstractConfig.java
    ├── Config.java
    ├── ConfigFile.java
    ├── ConfigLoader.java
    ├── ...
    └── exceptions
        ├── ConfigFileNotFoundException.java
        ├── UnsupportedFileTypeException.java
        └── ...
```

`ConfigFactory` 和 `YamlConverter` 都是对外提供 API 的类，故放在最外层目录下，另外有个 **core** 的文件夹，存放间接使用到的类。类似于操作系统和用户 UI 界面，内核 core 封装你的实现，不暴露给用户，可以非常复杂。外层的 API 提供给用户使用，应该尽量满足**不变**的原则，即**隔离变化**，为了避免升级后用户需要改动代码。

## 易用性

API 应该保持简单易用，尽量可以**望文知义**。由于现在的 IDE 都有智能感知，会自动提示可调用的方法，一个设计优秀的 API 应该**方法名即能显示用途**，再不济，用户跟进方法阅读注释或者源码也应知道如何使用。**能避免用户必须查阅文档才知道如何使用的应当尽量避免**。

提高易用性可以从如下几各方面：

#### **静态方法**

静态方法不需要实例化，一些工具类就大量运用静态方法，比如 Apache 的 StringUtils 字符串处理工具类，`StringUtils.isEmpty()、StringUtils.upperCase()` 等几乎所有方法都是 `static` 的。

#### **Fluent Interface**

流畅接口符合人的阅读习惯，比如 jOOQ 数据库映射库，使用流畅接口模拟 SQL。

```java
create.selectFrom(a)
      .where(exists(selectOne()
                 .from(BOOK)
                 .where(BOOK.STATUS.equal(BOOK_STATUS.SOLD_OUT))
                 .and(BOOK.AUTHOR_ID.equal(a.ID))));
```
    
#### **命名**

不管是类名、方法名还是参数名，好的命名应该**语义清晰简单**、**符合直觉**、**易于记忆**和**引导用户**，应尽量简单或使用业界常用的命名，比如我在编写 config 模块时借鉴了 Typesafe，它的 API 这样使用：`ConfigFactory.load(String resourceBasename)`，清晰明了，并且我知道入参是基于 resource 相对路径的文件名。

**不要重复局部命名**。在有上下文环境的调用中，减少不必要的描述可以提高 API 的精简和清晰度。

```java
class User {
  // good
  setName() {}
  
  // bad
  setUserName() {}
}
```
#### **封装**

使用 Java 的访问控制修饰符 `private、default、protected` 对方法访问权限进行限定，该隐藏的隐藏，仅暴露对外公开接口。这样在 IDE 智能提示时也方便用户快速定位方法。

## 兼容性

良好的 API 设计利于未来的版本升级——升级带来的用户兼容性成本较低，或者框架开发者的兼容性包袱较轻。

为了将来的兼容性考虑，设计 API 时建议考虑这些因素：

#### **不要提前公开 API**

如果你的某个 API 是为将来预留的，那么不要开放，因为你不清楚未来的设计需求是怎样的，提前公开的 API 在将来改变的可能性非常高。

#### **预留足够的扩展点**

没有良好扩展性的 API 通常会因为频繁的需求变更而导致 API 间接变化，这都是兼容性成本。如果在良好的设计下预留了足够的扩展点，那么这样的 API 能够应对未来一段时间内未知的需求变化，使得 API 变化在可控范围内。

要预留扩展点就意味着通常应该**使用接口或者抽象**的概念来描述 API，建议用清晰定位的接口替代具体的类型。

#### **明确的 API 迁移说明**

如果某个 API 过时了，也不建议删除它；应该标记为过时，Java 中使用 `@Deprecated` 注解，并告诉使用者新的 API 是什么。当然如果这个 API 会导致出现不可接受的问题，也可以标记它无法通过编译。

## References

- [好的框架需要好的 API 设计 —— API 设计的六个原则
](https://walterlv.com/post/framework-api-design.html#%E4%B8%80%E8%87%B4%E6%80%A7)
- [接口设计六大原则](https://www.cnblogs.com/zfc2201/p/3423370.html)