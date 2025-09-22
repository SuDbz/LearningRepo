# Kafka 

## Table of contents
 - [About](#name)
 - [Terminologies in Kafkas](#terminologies)
 - [Configuration example](#code)
 - [References and notes](#references)
    



## About <a name = "about"></a>
Apache Kafka is an open-source distributed streaming platform designed to handle real-time data feeds at scale. It's essentially a messaging system that can also store and process streams of records.

- **Kafka as a Pub-Sub System**
   - **Pub-Sub model**: Publishers (producers) send messages to topics, and subscribers (consumers) receive those messages.
   - **Kafka's suitability**: Kafka's partitioning, replication, and fault tolerance make it well-suited for pub-sub scenarios, especially those involving large-scale distributed systems.

- **Kafka as a Message Queue**
   - **Message queue model**: Messages are stored in a queue and processed sequentially.
   - **Kafka's suitability**: Kafka can be used as a message queue by configuring topics with appropriate partitioning and retention policies. This allows for reliable and asynchronous message delivery.


## Terminologies in Kafka<a name = "terminologies"></a>

#### 1. Core Components

- **Broker** :  A server node that stores and replicates data.
- **Topic** :   A logical grouping of messages.
- **Partition** : A subdivision of a topic, allowing for horizontal scaling and parallelism.
- **Producer** : An application that sends messages to a topic.
- **Consumer** : An application that reads messages from a topic.
- **Consumer Group** :  A group of consumers that share a subscription to a topic

#### 2. Key Concepts

- **Message**: A unit of data that is sent and received over Kafka.
- **Offset**: A unique identifier for a message within a partition.
- **Partition Key**: A property of a message used to determine its partition.
- **Replication Factor**: The number of replicas of a partition across brokers.
- **Durability**: The guarantee that messages will not be lost.
- **Exactly-Once Delivery**: A guarantee that a message will be processed exactly once.
- **At-Least-Once Delivery**: A guarantee that a message will be processed at least once.
- **ZooKeeper**: A distributed coordination service used by Kafka for configuration management and naming

#### 3. In-deepth 
  - [A to Z ](/kafka/atoz/kafka_a_z.md)


## Kafka Configuration / Code Sample <a name="code" ></a>

#### 1. Create a kafka topic 
```java
// Kafka broker connection properties
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092"); // Replace with your broker's hostname and port

        // Create an AdminClient instance
        AdminClient adminClient = AdminClient.create(props);

        // Define a new topic with desired configuration
        NewTopic newTopic = new NewTopic(
                "my-topic",
                3, // Number of partitions
                1 // Replication factor
        );

        // Create the topic
        try {
            adminClient.createTopics(Collections.singleton(newTopic)).get();
            System.out.println("Topic created successfully!");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            adminClient.close();
        }
```
#### 2.  Configure a Kafka Producer

```java 
Properties producerProps = new Properties();
producerProps.put("bootstrap.servers", "localhost:9092");
KafkaProducer<String, String> producer = new KafkaProducer<>(producerProps);
ProducerRecord<String, String> record = new ProducerRecord<>("my-topic",   
 "Hello, world!");
producer.send(record);
```

#### 3. Configure a Kafka Consumer 
```java 
Properties consumerProps = new Properties();
consumerProps.put("bootstrap.servers", "localhost:9092");
consumerProps.put("group.id", "my-group");   

KafkaConsumer<String, String> consumer = new KafkaConsumer<>(consumerProps);
consumer.subscribe(Collections.singletonList("my-topic"));   

ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
for (ConsumerRecord<String, String> record   
 : records) {
    System.out.printf("Offset: %d, Key: %s, Value: %s%n", record.offset(), record.key(), record.value());   

}
consumer.commitSync();
```


### Kafka references <a name="references" ></a>
- [code](/kafka/code/help.md)
- [A-to-Z](/kafka/atoz/kafka_a_z.md)
- [Docker](/kafka/atoz/run_kafka_docker.md)
- [Consumer off-set](https://www.youtube.com/watch?v=KOu6DVdaY24) 
- [Message queue (kmq)](https://www.youtube.com/watch?v=-As92HV0O4E)
- [Kafka Topics, Partitions and Offsets Explained](https://www.youtube.com/watch?v=_q1IjK5jjyU)
- [Out of Order Consumer Offset Commit in Kafka](https://www.youtube.com/watch?v=cfj3vlkvE4Q)
- [Cluster behavior](/kafka/kafka-cluster-behavior.md)
- [Java Concurrency Interview Question: Multi-threaded Message Queue like Kafka, SQS, RabbitMQ](https://www.youtube.com/watch?v=4BEzgPlLKTo)
