---
title: 'TiCDC Sink 开发指南'
layout: post

categories: post
tags:
- Golang
- Go
- TiCDC
- TiDB
---

我过去近大半年的时间都在做 TiCDC [Sink 模块]的改造工作，目前新的 Sink 实现已经成功替换了旧的实现。最近有客户希望通过自己实现 Sink 的方式来接入 TiCDC，所以我想把这段时间的改造和设计经验分享出来，希望能帮助到大家。

此博客在 [GitHub](https://github.com/hi-rustin/hi-rustin.rs) 上公开发布。 如果您有任何问题或疑问，请在此处打开一个 [issue](https://github.com/hi-rustin/hi-rustin.rs/issues)。

> ⚠️ 注意：
> 1. 该指南主要面向开发者，如果您只是想使用 TiCDC，请参阅 [TiCDC 使用文档](https://docs.pingcap.com/zh/tidb/stable/ticdc-overview)。
> 2. 在阅读该指南前，请先阅读 [TiCDC 架构和数据同步链路解析](https://hi-rustin.rs/TiCDC-%E6%9E%B6%E6%9E%84%E5%92%8C%E6%95%B0%E6%8D%AE%E5%90%8C%E6%AD%A5%E9%93%BE%E8%B7%AF%E8%A7%A3%E6%9E%90/)了解 TiCDC 的基本架构和数据同步流程。

## 基本概念

> 可以先简单浏览这些子组件概念，后面会有详细的介绍。

- Sink：TiCDC 的 Sink 模块负责将 TiCDC 的数据变更输出到外部系统中。目前 TiCDC 支持输出到 MySQL、TiDB、Kafka、S3 等外部系统中。
- Table Sink：负责将 TiCDC 的数据变更按照表为单位进行聚合，然后输出到外部系统中。
- Event Sink：负责与外部系统进行交互，将 TiCDC 的数据变更输出到外部系统中。这里的 Event 主要指的是 TiCDC 的数据变更事件，比如 Insert、Update、Delete 等。
- MQ Event Sink：负责将 TiCDC 的数据变更输出到 Message Queue 中。MQ Sink 会将数据变更事件转换为 MQ 消息，然后输出到 MQ 中。目前 TiCDC 支持输出到 Kafka 中。
- Txn Event Sink：负责将 TiCDC 的数据变更按照事务为单位进行聚合，然后输出到外部系统中。目前 TiCDC 支持输出到 MySQL、TiDB 中。
- DDL Sink：负责将 TiCDC 的 DDL 语句输出到外部系统中。目前 TiCDC 支持输出到 Kafka、MySQL、TiDB 中。
- MQ DDL Sink：负责将 TiCDC 接受到的 DDL 语句输出到 Kafka 中。
- Txn DDL Sink：，负责将 TiCDC 接受到的 DDL 语句输出到 MySQL、TiDB 中。
- Blackhole Sink：它是一个空 Sink 实现，不会将数据变更输出到外部系统中。它主要用于测试和调试。

## 基本架构

我们可以将 TiCDC 接收到的数据分为两类：
- DML：TiCDC 接收到的数据变更事件，比如 Insert、Update、Delete 等。
- DDL：TiCDC 接收到的 DDL 语句。

Sink 模块也就根据不同的数据类型抽象出了不同的 Sink 子模块，分别是 Event Sink、DDL Sink。Event Sink 负责将过滤和聚合后的 DML 数据输出到外部系统中，DDL Sink 负责将过滤后的 DDL 数据输出到外部系统中。

DDL Sink 很容易理解，因为它就是简单的将收到每个 schema 和 table 的 DDL 语句输出到外部系统中。而 Event Sink 则更加复杂，我们会接收到大量不同表的变更数据，但是 TiCDC 需要按照表为单位进行数据同步。所以我们又引入了 Table Sink，它负责将收到的数据按照表进行聚合，然后输出到 Event Sink 中。

我们可以将 TiCDC 的 Sink 模块抽象为下面这个图：

[![](https://www.plantuml.com/plantuml/png/ROwn2eCm48RtFCNLiU0BT7BGhHP4XwvNUz3GorMysEUlab0IT3luVhxlAlKu-yMnEVaNEOA9qOeP6LLXw04LYW4VJD1RUHSHD04qNnAVWROBfErW3uVxBGfd5CNHsuzaCxC6psMvwI-mA2LIwoOcjqwtUEhBWOcehgyXjHvH_WV9ZmVqwCbabsgo-5RCBJjQaJmNnpy0)](https://www.plantuml.com/plantuml/uml/ROwn2eCm48RtFCNXMF05EZdeLWjYXgvNco4qkHOlwVCtaY0NT3luVhxlEWwuXkYTMXmoWvu16HgDJsTWjlHk2XWo67w6GWN6APDnWYFKZtiHhaYCIx0VgcOg1Izjk-cl4Da8gVLVPtoQRd7fgu4ggEzlEPME8j-1-F64dZqfTb8ZuhbOMOwqC_IAtlq1)

有了这个基本的架构，我们就可以看看数据是如何在各个子模块之间流动的了。

## 数据流程
数据同步流程也可以根据数据类型分为两部分，一部分是 DML 数据，另一部分是 DDL 数据。

### DML 数据
TiCDC 从 TiKV 接受到变更数据后，会对数据进行排序，但是整个排序过程中数据都是所有表的数据放在一起进行排序的。排序完成后我们还需要以表为单位进行分发，这个过程就是 Table Sink 负责的。所以其他组件跟 Sink 模块的交互就是通过调用 Table Sink 的接口来完成的。

这个过程中 Table Sink 可以理解成一个缓冲区，它会将收到的数据按照表进行缓存，但是并不会真实的将数据写入外部系统。与外部系统的交互是通过 Event Sink 来完成的，通过这样的抽象，多个 Table Sink 可以共享一个 Event Sink，我们可以在底层并发的进行数据写入。并且我们能共用一些公共的资源，比如数据库连接池，Kafka 的生产者等等。

下面是 DML 数据在 MQ Event Sink 模块中流转的时序图：

[![](https://www.plantuml.com/plantuml/png/VT2n2eCm40RWFKznTNSmeqEn5QSWL1Gwd_5AGsAK6FlyZQ6Kwj0bm_z_kOCh5e_EhwDX9_-aaM0sg2oRGwYacj5wwDeCS86amzuGjChgB3a0VW1y3-gcQgEe6wXUP7r4UoCY4FZG25StQN89Ozlgz1p_vo3H6BWxvIdEM4BB_xHRlDKYXvkRXbKI4v1-_QKKaSGexFbCACFJezJPRyaF9MS5sI4SxGq0)](https://www.plantuml.com/plantuml/uml/VT2n2eCm40RWFKznTNSmeqEn5QSWL1Gwd_5AGsAK6FlyZQ6Kwj0bm_z_kOCh5e_EhwDX9_-aaM0sg2oRGwYacj5wwDeCS86amzuGjChgB3a0VW1y3-gcQgEe6wXUP7r4UoCY4FZG25StQN89Ozlgz1p_vo3H6BWxvIdEM4BB_xHRlDKYXvkRXbKI4v1-_QKKaSGexFbCACFJezI_7Jzs1TaXdEmD)

在上图中 Table Sink1 和 Table Sink2 通过调用 Event Sink 的 `WriteEvents` 将数据异步的写入到 Kafka 中，在等到写入的消息收到 ACK 之后，再通过调用每个事件的 Callback 来通知 Table Sink 数据已经写入成功。**整个数据写入过程都是异步的，这样可以提高数据写入的吞吐量。**


### DDL 数据

TiCDC 从 TiKV 接受到 DDL 变更数据后，会将数据直接发送到具体的 DDL Sink 实现中，这个过程是同步的，也就是说 TiCDC 会等待 DDL Sink 返回成功之后才会继续处理后续的数据。在写入 DDL 的时候我们并没有使用 Table Sink，因为 DDL 数据是全局且顺序的，所以我们可以直接将数据写入到 DDL Sink 中。

下面是 DDL 数据在 MQ DDL Sink 模块中流转的时序图：

[![](https://www.plantuml.com/plantuml/png/LSungiCm40JGVayntxqluE9ZWZl54ECJBD862yah8Sj5Slg894swPUSDp7XKBlNS8_tLJNP1ZkoLSdjwwpDhnRnCqtK57-Zc1Ut6wZLqFyQyOyFtmBYK5AHqHDzY_mypm7ACM1zgjvBKFyNXLf8xhP11tyW73qY1Hb7N5hq0)](https://www.plantuml.com/plantuml/uml/LSungiCm40JGVayntxqluE9ZWZl54ECJBD862yah8Sj5Slg894swPUSDp7XKBlNS8_tLJNP1ZkoLSdjwwpDhnRnCqtK57-Zc1Ut6wZLqFyQyOyFtmBYK5AHqHDzY_mypm7ACM1zgjvBKFyNXLf8xhP11tyW73qY1Hb7N5hq0)

## Table Sink

## Event Sink

## DDL Sink

## 新建一个 Blackhole Sink

## 总结

## 参考资料


[sink 模块]: https://github.com/pingcap/tiflow/tree/master/cdc/sinkv2
