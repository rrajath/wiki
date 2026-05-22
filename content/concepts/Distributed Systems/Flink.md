---
title: Flink
type: concept
created: 2026-05-12
updated: 2026-05-12
sources:
  - raw/notes/20250425115216-flink.org
tags: [distributed-systems, stream-processing, flink, real-time]
---

# Apache Flink

Distributed stream processing engine. Provides exactly-once guarantees with high throughput and low latency. Built around a **dataflow graph** (DAG).

## When to use

Many problems don't need Flink. Ask: *Do I actually need real-time, sub-second latency?* If batch processing (Spark, Hadoop) is sufficient, use that. If you only need simple Kafka message transformation, a lightweight consumer service may be enough.

Use Flink when:
- State must be maintained across events (e.g., count clicks per user in the last 5 minutes).
- Exactly-once processing is required.
- Complex windowing over time is needed.

## Core concepts

### Dataflow graph

DAG of **operators** (nodes) connected by **streams** (edges).
- **Sources**: read from Kafka, Kinesis, file systems, custom.
- **Sinks**: write to databases, data warehouses (Snowflake, BigQuery), message queues, file systems.
- **Operators**: stateful transformations — map, filter, reduce, window, join, flatMap, aggregate.
- **Streams**: unbounded sequences of data elements (infinite arrays).

### State

Operators can maintain state across events. Flink manages state internally and checkpoints it for fault tolerance.

State types:
- *Value state*: single value per key.
- *List state*: list per key.
- *Map state*: map per key.
- *Aggregating/Reducing state*: incremental aggregations.

### Watermarks

A watermark is a timestamp flowing through the system alongside data. It declares: "all events with timestamps before this watermark have arrived."

**Purpose**: determine when to trigger window computations; handle late-arriving events gracefully.

**Strategies**:
- *Unbounded out-of-orderness*: wait up to a configured delay for late events.
- *No watermarks*: process events immediately as they arrive.

Configured at the source.

### Windows

Group stream items by time or count. Types:
- *Tumbling window*: fixed-size, non-overlapping.
- *Sliding window*: fixed-size, overlapping.
- *Session window*: dynamic-size based on activity gaps.
- *Global window*: custom windowing logic.

Windows work with watermarks to determine when to trigger and how to handle late events. Allowed lateness can be set to process events arriving after a window closes but within a grace period.

## Example (Kafka → Redis pipeline)

```java
DataStream<ClickEvent> clickstream = env
    .addSource(new FlinkKafkaConsumer<>("clicks", schema, kafkaProps));

DataStream<PageViewCount> pageViews = clickstream
    .keyBy(click -> click.getPageId())
    .window(TumblingProcessingTimeWindows.of(Time.minutes(1)))
    .aggregate(new CountAggregator());

pageViews.addSink(new RedisSink<>(redisConfig, new PageViewCountMapper()));
```

## Interview considerations

- Significant operational overhead: deployment, monitoring, scaling of the Flink cluster.
- State growth and recovery must be planned.
- Exactly-once comes with performance overhead — justify whether it's needed.
- Window choice dramatically impacts accuracy and resource usage — justify your window type.
- For simple transformations, a Kafka consumer service is likely sufficient.
