---
title: 使用流畅接口编写 Kafka 集成测试框架
tags: [DSL, API Design, Java]
categories: [Design]
date: 2020-01-03 19:26:40
---
## 前言

该文档是我在团队中负责集成测试模块时为团队成员编写的 API 接口文档，这次拿过来修改了一下，并已经将业务无关的代码剥离出来上传到 [GitHub](https://github.com/s1mplecc/kafka-integration-test) 上。主要是向大家展示一种流畅接口设计框架代码的模式。

所谓流畅接口，可以参考我之前翻译的 Martin Flower 的[博客译文](https://s1mple.online/2019/01/23/Fluent-Interface-%E2%80%94%E2%80%94-Martin-Fowler-%E5%8D%9A%E5%AE%A2%E8%AF%91%E6%96%87/)。简单来说，流畅接口被设计为**可读的**和**流式的**，使用起来几乎和自然语言一般流畅，并且配合 IDE 的智能提示，易于 API 使用者的理解和使用。譬如下面我编写的集成测试用例：

```java
public class InboundToOutboundExampleJobTest extends JobIntegrationTest {

    @Test
    public void should_send_ACTT_and_receive_trigger_event_from_kafka() {
        kafkaSuite.send(new ResourceFile("sample/send/pek/ACTT.xml"))
                .toTopic()
                .ofConfig("inbound_imf");

        SubjectEvent event = kafkaSuite
                .await().latestOne()
                .fromTopic()
                .ofConfig("trigger")
                .toJavaObject(SubjectEvent.class);

        assertThat(event.getEventName()).isEqualTo("FLOP_ACTT");
        assertThat(event.getGids().size()).isEqualTo(1);
        assertThat(event.getGids().get(0)).isEqualTo("PEK_80664162170310656");
    }
}
```

代码读起来非常明了，`KafkaSuite` 是我编写的 Kafka 集成测试套件，负责将 xml 文件发送至 Kafka 的一个 Topic 中，该 Topic 配置在配置文件中，对应着 `inbound_imf` 项。同时 `KafkaSuite` 监听另一个 Topic `trigger` 的最新一条消息并转换成 Java 对象，对结果进行验证（其间过程 xml 文件会被其他模块解析并经过一系列的业务处理）。

上述代码中，`assertThat().isEqualTo()` 就是断言框架 `assertj` 的 API，它是一个非常优秀的流畅接口的框架，建议大家阅读其源代码。我在编写 Kafka 集成测试框架时就借鉴了这种编码模式。

## 集成测试

代码目前完成了 Kafka 集成测试的发送和监听功能。我们的流处理框架采用的是 Flink，将任务分解为一个个 Job 独立运行，所以集成测试首先得启动 Job，并且加载包括 Kafka IP地址等相关配置。为此我编写了 `JobIntegrationTest` 类，并规定所有集成测试用例必须继承该类。此外，`KafkaSuite` 是负责 Kafka 集成测试的套件，封装了各种流畅接口 API。

### JobIntegrationTest

所有的集成测试必需继承 JobIntegrationTest 类，该类提供 final 的 `setUp()` 方法和 `tearDown()` 方法。

集成测试运行的流程是 `SetUp -> All Tests -> TearDown`：

1. `setUp()` 方法中使用了单独的线程启动 Job，所以**不会阻塞主线程（测试方法）的运行**。

2. 测试方法主体由集成测试使用者编写，如果用到 KafkaSuite 的 send 和 await 方法则会被阻塞，直到**成功发送或者接收到消息**，或者**超时抛出异常**退出。

3. `tearDown()` 方法关闭 Job 和用到的资源。

### KafkaSuite

目前集成测试实现了 Kafka 消息的**阻塞发送**和**阻塞监听**，抽象出来的 KafkaSuite 提供了流畅接口以支持集成测试。`kafkaSuite` 作为 JobIntegrationTest 的 protected 字段可以直接在集成测试类中使用。我将 KafkaSuite 的流畅接口分为三大阶段：配置阶段（PrepareStep）、发送阶段（SendStep）和接受阶段（AwaitStep）。具体的流畅接口使用可以查看 KafkaSuiteTest 或者直接阅读源码 JavaDoc。

#### PrepareStep

由于 JobIntegrationTest 会加载配置文件并由此创建 KafakSuite 实例，所以正常情况下不需要独立配置 IP、端口等。但为提供给特殊需求使用，仍设计相关接口：

```java
kafkaSuite.clusterIps("172.20.10.120:9092,172.20.10.152:9092,172.20.10.171:9092")
            .noSsl()
            .send() //...
```

#### SendStep

KafkaSuite 发送消息是阻塞的，设置的超时时间是 10s，**如果 10s 未发送成功则会抛出 SendTimeoutException 异常**，这时候需要检查配置文件中的 IP 和 SSL 配置是否正确。

API 使用如下：

```java
kafkaSuite.send(new ResourceFile("kafka/test-suite.json"))
            .toTopic()
            .ofConfig("INTEGRATION-TEST");
```

`send()` 方法提供发送字符串、文件或数据流的接口：

```java
public interface DSLSendStep {
    DSLSendToStep send(String value);

    DSLSendToStep send(ResourceFile file) throws ResourceFileNotFoundException;

    DSLSendToStep send(InputStream inputStream) throws IOException;
}
```

`toTopic()` 方法可以直接 `toTopic("xxx")` 指定明确 Topic，但更加推荐从配置文件读取的写法：

```java
public interface DSLSendToStep {
    void toTopic(String topic);

    DSLSentLoadConfigStep toTopic();
}
```

#### AwaitStep

KafkaSuite 监听消息同样是阻塞的。设置的**默认超时时间是 60s**，也提供自定义超时时间的接口如 `await(300)`，单位为秒，**超过这个时间未接收到消息则抛出 AwaitTimeoutException**。理论上我们都是先发再收，所以如果发送成功则代表 Kafka 连接不存在问题（也就是配置没有问题），接收不到就应该是业务代码有问题，这时候就需要 Debug 排查问题。

监听结果提供**监听一条**和**监听多条**的 API，但不管是使用哪一个，`await()` 后会取到 Kafka 该 Topic 的上一个 offset 以来的所有新消息，然后变更 offset。可以根据 Index 获取多条消息里的某一条，Index 从 0 开始计算。大致用法是：使用 `await().latestOne()` 获取最新一条，`await().multi()` 或者直接 `await()` 获取多条。

```java
public interface DSLAwaitIndexStep extends DSLAwaitMultiFromStep {

    // --------------------------------------------------------------------------------------------
    //  Kafka Consumer Fetch latest one record
    // --------------------------------------------------------------------------------------------

    DSLAwaitOneFromStep latestOne();

    DSLAwaitOneFromStep one();

    DSLAwaitOneFromStep lastOne();

    // --------------------------------------------------------------------------------------------
    //  Kafka Consumer Fetch record by index in records
    // --------------------------------------------------------------------------------------------

    DSLAwaitOneFromStep one(int index);

    DSLAwaitOneFromStep firstOne();

    DSLAwaitOneFromStep secondOne();

    DSLAwaitOneFromStep thirdOne();

    // --------------------------------------------------------------------------------------------
    //  Kafka Consumer Fetch all latest records
    // --------------------------------------------------------------------------------------------

    DSLAwaitMultiFromStep all();

    DSLAwaitMultiFromStep multi();

    // --------------------------------------------------------------------------------------------
    //  Kafka Consumer Fetch multi records directly from topic
    // --------------------------------------------------------------------------------------------

    @Override
    DSLAwaitMultiLoadConfigStep fromTopic();

    @Override
    DSLAwaitMultiRecordsStep fromTopic(String topic);

    @Override
    DSLAwaitMultiRecordsStep fromTopic(String topic, String groupId);
}
```

下面是**监听一条并且验证消息是否正确**的写法，这里使用了 `toJavaObject()` 转成了 Java 对象，这个是我提供的转换消息的 API，后面会有介绍。

```java
kafkaSuite.send("{\"name\":\"jack\",\"age\":25}")
                .toTopic()
                .ofConfig("INTEGRATION-TEST");
                
Person record = kafkaSuite
                .await().latestOne()
                .fromTopic()
                .ofConfig("INTEGRATION-TEST")
                .toJavaObject(Person.class);
                
assertThat(record.getName()).isEqualTo("jack");
assertThat(record.getAge()).isEqualTo(25);
```

**由于有的业务场景会出口多条消息，所以接下来演示验证多条消息的正确写法**。提供了 `fetchFirstOne()` 和 `fetchOne(int index)` 根据 Index 获取多条中的一条等 API：

```java
kafkaSuite.send("{\"name\": \"jack\", \"age\": 25}")
        .toTopic()
        .ofConfig("INTEGRATION-TEST");

kafkaSuite.send("{\"name\": \"tony\", \"age\": 20}")
        .toTopic()
        .ofConfig("INTEGRATION-TEST");

DSLAwaitMultiRecordsStep records = kafkaSuite
        .await().multi()
        .fromTopic()
        .ofConfig("INTEGRATION-TEST");

Person jack = records.fetchFirstOne().toJavaObject(Person.class);
Person tony = records.fetchOne(1).toJavaObject(Person.class);

assertThat(jack.getName()).isEqualTo("jack");
assertThat(jack.getAge()).isEqualTo(25);
assertThat(tony.getName()).isEqualTo("tony");
assertThat(tony.getAge()).isEqualTo(20);
```

最后结果的**输出提供了转换和输出到文件的 API**。需要注意，输出到文件只是方便调试 Bug 时使用，不应该出现在正式的代码中；输出的文件被写入到 `target/test-classes/resources` 下。

```java
public interface DSLAwaitOneRecordStep extends DSLAwaitRecordStep, DSLAwaitOneRecordOutputStep, DSLAwaitOneRecordConvertStep {

    @Override
    String toString();

    @Override
    JsonObject toJsonObject();

    @Override
    JSONArray toJsonArray();

    @Override
    void toXml(); // 编码未完成

    @Override
    <T> T toJavaObject(Class<T> clazz);

    @Override
    void writeAsTxt(String absolutePath);

    @Override
    void writeAsTxt(ResourceFile resourceFile);
}
```

## 代码结构

流畅接口 API 的设计一个重中之重就是**接口和抽象类的使用**，为了契合流式接口，有时不能按照面向对象的模式设计接口继承关系。简言之，在编写流畅接口 API 框架时，大致思路是先编写接口（Interface），按照流式的原则设计接口之间的继承关系，然后再编写具体的实现类。以下是我的代码结构，与 `impl` 文件夹并列的都是接口。具体代码详见我的 [GitHub](https://github.com/s1mplecc/kafka-integration-test) 项目。

```
.
├── Assertions.java
├── JobIntegrationTest.java
├── KafkaSuite.java
└── core
    ├── dsl
    │   ├── await
    │   │   ├── DSLAwaitFromStep.java
    │   │   ├── DSLAwaitIndexStep.java
    │   │   ├── DSLAwaitLoadConfigStep.java
    │   │   ├── DSLAwaitMultiFromStep.java
    │   │   ├── DSLAwaitMultiLoadConfigStep.java
    │   │   ├── DSLAwaitMultiRecordsConvertStep.java
    │   │   ├── DSLAwaitMultiRecordsFetchOneStep.java
    │   │   ├── DSLAwaitMultiRecordsStep.java
    │   │   ├── DSLAwaitOneFromStep.java
    │   │   ├── DSLAwaitOneLoadConfigStep.java
    │   │   ├── DSLAwaitOneRecordConvertStep.java
    │   │   ├── DSLAwaitOneRecordOutputStep.java
    │   │   ├── DSLAwaitOneRecordStep.java
    │   │   ├── DSLAwaitRecordStep.java
    │   │   ├── DSLAwaitStep.java
    │   │   └── impl
    │   │       ├── AbstractDSLAwaitFromStep.java
    │   │       ├── AbstractDSLAwaitLoadConfigStep.java
    │   │       ├── DSLAwaitIndexStepImpl.java
    │   │       ├── DSLAwaitMultiFromStepImpl.java
    │   │       ├── DSLAwaitMultiLoadConfigStepImpl.java
    │   │       ├── DSLAwaitMultiRecordsStepImpl.java
    │   │       ├── DSLAwaitOneFromStepImpl.java
    │   │       ├── DSLAwaitOneLoadConfigStepImpl.java
    │   │       └── DSLAwaitOneRecordStepImpl.java
    │   ├── send
    │   │   ├── DSLSendStep.java
    │   │   ├── DSLSendToStep.java
    │   │   ├── DSLSentLoadConfigStep.java
    │   │   └── impl
    │   │       ├── DSLSendToStepImpl.java
    │   │       └── DSLSentLoadConfigStepImpl.java
    │   └── start
    │       ├── DSLPrepareStep.java
    │       └── DSLStartStep.java
    └── exceptions
        ├── AwaitTimeoutException.java
        ├── InvalidIntegrationTestNameException.java
        ├── SendNullValueException.java
        └── SendTimeoutException.java
```
