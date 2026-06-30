# Apache Kafka With Docker Compose

This project is a small Docker Compose setup for running Apache Kafka locally and understanding producers, consumers, topics, partitions, and offsets.

It starts:

- One Kafka broker in KRaft mode, so there is no ZooKeeper container.
- Kafka UI at <http://localhost:8080>.
- A topic setup container that creates the topic.
- A producer container that sends fake order events every 3 seconds.
- A consumer container that reads those events from Kafka.

This setup is intentionally simple and local. It is not a production Kafka setup.

## Requirements

- Docker Engine
- Docker Compose plugin, usually available as `docker compose`

## Files

```text
.
|-- docker-compose.yml
`-- README.md
```

## Start

Run:

```bash
docker compose up -d
```

Open Kafka UI:

```text
http://localhost:8080
```

You should see:

- Cluster: `local-kafka`
- Topic: `order-events`
- Producer container: `kafka-order-producer`
- Consumer container: `kafka-order-consumer`

The producer prints events like:

```text
Produced: order_id=1,user=user2,item=book,amount=101,status=CREATED
Produced: order_id=2,user=user3,item=book,amount=102,status=CREATED
```

The consumer prints the records it receives, including partition and offset.

## What This Shows

The producer writes events to Kafka:

```text
order-producer -> Kafka topic: order-events
```

The consumer reads events from Kafka:

```text
Kafka topic: order-events -> order-consumer
```

Kafka sits between systems. Producers do not need to know which consumers exist, and consumers do not need to call producers directly.

## Useful Commands

List running containers:

```bash
docker compose ps
```

List topics:

```bash
docker compose exec kafka /opt/kafka/bin/kafka-topics.sh \
  --bootstrap-server localhost:9092 \
  --list
```

Describe the topic:

```bash
docker compose exec kafka /opt/kafka/bin/kafka-topics.sh \
  --bootstrap-server localhost:9092 \
  --describe \
  --topic order-events
```

You should see that `order-events` has 3 partitions.

Manually produce messages:

```bash
docker compose exec kafka /opt/kafka/bin/kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic order-events
```

Then type messages and press Enter after each one:

```text
user clicked login
user searched for kafka
user completed payment
```

Press `Ctrl+C` to stop the manual producer.

Manually consume messages from the beginning:

```bash
docker compose exec kafka /opt/kafka/bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic order-events \
  --from-beginning \
  --property print.partition=true \
  --property print.offset=true
```

Consume as a named consumer group:

```bash
docker compose exec kafka /opt/kafka/bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic order-events \
  --group order-group \
  --from-beginning \
  --property print.partition=true \
  --property print.offset=true
```

Stop everything:

```bash
docker compose down
```

Stop everything and remove Kafka data:

```bash
docker compose down -v
```

## What Kafka Is

Kafka is an event streaming platform.

An event is a record that says something happened.

Examples:

- A user logged in.
- An order was created.
- A payment was completed.
- A package location changed.
- A sensor sent a temperature reading.
- A video was played, paused, or skipped.

Instead of one application directly calling many other applications, Kafka acts as a durable event log in the middle.

```text
Producer applications -> Kafka topics -> Consumer applications
```

## Core Kafka Terms

Producer:

An application that writes events into Kafka. In this setup, `order-producer` writes fake order events.

Consumer:

An application that reads events from Kafka. In this setup, `order-consumer` reads the order events.

Topic:

A named stream of related events. In this setup, the topic is `order-events`.

Broker:

A Kafka server. A production Kafka cluster normally has multiple brokers. This setup uses one broker to keep the local environment small.

Partition:

A topic is split into partitions. Partitions let Kafka scale because different consumers can read from different partitions. This setup creates `order-events` with 3 partitions.

Offset:

The position number of a record inside a partition. Kafka identifies a record by topic, partition, and offset.

Consumer group:

A set of consumers that work together. If a topic has 3 partitions and a consumer group has 3 active consumers, Kafka can assign one partition to each consumer.

## How Kafka Works Internally

Kafka stores records in append-only logs.

Append-only means new records are added to the end of a log. Old records are not updated in place.

Example partition log:

```text
partition 0
offset 0: order created
offset 1: payment completed
offset 2: package shipped
```

Kafka does not immediately delete a message after one consumer reads it. Records stay in Kafka for a configured retention period.

That means many different consumers can read the same event independently.

For example, one `PaymentCompleted` event can be read by:

- Billing service
- Fraud detection service
- Email service
- Analytics pipeline

Each consumer group tracks its own offsets. A slow analytics consumer does not stop a billing consumer from reading the same topic.

This is a major difference from a simple queue:

- In many queues, a message is consumed once and removed.
- In Kafka, events are retained for a period of time and can be replayed.

## Simple Real-Life Analogy

Imagine a company notice board.

Managers do not personally call every employee for every update. They post notices on the board.

Employees read the board when they are available. Different employees may care about different notices. Someone who missed an earlier update can still look back at older notices.

Kafka is similar, but for software systems. Producers post events. Consumers read the events they care about.

## Simple Software Example

An online store receives an order.

Without Kafka:

```text
Order Service directly calls Inventory Service
Order Service directly calls Payment Service
Order Service directly calls Email Service
Order Service directly calls Analytics Service
```

If one service is slow or unavailable, the order flow can become fragile.

With Kafka:

```text
Order Service produces: OrderCreated

Kafka topic: orders

Inventory Service consumes OrderCreated
Payment Service consumes OrderCreated
Email Service consumes OrderCreated
Analytics Service consumes OrderCreated
```

The order service only publishes the event. Other services react independently.

## Real-Life Use Cases

Real-time analytics:

Companies can track clicks, searches, orders, payments, and errors as they happen.

Microservice communication:

Services publish events instead of directly calling many other services.

Log aggregation:

Applications send logs to Kafka, and consumers move those logs into search, monitoring, alerting, or long-term storage systems.

Fraud detection:

Payment events can stream into fraud detection systems quickly, allowing suspicious activity to be checked in near real time.

IoT:

Sensors can continuously publish readings. Consumers can process alerts, dashboards, and storage.

## Companies Using Kafka

LinkedIn:

Kafka was originally created at LinkedIn. LinkedIn Engineering describes Kafka as a major part of its data infrastructure for activity tracking, messaging, metrics, and stream processing.

Source: <https://engineering.linkedin.com/teams/data/data-infrastructure/streams/kafka>

Netflix:

Netflix has written about using Kafka in its data movement and stream processing platforms. In its Data Mesh architecture, Kafka is used as part of the transport layer between processing systems.

Source: <https://netflixtechblog.com/data-mesh-a-data-movement-and-processing-platform-netflix-1288bcab2873>

Apache Kafka project:

The Apache Kafka project says Kafka is used by thousands of companies and lists many organizations on its Powered By page.

Source: <https://kafka.apache.org/powered-by/>

## Walkthrough

1. Run `docker compose up`.
2. Open <http://localhost:8080>.
3. Show the `order-events` topic in Kafka UI.
4. Show `kafka-order-producer` producing order events.
5. Show `kafka-order-consumer` consuming the same events.
6. Point out partition and offset in the consumer output.
7. Stop the consumer and let the producer continue.
8. Start a manual consumer with `--from-beginning` to show replay.
9. Run two manual consumers with the same `--group` to explain consumer groups.
10. Describe the topic and point out the 3 partitions.

## Important Points

- Kafka is not just a message queue. It is a distributed commit log.
- Producers write events to topics.
- Consumers read events from topics.
- Kafka stores events for a configured retention period.
- Consumers pull data at their own speed.
- Different consumer groups can read the same events independently.
- Partitions give Kafka scalability.
- Ordering is guaranteed only inside a single partition, not across the whole topic.
