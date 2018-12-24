---
title: 结合案例使用 Java 注解和反射
date: 2018-11-29T04:53:50.000Z
tags: ['Java']
categories: []
---
## 前言

在项目开发时遇到这样一个场景：从上游传过来一个实体类对象 `newEntity`，但它只有部分字段，需要去库中查出对应的旧对象 `oldEntity` 做一次补全（相当于一次部分更新）。

一开始我们这样编码的：

```java
public FlightBasic merge(FlightBasic newEntity){
    if (newEntity.getAirportCode() != null){
        this.setAirportCode(newEntity.getAirportCode());
    }
    if (newEntity.getCraftNo() != null){
        this.setCraftNo(newEntity.getCraftNo());
    }
    if (newEntity.getCraftType() != null){
        this.setCraftType(newEntity.getCraftType());
    }
    // too long ...
    return this;
}
```

很快我们发现了问题：不仅类的字段很多，这样的 Entity 也有很多，所以到处都是又臭又长的 `merge` 方法。

其实不难总结这些代码的共性：

- 一个 Entity 只能和本身的类型 Merge
- 代码都是判空后对属性的 `getter/setter` 方法的重复调用

为了简化代码（偷懒），就在想能不能通过统一的处理完成这些 Merge 逻辑。首先想到的是代码生成，类似 MyBatis Generator，重复的工作交给脚本或者工具多好。但是很快就否定了这种方案，配置这些类的工作量也不小，而且很多类中有自己的业务逻辑，一不小心覆盖了也不行。那注解行不行呢？Spring Boot 就大量使用注解替换了之前配置繁杂的 XML 文件。实际编码后验证是可行的！本文将介绍如何**配合使用 Java 的注解和反射**实现上述问题中的 Merge 操作。示例代码已上传到 [GitHub](https://github.com/s1mplecc/merging) 上。

## 注解

每当你创建描述符性质的类或者接口时，一旦其中包含重复性的工作，就可以考虑使用注解来简化与自动化该过程。一个注解准确意义上来说，只不过是一种特殊的注释而已，如果没有**解析**它的代码，它可能连注释都不如。而解析注解往往有两种方式，一种是**编译期扫描**，典型的例如 `@Override`，这种只适用于编译器已经认识的注解，一般都是 JDK 内置注解；另一种则是通过**运行期反射**，也是在我们自定义注解后需要自己编码的。

### 元注解

Java提供了四种元注解，专门负责**修饰其他注解**：

- `@Target`：注解的作用目标
- `@Retention`：注解的生命周期
- `@Documented`：注解是否要包含在 JavaDoc 文档中
- `@Inherited`：子类是否继承该注解

#### @Target

`@Target` 用于定义注解的**作用域**，源码如下：

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Target {
    /**
     * Returns an array of the kinds of elements an annotation type
     * can be applied to.
     */
    ElementType[] value();
}
```

其中 `ElementType` 是一个枚举类型，包含以下值：

- `ElementType.TYPE`：允许注解作用在类、接口和枚举上
- `ElementType.FIELD`：允许作用在属性字段上
- `ElementType.METHOD`：允许作用在方法上
- `ElementType.PARAMETER`：允许作用在方法参数上
- `ElementType.CONSTRUCTOR`：允许作用在构造器上
- `ElementType.LOCAL_VARIABLE`：允许作用在本地局部变量上
- `ElementType.ANNOTATION_TYPE`：允许作用在注解上
- `ElementType.PACKAGE`：允许作用在包上

当允许有多个作用域时使用花括号 `{}` 包裹，比如这样：`@Target({ ElementType.TYPE, ElementType.FIELD })`

#### @Retention

`@Retention` 用于标明注解存在的**生命周期**，源码如下：

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Retention {
    /**
     * Returns the retention policy.
     */
    RetentionPolicy value();
}
```

`RetentionPolicy` 也是一个枚举类型，包含以下值：

- `RetentionPolicy.SOURCE`：在源文件中保留，编译为 .class 文件时会被丢弃，比如 `@Override`、`@SuppressWarnings`
- `RetentionPolicy.CLASS`：.class 文件中会保留，但运行时丢弃
- `RetentionPolicy.RUNTIME`：运行时保留，可以通过**反射获取**

剩下两种类型的注解用的不多，也比较简单，这里不再详细的介绍了，只需要知道他们各自的作用即可。`@Documented` 修饰的注解，当执行 JavaDoc 文档打包时会被保存进文档，否则将被丢弃。`@Inherited` 注解修饰的注解是有**继承性**的，也就说我们的注解修饰了一个类，而该类的子类将自动继承父类的该注解。该注解只作用于类，对属性或方法无效。

## 反射

Java 反射机制是指在**程序运行时（Runtime）识别和使用对象的类型信息**。以下内容摘自《Thinking in Java》，是我认为的有利于理解反射的核心概念。

> 要理解反射的工作原理，首先必须知道**类型信息**在运行时是如何表示的。这项工作是由称为 **Class** 对象的特殊对象完成的，它包含了与类有关的信息。事实上，Class 对象就是用来创建类的所有“常规”对象的。
> 
> 类是程序的一部分，每个类都有一个 Class 对象。换言之，每当编写并且编译了一个新类，就会产生一个 Class 对象（更恰当地说，是被保存在一个同名的 `.class` 文中）。为了生成这个类的对象，运行这个程序的 Java 虚拟机（JVM）将使用被称为**类加载器**的子系统。类加载器首先检查这个类的 Class 对象是否已经加载。如果尚未加载，默认的类加载器就会根据类名查找 `.class` 文件。**一旦某个类的 Class 对象被载入内存，它就被用来创建这个类的所有对象**。
> 
> `Class` 类和 `java.lang.reflect` 类库一起对反射的概念提供了支持，该类库包含 `Field`、`Method` 以及 `Constructor` 类。这些类型的对象是由 JVM 在运行时创建的，用以**表示未知类里对应的成员**。这样你就可以使用 `Constructor` 创建新的对象，用 `get()` 和 `set()` 方法读取和修改与 `Field` 对象关联的字段，用 `invoke()` 方法调用与 `Method` 对象关联的方法等等。另外，还可以调用 `getFields()`、`getMethods()` 和 `getConstructors()` 等方法以返回表示字段、方法以及构造器的对象的数组（可以查看 Class 类源码了解更多资料）。
> 
> 重要的是，要认识到反射机制并没有什么神奇之处。当通过反射与一个未知类型的对象打交道时，JVM 只是简单地检查这个对象，看它属于哪个特定的类。在用它做其他事情之前必须先加在那个类的 Class 对象。对于反射机制来说，`.class` 文件在编译时是不可获取的，所以是在运行时打开和检查 `.class` 文件。

这里简单的介绍一下我们编码时用到的与反射有关的方法：

- `obj.getClass();` Java 提供了三种方式获取类的 Class 对象的引用：`Class.forName()` 通过类名获取；`obj.getClass();` 通过对象获取和 `Object.class` 通过类字面常量获取。
- `getDeclaredFields();` 获取类中声明的所有字段，不包括父类和接口中的字段。它与 `getFields()` 的区别在于，`getFields()` 是获取所在类以及父类和接口中的所有访问修饰符为 `public` 的字段。
- `getAnnotation();` 获取标注的注解实例对象。

## 实现

### Merge 粒度与等级

回到我们的问题上来，首先考虑一个问题：Merge 的粒度应该位于什么层次？如果是类级别的，意味着整个类的所有属性将一视同仁，要么都合并要么都不，显然这样是不行的。Merge 的**粒度应该在属性字段上**，为此，我们专门定义了字段的 Merge 等级 `Level`：

```java
public enum Level {
    /* 无论新值为何值，都会覆盖旧值 */
    Mandatory,
    /* 如果新值不为null，覆盖旧值，否则不覆盖 */
    Required,
    /* 忽略，不做合并处理 */
    Ignored;
}
```

### 自定义注解 @MergeOn

接下来就需要自定义我们的注解：`@MergeOn`，作用在字段上，在运行时有效：

```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MergeOn {
    Level level() default Level.Required;
}
```
使用 `default` 关键字可以设置缺省等级为 `Required`。

### 通过反射将字段设值

现在，还差最后一步，就是如何通过反射将字段设值。要想让实体类实现 Merge，首先需要定义一个接口 `Merging` 让实体类去继承它，然后接口中得有一个默认方法 `mergeWith(T newEntity)`，它将做下面这几件事：

- 通过反射获取 Entity 声明的所有字段
- 通过反射获取字段上 `@MergeOn` 注解的 Level
- 通过 Level 走不同的分支去给字段设值，是强制或是忽略亦或是只有非空情况下才设值。

```java
public interface Merging<T extends Merging> {

    default void mergeWith(T newEntity) {
        if (this.getClass() != newEntity.getClass()) {
            throw new MergeWithDifferentClassTypeException(
                    String.format("<%s> can not merge with other class type <%s>", this.getClass().getName(), newEntity.getClass().getName()));
        }

        Field[] fields = this.getClass().getDeclaredFields(); // 1

        for (Field field : fields) {
            MergeOn mergeOn = field.getAnnotation(MergeOn.class); // 2
            field.setAccessible(true); // 3

            try {
                if (mergeOn == null || mergeOn.level().isRequired()) { // 4...
                    Object newFieldValue = field.get(newEntity);
                    if (newFieldValue != null) {
                        field.set(this, newFieldValue);
                    }
                    continue;
                }
                if (mergeOn.level().isMandatory()) { // 4...
                    field.set(this, field.get(newEntity)); // 5
                }
            } catch (IllegalAccessException ex) {
                throw new MergeOnFieldIllegalAccessException(field.getName(), ex);
            }
        }
    }
}
```

我们看看代码具体的实现步骤（注释中标明了顺序）：

1. `this.getClass().getDeclaredFields();` 获取类中声明的所有字段，不包含父类和接口中的字段。`this` 指向的是实现接口的实体类。
2. `field.getAnnotation(MergeOn.class);` 获取字段上的 `@MergeOn` 注解实例。
3. `field.setAccessible(true);` 由于字段通常是 `private` 修饰的，就需要**获取访问权**（并不是修改实际权限），否则将抛出 `IllegalAccessException` 异常。由此可见，反射有可能破环封装性。
4. 根据 Level 做相应设值。如果无注解或 Level 为 Required，则新值非空时覆盖旧值；如果是强制（Mandatory）则始终覆盖；否则（Ignored），忽略不做处理。
5. `field.set(this, field.get(newEntity));` 字段设值的实际操作，接口设计看上去有点反人类。第一个参数为需要设置字段的对象，此处为 oldEntity，第二个参数为要设置的值，此处为新对象的字段值。

## 验证

接下来我将演示如何使用这个接口，并编写单元测试验证其正确性。

首先定义实体类 Entity。

- 为了测试全面性，加入了各种类型的属性字段，包括基础类型、集合、数组和类；同时，`@MergeOn` 注解覆盖了所有 Level，还包括缺省和无注解的情况。
- 为了简单，直接在字段声明时赋初值。即 `new Entity()` 出来的可以看成旧的模型对象（oldEntity）。

```java
public class Entity implements Merging<Entity> {
    @MergeOn(level = Level.Ignored)
    private String string = "string";
    @MergeOn
    private int anInt = 1;
    @MergeOn(level = Level.Mandatory)
    private List<String> stringList = Arrays.asList("a", "b", "c");
    @MergeOn(level = Level.Required)
    private String[] strings = new String[]{"sArr1", "sArr2"};
    private Email email = new Email("zz@163.com");
    private Set<Email> emails = Sets.newLinkedHashSet(new Email("zz@163.com"), new Email("zz@gmail.com"));

    public static class Email {
        private String value;

        public Email(String value) {
            this.value = value;
        }
    }
}
```

理想情况下，`string` 字段新值应当被忽略；`stringList` 在新值为 null 时也会被强制覆盖；其他缺省和无注解的应当行为与 `@MergeOn(level = Level.Required)` 一致，即非空时才会覆盖旧值。以下为测试代码：

```java
@Test
public void should_merge_new_entity_with_different_merge_level() {
    Entity newEntity = new Entity();
    newEntity.setString("newString");  // Ignored
    newEntity.setAnInt(2);
    newEntity.setStringList(null);  // Mandatory
    newEntity.setStrings(new String[]{"newS1", "newS2", "newS3"});
    newEntity.setEmail(new Entity.Email("newEmail@163.com"));
    newEntity.setEmails(null); // Required, wished not be overwrited

    Entity entity = new Entity(); // oldEntity
    entity.mergeWith(newEntity);

    assertThat(entity.getString()).isEqualTo("string");
    assertThat(entity.getAnInt()).isEqualTo(2);
    assertThat(entity.getStringList()).isNull();
    assertThat(entity.getStrings()).containsSequence("newS1", "newS2", "newS3");
    assertThat(entity.getEmail().getValue()).isEqualTo("newEmail@163.com");
    assertThat(entity.getEmails()).hasSize(2);
    assertThat(entity.getEmails()).contains(new Entity.Email("zz@163.com"));
    assertThat(entity.getEmails()).contains(new Entity.Email("zz@gmail.com"));
}
```

## 参考

- [JAVA 注解的基本原理 —— 掘金](https://juejin.im/post/5b45bd715188251b3a1db54f)
- Thinking in Java 第14章：类型信息