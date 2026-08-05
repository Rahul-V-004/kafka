# Kafka Hands-on Demo Using Docker

This guide demonstrates the basic working of Kafka using Docker.

Architecture:

```
+-----------------+
| Kafka Broker    |
|  (Container)    |
+--------+--------+
         |
         | Topic: greetings
         |
+--------+--------+
| Producer         |
| (Terminal 1)     |
+------------------+

+------------------+
| Consumer         |
| (Terminal 2)     |
+------------------+
```

---

# Step 1: Create docker-compose.yml

Create a file named `docker-compose-test.yml`

```yaml
version: '3.8'

services:
  kafka:
    image: apache/kafka:4.0.0
    container_name: kafka
    ports:
      - "9092:9092"

    environment:
      KAFKA_NODE_ID: 1
      KAFKA_PROCESS_ROLES: broker,controller
      KAFKA_LISTENERS: PLAINTEXT://:9092,CONTROLLER://:9093
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@localhost:9093
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
```

---

# Step 2: Start Kafka

```bash
docker compose -f docker-compose-test.yml up -d
```

Verify the container is running.

```bash
docker ps
```

Expected output:

```
CONTAINER ID   IMAGE               NAME
xxxxxxxxxxxx   apache/kafka:4.0.0  kafka
```

---

# Step 3: Enter Kafka Container

```bash
docker exec -it kafka bash
```

---

# Step 4: Create a Topic

```bash
/opt/kafka/bin/kafka-topics.sh \
--create \
--topic greetings \
--bootstrap-server localhost:9092
```

---

# Step 5: Verify Topic

```bash
/opt/kafka/bin/kafka-topics.sh \
--list \
--bootstrap-server localhost:9092
```

Expected output

```
greetings
```

---

# Step 6: Start Producer (Terminal 1)

Open **Terminal 1**

```bash
docker exec -it kafka bash
```

Start the producer.

```bash
/opt/kafka/bin/kafka-console-producer.sh \
--topic greetings \
--bootstrap-server localhost:9092
```

You'll see

```
>
```

Now type

```
Hi
```

Press **Enter**

Again you'll see

```
>
```

This **does NOT mean an error**.

It means Kafka has accepted the message and is waiting for the next message.

Send more messages if you want.

```
Hello
Kafka
How are you?
```

---

# Step 7: Start Consumer (Terminal 2)

Open another terminal.

```bash
docker exec -it kafka bash
```

Run

```bash
/opt/kafka/bin/kafka-console-consumer.sh \
--topic greetings \
--bootstrap-server localhost:9092 \
--from-beginning
```

Output

```
Hi
Hello
Kafka
How are you?
```

The consumer prints every message stored in the topic.

---

# Step 8: Live Messaging

Keep both terminals open.

### Producer

```
>Apache Kafka
```

### Consumer immediately receives

```
Apache Kafka
```

Producer

```
>Docker Demo
```

Consumer

```
Docker Demo
```

---

# Step 9: Create Another Consumer

Open Terminal 3

```bash
docker exec -it kafka bash
```

Run

```bash
/opt/kafka/bin/kafka-console-consumer.sh \
--topic greetings \
--bootstrap-server localhost:9092 \
--group group1
```

Open Terminal 4

```bash
docker exec -it kafka bash
```

Run

```bash
/opt/kafka/bin/kafka-console-consumer.sh \
--topic greetings \
--bootstrap-server localhost:9092 \
--group group1
```

Now produce

```
Message 1
Message 2
Message 3
```

Only **one** consumer in **group1** receives each message.

Kafka distributes messages among consumers in the same group.

---

# Step 10: Different Consumer Group

Open another terminal.

```bash
docker exec -it kafka bash
```

Run

```bash
/opt/kafka/bin/kafka-console-consumer.sh \
--topic greetings \
--bootstrap-server localhost:9092 \
--group group2
```

Now produce

```
Testing Consumer Groups
```

Result

- One consumer from **group1** receives the message.
- The consumer in **group2** also receives the message.

Different consumer groups each receive their own copy of every message.

---

# Useful Commands

### List Topics

```bash
/opt/kafka/bin/kafka-topics.sh \
--list \
--bootstrap-server localhost:9092
```

---

### Describe Topic

```bash
/opt/kafka/bin/kafka-topics.sh \
--describe \
--topic greetings \
--bootstrap-server localhost:9092
```

---

### Delete Topic

```bash
/opt/kafka/bin/kafka-topics.sh \
--delete \
--topic greetings \
--bootstrap-server localhost:9092
```

---

### Exit Producer or Consumer

```
Ctrl + C
```

---

### Stop Kafka

```bash
docker compose down
```

---

# Kafka Workflow

```
                Producer
                    |
                    | "Hi"
                    v
          +-------------------+
          |   Kafka Broker    |
          +-------------------+
                    |
                    |
              Topic: greetings
                    |
                    v
                Consumer
```

---

# Interview Explanation (30 Seconds)

> Kafka is a distributed event streaming platform. A producer publishes messages to a topic on the Kafka broker. The broker stores the messages durably on disk. Consumers subscribe to the topic and pull messages independently. Consumer groups allow multiple consumers to share the workload, enabling Kafka to scale horizontally while ensuring fault tolerance and high throughput.

---

# Key Concepts

| Component | Description |
|-----------|-------------|
| Producer | Sends messages to Kafka |
| Broker | Kafka server that stores messages |
| Topic | Logical channel where messages are stored |
| Partition | Division of a topic for parallel processing |
| Consumer | Reads messages from Kafka |
| Consumer Group | Group of consumers that share message processing |
| Offset | Position of a message within a partition |
