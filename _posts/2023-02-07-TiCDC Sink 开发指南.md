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
- Event Sink：负责与外部系统进行交互，将 TiCDC 的数据变更编码后输出到外部系统中。这里的 Event 主要指的是 TiCDC 的数据变更事件，比如 Insert、Update、Delete 等。
- MQ Event Sink：负责将 TiCDC 的数据变更输出到 Message Queue 中。MQ Sink 会将数据变更事件编码为 MQ 消息，然后输出到 MQ 中。目前 TiCDC 支持输出到 Kafka 中。
- Txn Event Sink：负责将 TiCDC 的数据变更按照事务为单位进行聚合，然后输出到外部系统中。目前 TiCDC 支持输出到 MySQL、TiDB 中。
- DDL Sink：负责将 TiCDC 的 DDL 语句输出到外部系统中。目前 TiCDC 支持输出到 Kafka、MySQL、TiDB 中。
- MQ DDL Sink：负责将 TiCDC 接受到的 DDL 语句输出到 Kafka 中。
- Txn DDL Sink：，负责将 TiCDC 接受到的 DDL 语句输出到 MySQL、TiDB 中。

## 基本架构

我们可以将 TiCDC 接收到的数据分为两类：
- DML：TiCDC 接收到的数据变更事件，比如 Insert、Update、Delete 等。
- DDL：TiCDC 接收到的 DDL 语句。

Sink 模块也根据上述不同的数据类型抽象出了不同的 Sink 子模块，分别是 Event Sink、DDL Sink。Event Sink 负责将过滤和聚合后的 DML 数据输出到外部系统中，DDL Sink 负责将过滤后的 DDL 数据输出到外部系统中。

DDL Sink 很容易理解，因为它就是简单的将收到每张表的 DDL 语句编码后输出到外部系统。而 Event Sink 则更加复杂，我们会接收到大量不同表的变更数据，但是 TiCDC 需要按照表为单位进行数据同步。所以我们又引入了 Table Sink，它负责将收到的数据按照表进行聚合，然后输出到 Event Sink 中。

我们可以将 TiCDC 的 Sink 模块抽象为下面这个图：

[![](https://www.plantuml.com/plantuml/png/ROwn3e8m48RtFiN9Q08FG1mOO1CJ0eFhYPVOs5P2MoSVtjCsX0Qsotr_ll-lhCFPUQt4mJr84qmAfH6ZGcjXw04jP0FU544lpJEBe0cWUPDn2MYxGDeEjd2uNg9mHcDnTF9bafZWmcEU__GbU4j2y7Nw5CNVMuBKaoBD-UNFoXJ4ghe-Xoe-edm1guqxT6_aAYVu3DrHbRGdSBEj8dFtMdq1)](https://www.plantuml.com/plantuml/uml/ROwn3e8m48RtFiN9Q08FG1mOO1CJ0eFhYPVOs5P2MoSVtjCsX0Qsotr_ll-lhCFPUQt4mJr84qmAfH6ZGcjXw04jP0FU544lpJEBe0cWUPDn2MYxGDeEjd2uNg9mHcDnTF9bafZWmcEU__GbU4j2y7Nw5CNVMuBKaoBD-UNFoXJ4ghe-Xoe-edm1guqxT6_aAYVu3DrHbRGdSBEj8dFtMdq1)

有了这个基本的架构，我们就可以看看数据是如何在各个子模块之间流动的了。

## 数据流程
数据同步流程也可以根据数据类型分为两部分，一部分是 DML 数据，另一部分是 DDL 数据。

### DML 数据
TiCDC 从 TiKV 接受到变更数据后，会对数据进行排序，但是整个排序过程中数据都是所有表的数据放在一起进行排序的。排序完成后我们还需要以表为单位将数据进行分发，这个过程就是 Table Sink 负责的。所以其他组件跟 Sink 模块的交互都是通过调用 Table Sink 的接口来完成的。

这个过程中 Table Sink 可以理解成一个缓冲区，它会将收到的数据按照表进行缓存，但是并不会真实的将数据写入外部系统。与外部系统的交互是通过 Event Sink 来完成的，通过这样的抽象，多个 Table Sink 可以共享一个 Event Sink，我们可以在底层并发的进行数据写入。并且我们能共用一些公共的资源，比如数据库连接池，Kafka 的生产者等等。

下面是 DML 数据在 MQ Event Sink 模块中流转的时序图：

[![](https://www.plantuml.com/plantuml/png/VT2n2eCm40RWFKznTNSmeqEn5QSWL1Gwd_5AGsAK6FlyZQ6Kwj0bm_z_kOCh5e_EhwDX9_-aaM0sg2oRGwYacj5wwDeCS86amzuGjChgB3a0VW1y3-gcQgEe6wXUP7r4UoCY4FZG25StQN89Ozlgz1p_vo3H6BWxvIdEM4BB_xHRlDKYXvkRXbKI4v1-_QKKaSGexFbCACFJezJPRyaF9MS5sI4SxGq0)](https://www.plantuml.com/plantuml/uml/VT2n2eCm40RWFKznTNSmeqEn5QSWL1Gwd_5AGsAK6FlyZQ6Kwj0bm_z_kOCh5e_EhwDX9_-aaM0sg2oRGwYacj5wwDeCS86amzuGjChgB3a0VW1y3-gcQgEe6wXUP7r4UoCY4FZG25StQN89Ozlgz1p_vo3H6BWxvIdEM4BB_xHRlDKYXvkRXbKI4v1-_QKKaSGexFbCACFJezI_7Jzs1TaXdEmD)

在上图中 Table Sink1 和 Table Sink2 通过调用 Event Sink 的 `WriteEvents` 将数据异步的写入到 Kafka 中，在等到写入的消息收到 ACK 之后，再通过调用每个事件的 Callback 来通知 Table Sink 数据已经写入成功。**整个数据写入过程都是异步的，这样可以提高数据写入的吞吐量。**


### DDL 数据

TiCDC 从 TiKV 接受到 DDL 变更数据后，会将数据直接发送到具体的 DDL Sink 实现中，这个过程是同步的，也就是说 TiCDC 会等待 DDL Sink 返回成功之后才会继续处理后续的 DDL 变更数据。在写入 DDL 的时候我们并没有使用 Table Sink，因为 DDL 数据是全局共用且有序的，所以我们可以直接将数据发送到 DDL Sink 中。

下面是 DDL 数据在 MQ DDL Sink 模块中流转的时序图：

[![](https://www.plantuml.com/plantuml/png/LSungiCm40JGVayntxqluE9ZWZl54ECJBD862yah8Sj5Slg894swPUSDp7XKBlNS8_tLJNP1ZkoLSdjwwpDhnRnCqtK57-Zc1Ut6wZLqFyQyOyFtmBYK5AHqHDzY_mypm7ACM1zgjvBKFyNXLf8xhP11tyW73qY1Hb7N5hq0)](https://www.plantuml.com/plantuml/uml/LSungiCm40JGVayntxqluE9ZWZl54ECJBD862yah8Sj5Slg894swPUSDp7XKBlNS8_tLJNP1ZkoLSdjwwpDhnRnCqtK57-Zc1Ut6wZLqFyQyOyFtmBYK5AHqHDzY_mypm7ACM1zgjvBKFyNXLf8xhP11tyW73qY1Hb7N5hq0)

## Table Sink

在 Sink 模块中，Table Sink 是一个抽象的接口：

```golang
// 用于将数据以表为单位进行缓存。
type TableSink interface {
	// AppendRowChangedEvents 将行变更事件追加到 Table Sink 中。
	// 注意：此方法不是线程安全的，请不要并发调用。
	AppendRowChangedEvents(rows ...*model.RowChangedEvent)
	// UpdateResolvedTs 将聚合完成的数据发送到 Event Sink 中。
	// 注意：此方法是异步的且不是线程安全的。
	UpdateResolvedTs(resolvedTs model.ResolvedTs) error
	// GetCheckpointTs 返回 Table Sink 中的 CheckpointTs。
	// 注意：此方法是线程安全的。
	GetCheckpointTs() model.ResolvedTs
	// Close 关闭 Table Sink 并释放资源。
	// 注意：我们需要保证这个方法是可取消或者中断的。
	Close(ctx context.Context)
}
```

它最重要的两个方法是 `AppendRowChangedEvents` 和 `UpdateResolvedTs`，前者用于将行变更事件追加到 Table Sink 的缓存中，后者用于将聚合完的数据发送到 Event Sink 中。在 TiCDC 中，我们实现了一个基于内存的 Table Sink，它会将行变更事件聚合到内存中。

因为针对的外部系统的不同，我们需要实现不同的聚合策略。所以我们为 Table Sink 的实现添加了一个范型参数 `E`，用于指定聚合策略：

```golang
type EventTableSink[E eventsink.TableEvent] struct {
	...
	// 就是具体的 Event Sink，比如 MQ Event Sink。
	backendSink     eventsink.EventSink[E]
	...
	// 用于实现不同的聚合策略。
	eventAppender   eventsink.Appender[E]
	// 注意：数据是按照 CommitTs 排序的。
	eventBuffer []E
	...
}
```

可以看到范型参数 `E` 的类型为 `eventsink.TableEvent`，它是一个非常简单的接口：

```golang
type TableEvent interface {
	// GetCommitTs 返回事件的 CommitTs。
	GetCommitTs() uint64
}
```

通过这个接口抽象，任何可以获取 CommitTs 的事件都可以作为 Table Sink 的缓存对象。在 TiCDC 中，我们实现了两种聚合策略：

- `RowChangedEvent`：用于单行变更，比如 MQ Event Sink 就是将行变更一条一条发送到 Kafka 中。
- `SingleTableTxn`：用于单表事务，比如 Txn Event Sink 就是以事务为单位提交到 MySQL 中。

还记得我们在数据流程中提到 DML 数据的写入是异步的吗？所以我们需要为每个 Event 添加一个 Callback，用于在数据写入完成后通知 Table Sink：

```golang
type CallbackFunc func()

type CallbackableEvent[E TableEvent] struct {
	Event     E
	Callback  CallbackFunc
	...
}

// RowChangeCallbackableEvent 是行变更事件，它可以被回调。
type RowChangeCallbackableEvent = CallbackableEvent[*model.RowChangedEvent]

// TxnCallbackableEvent 是单表事务事件，它可以被回调。
type TxnCallbackableEvent = CallbackableEvent[*model.SingleTableTxn]
```

有了这两种不同的可以回调的 Event，我们就可以实现具体的聚合策略了。为了复用代码，我们将聚合策略抽象为一个接口：

```golang
type Appender[E TableEvent] interface {
	// Append 添加一批行变更事件到缓存中。
	Append(buffer []E, rows ...*model.RowChangedEvent) []E
}
```

**这样我们只需要实现 `Append` 方法，就可以实现不同的聚合策略了。**

对于 `RowChangeCallbackableEvent` 来说，我们并没有实际上的聚合操作，只是将行变更事件**顺序**追加到当前的缓存中：

```golang
func (r *RowChangeEventAppender) Append(
	buffer []*model.RowChangedEvent,
	rows ...*model.RowChangedEvent,
) []*model.RowChangedEvent {
	return append(buffer, rows...)
}
```

对于 `TxnCallbackableEvent` 来说，我们需要将行变更事件聚合到一个事务中：

```golang
func (t *TxnEventAppender) Append(
	buffer []*model.SingleTableTxn,
	rows ...*model.RowChangedEvent,
) []*model.SingleTableTxn {
	for _, row := range rows {
		// 这意味我们目前还没有任何事务，所以我们需要创建一个新的事务。
		if len(buffer) == 0 {
			txn := &model.SingleTableTxn{
				StartTs:   row.StartTs,
				CommitTs:  row.CommitTs,
				Table:     row.Table,
				TableInfo: row.TableInfo,
			}
			txn.Append(row)
			buffer = append(buffer, txn)
			continue
		}

		lastTxn := buffer[len(buffer)-1]

		lastCommitTs := lastTxn.GetCommitTs()
		...
		// 使用 StartTs 来判断是否是同一个事务。如果不是同一个事务，我们需要创建一个新的事务。
		if row.SplitTxn || lastTxn.StartTs != row.StartTs {
			buffer = append(buffer, &model.SingleTableTxn{
				StartTs:   row.StartTs,
				CommitTs:  row.CommitTs,
				Table:     row.Table,
				TableInfo: row.TableInfo,
			})
		}

		buffer[len(buffer)-1].Append(row)
	}

	return buffer
}
```

在上面的代码中，我们将 StartTs 作为界限，将行变更事件聚合到一个事务中。**在 TiDB 中两个事务的 CommitTs 可能相同，但是 StartTs 一定不同，所以我们可以通过 StartTs 来判断行变更是否属于同一个事务。**

有了不同聚合策略的 `Appender`，我们就可以实现具体的 Table Sink 了。首先是 `AppendRowChangedEvents` 方法，得益于 `Appender` 的抽象，我们的具体实现可以非常简单：

```golang
func (e *EventTableSink[E]) AppendRowChangedEvents(rows ...*model.RowChangedEvent) {
	e.eventBuffer = e.eventAppender.Append(e.eventBuffer, rows...)
}
```

其次是 `UpdateResolvedTs` 方法，它会将当前的缓存中的所有 Event 写入到具体的 Event Sink 中：

```golang
func (e *EventTableSink[E]) UpdateResolvedTs(resolvedTs model.ResolvedTs) error {
	...
	// 从缓存中找到第一个大于 resolvedTs 的数据。
	i := sort.Search(len(e.eventBuffer), func(i int) bool {
		return e.eventBuffer[i].GetCommitTs() > resolvedTs.Ts
	})
	...
	// 将该数据之前的所有数据取出来。
	resolvedEvents := e.eventBuffer[:i]

	...

	// 为每一个取出来的数据创建一个 CallbackableEvent。
	resolvedCallbackableEvents := make([]*eventsink.CallbackableEvent[E], 0, len(resolvedEvents))
	for _, ev := range resolvedEvents {
		ce := &eventsink.CallbackableEvent[E]{
			Event:     ev,
			Callback:  e.progressTracker.addEvent(),
			SinkState: &e.state,
		}
		resolvedCallbackableEvents = append(resolvedCallbackableEvents, ce)
	}

	...

	// 将取出来的数据写入到下游。
	return e.backendSink.WriteEvents(resolvedCallbackableEvents...)
}
```

可以看到在 `UpdateResolvedTs` 方法中，我们将缓存中的数据以 `ResolvedTs` 为界限取出来，然后将其转换为 `CallbackableEvent`，最后将其写入到具体的 backendSink 中。**需要注意的是，我们能这样做的前提是数据在之前的模块中已经经过了排序，这样我们才能保证数据的顺序性。**

以上就是 Table Sink 的核心实现，只要实现不同的 `Appender`，我们就可以实现不同的 Table Sink。目前 TiCDC 主要支持的就是上述的两种聚合策略，你可以根据自己的需求来选择不同的聚合策略。**一般来说，你不需要自己实现 Table Sink，目前的两种聚合策略已经能满足大部分的需求。**

## Event Sink

## DDL Sink

## 新建一个 Blackhole Sink

## 总结

## 参考资料


[sink 模块]: https://github.com/pingcap/tiflow/tree/master/cdc/sinkv2
