# SMACK Learning and Certification Resources

SMACK Stack is the collection of five BigData tools, including Spark, Mesos, Akka, Cassandra and Kafka (all are open source tools). This term was first introduced in 2015 when a group of programmers meet at a conference with the participation of Mesosphere – an US tech company based in San Francisco, developing software for data center based on Apache Mesos- and Mesosphere is also a company that has major contribution to popularize SMACK stack [1].

The aim of SMACK stack is to create an ideal enviroment with necessary tools and features for handling Big data (apart from batch-processsing, SMACK Stack has more focus on real-time processing). At present, SMACK stack is widely used in many Big data processing model, especially data streaming processing.

## Five components of SMACK Stack are:

#### Spark 
 
 Spark is an unified analytics engine developed for big data processing. It can be consider as the next version of Hadoop that improve the processing speed and address some shortcoming of Hadoop. In SMACK Stack, Spark is the main component used to process and analyze the collected data

#### Mesos
 
 Mesos is a platform that other components of SMACK Stack can run on it. Mesos is installed on Clusters to manage and share resources between nodes and distribute works to each node to utilize all available resources of a distributed system. In other words, we can consider Mesos as an Operating System for Distributed System.

Mesos cluster consists of Master nodes and Slave nodes. Master node provide information about available resources of Cluster to the Framework and distribute works to Slave nodes. On the other hand, Slave nodes is responsible for completing assigned works. Normally, the operation of Mesos can be described with 4 steps below:

1. Slave sends information about its available resources to Master (Eg: Slave 1 has 4 CPU and 16 GB RAM can be used)
2. Master sends information about resources that cluster can provide to Frameworks
3. Scheduler of Framework responds to Master with the tasks to be processed (Ex: Task 1 needs 1CPU, 3GB RAM; Task 2 needs2CPU, 8GB RAM)
4. Master sends tasks to Slave and Slave uses its resources to complete the tasks. If Slave still has resources leftover, other Frameworks can use these resources to execute their tasks.

#### Akka
 Akka is an open source tool developed based on Actor Model to help developer build concurrent and distributed applications. In SMACK Stack, Akka is used to create Actors and each Actor carries out a specific task. Actors interact with each other using messages
 
#### Cassandra
 
Cassandra is a distributed NoSQL database to store a large amount of data in several nodes.

#### Kafka
 
Kafka is an open source distributed streaming platform based on Publish – Subscribe model. Producer publishes stream data to Kafka topics and Consumer subscribes to these topic to receive and process data produced to them. In SMACK Stack, Kafka is used to temporarily store data from various sources and then distribute to other components of the system 

``**Note**: Each component of SMACK Stack can be replaced by a similar tool. For example, Apache Spark can be replaced by Apache Flink or we can replace Mesos by Hadoop Yarn,…``

For more details: [Click Here](https://www.hpe.com/us/en/insights/articles/understanding-the-smack-stack-for-big-data-1803.html)

## Lambda Architecture

Lambda architecture is a data processing architecture introduced by Nathan Marz. It takes the advantages of both batch processing and stream-processing to handle a large amount of data effectively. Lambda architecture consists of 3 layers: Batch layer, Speed layer, and Serving layer.

![Image](https://ibb.co/M9mSDMY)

Batch layer acts like a ‘Data lake’ to store all collected data and process this data with batch processing (we need to pre-define the batch interval, which is the time between two batch processing, such as 30 mins, 3 hours or 1 day…). The advantage of using Batch processing in Lambda Architecture is because the collected data can be duplicated or contains unnecessary information. Therefore, we need an intermediate step to preprocess and clean the raw data. Batch processing can also address the late arrival problem caused by transmission disruption or being collected much later than the time of posting. In addition, the result of each Batch processing is updated frequently, thereby improving the accuracy of the system and fixing the incorrect result of the previous Real-time processing.

Speed layer is responsible for processing data in real time, thereby accomplishing the Batch layer as Batch layer has long latency and thus unable to process newly recieved data. However, the result of Speed layer is usually not as good as Batch layer due to limited processing time. At this layer, we can use a number of open source tools such as Apache Storm, Apache Spark or Apache Flume,…

Serving layer is responsible for storing outputs of Batch layer and Speed layer, therefore we can use database tools for this layer such as Apache Cassandra, MongoDB or ElasticSearch,…

The way Lambda Architecture work can be summarized into following five steps

1. All collected data is passed to both Batch layer and Speed layer
2. Batch layer carries out two works: storing data and processing data to produce batch views
3. Serving layer indexes batch views for quick access
4. Speed layer only processes recent data in real-time and produce real-time views
5. The system returns the result of users’ queries by combining both batch views and real-time views.

For more detail on Lambda Architecture: [Click Here](http://lambda-architecture.net/)


