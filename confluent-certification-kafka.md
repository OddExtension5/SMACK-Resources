# Confluent Certification for Apache Kafka
  Strategies for successfull in Confluent Certification exam

## Confluent Certified Administrator for Apache Kafka

+ 40 MCQs in 90 minutes
+ Desgined to validate professional with a minimum of 6-12 months of Confluent experience
+ Remotely proctored on your computer
+ Availability globally in English

### Test Domain

1. Kafka Fundamentals  **15%**
2. Managing, configuring, and optimizing a cluster for performance **30%**
3. Kafka Security **15%**
4. Designing, Trouubleshooting, and integrating systems **40**

## Confluent Certified Developer for Apache Kafka

+ 60 MCQs in 90 mins
+ Desgined to validate professional with a minimum of 6-12 months of Confluent expereince
+ Remotely proctored on your computer
+ Availability globally in English

### Test Domain

1. Application Design **40%**
2. Development **30%**
3. Deployment/Testing/Monitoring **30%**

## Scope

+ Confluent's certification exam questions may cover:
  + Apache Kafka
  + Other Apache ecosystem projects that are either required, or likely to be used, within a production Kafka environment
  + Confluent Community License components. Example are:
     + ksqlDB
     + Confluent Schema Registry
     + Kafka REST proxy
  
+ Confluent's certification exam questions will NOT cover features or components that are exclusively available under either Confluent Enterprise License, or exclusively part of Confluent Cloud

## Does the Certification Expire?

**Yes**, it expires after two years

## Required Knowledge Domains

### Kafka Fundamentals (ALL EXAMS)

+ Distributed Systems -- Scalability, Fault Tolerance, HA
+ Primary functions of: Producer, Consumer, Broker
+ Meaning of "immutable" log
+ Meaning of "committed"
+ Topics, Partitions
+ Essentials services of Apache Zookeeper
+ Replication, Leaders, FOllowers
+ Kafka Messages, structure, make-up, metadata
+ Kafka Controller
+ Exaclty Once Semantics

### Managing, COnfiguring, and Optimizing for Performance (Administrator)

+ Startup sequence; component dependencies
+ How many partitions? Tradeoffs
+ Scalability factors
+ Sources and tools for monitoring; DIsplay of metrics
+ InSyncReplicas (ISR); Fully and Under replicated, and offline
+ Consumer lag, Under/Over Consumption
+ Broke failure, detection, and recovery
+ Batching and its impacts/consequences
+ Determining and solving data imbalance across brokers
+ Impacts of average and maximum message sizes

### Kafka Security (Administrator and Developer)

+ Authentication and Authorization (meanings and methods)
+ In-flight encryption - where and how
+ At rest encryption - strategies
+ SST/TLS keystores and truststores
+ Authentication and Authorization troubleshooting
+ Access Control Lists (ACLs) - where and how used
  + Use of wildcards
  
### Designing, Troubleshooting, and Integrating Systems (Administrator)

+ Brokers and Zookeeper
  + CPU, RAM, network, storage considerations
  + Number of nodes
+ Rack awarness
+ Kafka Connect
  + SOurce and Sink Connectiors
  + Scalability and High Availability
+ Business Continuity / DR
+ Data retention

### Application Design (Developer)

+ Kafka's command line tools
+ Pub/Sub and Streaming
+ Overall Apache Kafka architecture and design
+ Apache Kafka APIs, configuartion and metrics
+ Message metadata
+ Systems metrics
+ Kafka message key selection (choices and factors)
+ Message schema management

### Application Developement (Developer)

+ Kafka CLients: Producer and COnsumer, concepts and functions
+ Clinet troubleshooting/debugging
+ Performance, throughput, latency, scaling
+ Message order and delivery guarantees
+ Serialization/Deserialization
+ Producer partition selection
+ Consumer offset management
+ Consumer Groups, parition assigments, parition rebalances
+ Data retention, strategies and implications
+ Topic co-partitioning

### Deployment, Testing and Monitoring (Developer)

+ Application deployment choices
+ Securing your data
+ monitoring and troubleshooting clients
+ Client tuning
+ Kafka Strams featurs and use cases
  + Essentials component parts of Kafka Streams Applications
+ Confluent KSQL/ksqlDB features and use cases
  + Essentials elements of KSQL/ksqlDB environment
  

## CCAAK : Sample Questions

Test Domain - Kafka Fundamentals
1. How is message ordering defined in Kafka?

a. Ordering is guaranteed across all topics in a Kafka cluster
b. Ordering is guaranteed for all messages in a single Topic from a single producer
c. Ordering is guaranteed for all messages with the same key delivered by all Producers wiritng to that Partition
d. Ordering is guaranteed for all messages with the same key delivered by a single Producer  [**Correct**]
e. There are no message ordering guarantees in Kafka

2. When first setting up a Kafka broker, which of the parameters below need to be unique to that broker in configuration file?

a. Log-dirs
b. Port
c. Broker.id [**correct**]
d. All of the above
