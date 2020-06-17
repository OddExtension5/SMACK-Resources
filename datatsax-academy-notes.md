## DS101: Introduction to Apache Cassandra

Relational Overview, Cassandra Overview, Choosing a Distribution, 

### Introduction to Relational Overview

#### Medium Data

+ Fits on 1 machine
+ RDBMS is fine
    + PostgreSQL
    + MySQL
+ Supports hundreds of concurrent users
+ ACID makes us feel good
+ Scales vertically

#### Third Normal Form Doesn't Scale

+ Queries are unpredicatble
+ Users are impatient
+ Data must be denormalized
+ If data ? memory, you = history
+ Disk seeks are the worst

#### Sharding is a Nightmare

+ Data is all over the place
+ No more joins
+ No more aggregations
+ Denormalize all the things
+ Querying secondary indexes requires hiting every shard
+ Adding shards requires manually moving data
+ Schema changes



****************************************** THE END ***************************************************************

## DS201: DataStax Enterpise 6 Foundation of Apache Cassandra

CQL, Partitions, Clustering Columns, Application Connectivity, Node, Ring, Peer to Peer, Vnodes, Gossip, Snitch, Replication, Consistency,Hinted Handoff, Read Repair, Node Sync, Write Path, Read Path, Compaction, Advance Performance

### Clustering Columns

#### Querying Clustering Columns

+ You must first provide a partition key
+ Clustering columns can follow thereafter
+ You can perform either equality (=) or range queries (<, >) on clustering columns
+ All equality comparisons must come before inequality comparisons
+ Since data is sorted on disk, range searches are binary search followed by a linear read

#### Changing Default Ordering

+ Clustering columns default ascending order
+ Change ordering direction via **WITH CLUSTERING ORDER BY**
+ Must include all columns including and up to the columns you wish to order descending
+ For example, we exclude **id** below and assume ASC

```python
CREATE TABLE users (
    state text,
    city text,
    name text,
    id uuid,
    PRIMARY KEY((state), city, name, id)
    ) WITH CLUSTERING ORDER BY(city DESC, name ASC);
```

#### Allow Filtering

+ ALLOW FILTERING relaxes the querying on partition key constraint
+ You can then query on just clustering columns
+ Causes Apache Cassandra to scan all partitions in the table
+ Don't use it
    + Unless you really have to
    + Best on small data


### Application Connectivity

#### Drivers

+ Drivers easily connect your application to your Apache Cassandra database
+ Driver languages:
   + Java, Python, C#, C++ etc.,
   + www.academy.datastax.com/short-courses

#### Drivers API

+ API is similar between languages
+ Policies are the same (load balancing, rebalancing etc.,)

```python
//Python
cluster = Cluster()
session = cluster.connect('killrvideo')
result = session.execute("<query>")[0]
```
```java
//Java
Cluster cluster = Cluster.builder().addContactPoint("127.0.0.1").build();
Session session = cluster.connect("killrvideo");
Result results = session.execute("<query>");
```
```ruby
//Ruby
cluster = cassandra.cluster
session = cluster.connect('killrvideo')
session.execute("<query>");
```
    
### Node

+ 6K-12K transactions/second/core
+ 2-4 Terabytes

#### Nodetool

+ Node management
+ located in the **bin/** folder
  + help: Lists all possible sub commands
  + info: Current node settings and stats
  + status: Reports basic node helath information
+ Many more...

### Ring

+ -2^{63} to +2^{63}-1
+ Murmur3

#### Joining the Cluster

+ nodes join the cluster by communication with any node
+ Apache Cassandra finds these seed nodes list of possible nodes in **cassandra.yaml**
+ Seed nodes communicate cluster topology to the joining node
+ Once the new node joins the cluster, all nodes are peers
+ Four states to each node: JOINING/LEAVING/UP/DOWN

#### Drivers

+ Drivers intelligently choose which node would best coordinate a request
+ Per-query basis:

ResultSet results = session.execute("<query>");

+ ```TokenAwarePolicy```: driver chooses node which contains the data
+ ```RoundRobinPolicy```: driver round robins the ring
+ ```DCAwareRoundRobinPolicy```: driver round robins the largest data center

### Vnodes

#### VNode Details

+ Adding/removing nodes with vnodes helps keeps the cluster balanced
+ By default, each node has 128 vnodes
+ VNodes automate token range assignemnt

#### Configuration

+ Configure vnode settings in cassandra.yaml
+ num_tokens
+ Value greater than one turns on vnodes

### Gossip

#### Choosing a Gossip Node

+ Each node initiates a gossip round every second
+ Picks one to three nodes to gossip with
+ Nodes can gossip with ANY other node in the cluster
+ Probabilistically (slightly favor) seed and downed nodes
+ Nodes do not track which nodes they gossiped with prior
+ Reliably and efficiently spreads node metadata through the cluster
+ Fault tolerant-continues to spread when nodes fail\

SeedNode:  https://www.quora.com/What-is-a-seed-node-in-Apache-Cassandra

#### What to Gossip - Cluster Metadata

```yaml
Endpoint State
  Heartbeat State:
      generation=5
      version=22
  Application State:
      STATUS=NORMAL
      DC=west
      RACK=rack1
      SCHEMA=c2edb
      LOAD=100.0
      so on.......
```

SYN:-  End:Gen:Ver {127.0.0.1:100:15, 127.0.0.3:50:15, etc.,}

{node} --SYN--> {other node}

#### Network Traffic

+ Constant rate (trickle) of network traffic
+ Minimal compared to data streaming, hints
+ Doesn't cause network spikes

### Snitch

+ Determines/declares each node's rack and data center
+ The "topology" of the cluster
+ Several different types of snitches
+ Configured in cassandra.yaml

**endpoint_snitch: SimpleSnitch**

+ There are two main groups of snitches
    
   |    Regular                    |   Cloud Based         |
   |-------------------------------|-----------------------|
   | SImpleSnitch                  | Ec2Snitch             |
   | PropertyFileSnitch            | Ec2MultiRegionSnitch  |
   | GossipingPropertyFileSnitch   | GoogleCloudSnitch     |
   | DynamicSnitch                 | CloudstackSnitch      |
   
 
 #### Simple Snitch
 
 + Places all nodes in the same data center and rack
 + Default snitch
 
 ```java
 public class SimpleSnitch extends AbstractEndpointSnitch {
    public String getRack(InetAddress endpoint) {
        return "rack1";
     }
     
     public String getDatacenter(InetAddress endpoint) {
        return "datacenter1";
     }
 }
 ```
 
 #### Property File Snitch
 
 + Reads datacenter and rack information for all nodes from a file
 + You must keep files in sync with all nodes in the cluster
 + ```cassandra-topology.properties``` file
 
 ```
 175.56.12.105=DC1:RAC1
 175.50.13.200=DC1:RAC1
 175.54.35.197=DC1:RAC2
 175.54.35.152=DC1:RAC2
 
 120.53.24.101=DC2:RAC1
 120.55.16.200=DC2:RAC1
 120.57.18.103=DC2:RAC2
 120.57.18.177=DC2:RAC2
 ```
 
 #### Gossiping Property File Snitch
 
 + Relieves the pain of the property file snitch
 + Declare the current node's DC/rank information in a file
 + You must set each individual node's settings
 + But you don't have to copy settings as with property file snitch
 + Gossip spreads the settings through the cluster
 + ``cassandra-rackdc.properties`` file
 
 ```
 dc=DC1
 rack=RAC1
 ```
 
 #### Rack Inferring Snitch
 
+ Infers the rack and DC from the IP address

#### Dynamic Snitch

+ Layered on top of your actual snitch
+ Maintains a pulse on each node's performance
+ Determines which node to query replicas from depending on node health
+ Turned on by default for all snitches
+ http://www.datastax.com/dev/blog/dynamic-snitching-in-cassandra-past-present-and-future
 
#### Cloud-Based Snitches
 
+ Ec2Snitch
  + Single region Amazon EC2 deployment
+ Ec2MultiRegionSnitch
  + Multi-region Amazon EC2 deployment
+ GoogleCloudSnitch
  + Multi-region Google cloud deployment
+ Cloudstack Snitch
  + For Cloudstack environments
  

#### Configuring Snitches

+ All nodes in the cluster must use the same snitch
+ Changing cluster newtwork topology requires restarting all nodes
+ Run sequential repair and cleanup on each node

### Replication
RF=3

```python
CREATE KEYSPACE killrvideo
WITH REPLICATION = {
    'clsss': 'NetworkTopologyStrategy'
    'dc-west':2, 
    'dc-east':3
}
```

### Consistency

+ Cassandra: AP database and tunable consistency
+ CL = ONE, QUORUM, ALL

#### Consistency Settings

Weakest to strongest


| Setting                 | Description                                                  |
|-------------------------|--------------------------------------------------------------|
| ANY                     | Storing a hint at minimum is satisfactory                    |
| ONE, TWO, THREE         | Checks closest nodes(s) to coordinator                       |
| QUORUM                  | Majortiy vote, (sum_of_replication_factors / 2) + 1          |
| LOCAL_ONE               | Closest node to coordinator in same data center              |
| LOCAL_QUORUM            | Closest quorum of nodes in same data center                  |
| EACH_QUORUM             | Quorum of nodes in each data center, applies to writes only  |
| ALL                     | Every node must participate                                  |


+ The higher the consistency, the less chance you may get stable data
  + Pay for this with latency
  + Depends on your situational needs
  

### Hinted Handoff
Depending on your consistancy level, you can still service write requests even when nodes are down. Apache Cassandra accomplishes this via hinted handoff

#### Settings

+ cassandra.yaml
+ You can disable hinted handoff
+ Choose directory to store hints file
+ Set the amount of time a node will store a hint
+ Default is three hours

#### Consistency Level ANY - Beware...

+ Consistency level of ANY means storing a hint suffices
+ Consistency level of ONE or more means at least one replica must successfully write
+ Hint does not suffice

### Read Repair

Read repair is one of the mechanisms that ensures that data in your Apache Cassandra cluster stays in sync

#### Anti-Entropy Operations

+ Network partitions cause nodes to get out of the sync
+ You must choose between availability vs. consistency level
+ CAP Theorem

#### Read Repair Chance

+ Performed when read is at consistency level less than ALL
+ Request reads only a subset of replicas
+ We can't be sure replicas are in sync
+ Generally you are safe, but no guarantees
+ Response sent immediately when consistency level is met
+ Read repair done asynchronously in the background
+ ``dclocal_read_repair_chance`` set to 0.1 (10%) by default
   + Read repair that is confined to the same datacenter as the coordinator node
+ ``read_repair_chance`` set 0 by default
   + For a read repair across all datacenters with replicas 
   
#### Nodetool Repair

+ Syncs all data in the cluster
+ Expensive
    + Grows with amount of data in cluster
+ Use with clusters servicing high writes/deletes
+ Last line of defense
+ RUn to synchronize a failed node coming back online
+ RUn on nodes not read from very often

### Node Sync
New feature in DataStax Enterprize 6

#### Full Repairs

+ Full repairs bog down the system
+ Bigger the cluster and dataset, the worse the time
+ In times past, we recommended running full repair within ``gc_grace_seconds``

#### Node Sync

+ Runs in the background continuously repairing your data
  + Quiet hum vs. everybody stop what you're doing
+ Better to repair in small chunks as we go rather than full repair
+ Automatic enabled by default
    + By you must enable it per table 

#### Details

+ Each node runs NodeSync
+ NodeSync continuously validates and repairs data
+ Enables on per-table basis
  + Default is disables
  
```python
CREATE TABLE myTable (...) WITH nodesync = { 'enables': 'true'};
```
#### Save Points

+ Each node splits its local range into segments
    + Small token range of a table
+ Each segment makes a save point
    + NodeSync repairs a segment
    + Then NodeSync saves its progress
    + Repeat
+ NodeSync priorities segments to meet the deadline target

#### Segments Sizes

+ Determining toekn range in a given segment is a simple recursive split
+ Target is each segment is less than 200MB
    + Configurable, but good by default, ``segment_size_target_bytes``
    + Greater than partition
    + So partitions greater than 200MB win over segemnts less than 200MB
+ Algorithm doesn't calculate data size but instead assumes acceptable distribution of data among your cluster

#### Segment Failures

+ Nodes validates/repair segments as a whole
+ If node fails during segment validation, node drops all work for that segment and starts over
+ Node records successful segment validations in the ``system_distributed.nodesync_status`` table

#### Segment Outcomes

+ ``full_in_sync``: All replicas were in sync
+ ``full_repaired``: Some repair necessary
+ ``partial_in_sync``: not all replicas responded (at least 2 did), but all respondent were in sync
+ ``partial_repaired``: not all replica responded (at least 2 did), with some repair needed
+ ``uncompleted``: one node available/responded; no validation occurred
+ ``failed``:unexpected error happened; check logs

#### Segment Validation

+ NodeSync simply performs a read repair on the segment
+ Read data from all replicas
+ Check for inconsistencies
+ Repair stale nodes

### Read Path

BloomFilter ---> KeyCache --> Partition Summary --> Partition Index ---> SSTable

#### DataStax Enterprize 6.0 Read Path Optimizations

+ No Partition Summary
+ Partition Index changed to a trie-based data structure
+ SSTable looksups in this format scream
+ Huge Performance improvments; especially for large SSTables

+ Migrating from OSS Apache Cassandra is seamless
+ Datastax Enterprize can tell what kind of SSTable format it's working with
+ As old SSTable are compacted, DataStax Enterpize writes them out in the new format

### Compaction

+ A process Apache Cassandra uses to remove all the stale data from the pre-existing SSTables
+ Compact two SSTables into new SSTable with latest data

#### Compaction Strategies

+ Compaction strategies are configurable. These strategies include:
    + ``Size Tiered Compaction``: (Default) triggers when multiple SSTables of a similar size are present
    + ``Leveled Compaction``: groups SSTables into levels, each of which has fixed size limit which is 10 times larger than the previous level
    + ``TimeWindow Compaction``: creates time window buckets of SStables that are compacted with each other using the SIze Tiered Compaction Strategy.
    
+ Use the ``ALTER TABLE`` command to change the strategy

```python
ALTER TABLE mykeyspace.mytable WITH compaction = {
    'class': 'LeveledCompactionStrategy' ":
```

*************************************************** THE END ****************************************************************

## DS210: DataStax Enterprise 6 Operations with Apache cassandra

Configuring Clusters, Cluster Sizing, Cassandra-STress, Linux Top Command, Linux dstat command, nodetool for Performance Analysis, System & Output Logs, JVM Garbage and Collection Logging, Adding/Removing Nodes, Bootstrapping, Replacing a Down Node,
Leveled Compaction, Sized Tiered Compaction, Time Window Compaction, Repair, Nodesync, sstablesplit, Multi-DC Concepts, CQL Copy, sstabledump, sstableloader, SPark for Data Loading, DSE DSBulk, Backup FUndamentals, JVM Settings, Garbage COllections, Heap Dump, Tuning The Kernel, Hardware Selections, Cloud, Security Considerations, OpsCenter and Lifecycle








**************************************** THE END *****************************************************************

## DS220: DataStax Enterprize 6 Practical Application Data Modeling with Apache Cassandra

Data Modeling Overview, Relational Vs. Cassandra, Cassandra Table, Partition ANd Storage Structure, Clustering Columns, Denormalization, Table Features, Collections, UDTs, Counters, UDFs and UDAs, Conceptual Data Modelings, Application Workflow And Access Patterns, Logical Data Modeling, Analysis And Validation, Write Techniques, Read Techniques, Table/Key Optimization, Data Model Migration, Data Model-Anti Patterns, Data Modeling Use Cases


### Data Modeling

#### What is Data Modeling?

+ Analyze requirements of the domain
+ Identify entities and relationships ---- Conceptual Data Model
+ Identify queries --- Workflow and Access Patterns
+ Specify the schema --- Logical Data Model
+ Get something working with CQL --- Physical Data Model
+ Optimize and tune

#### Data Modeling Methodology

| Conceptual Data Model & Application Workflow | --> | Mapping Conceptual to Logical | --> | Logical Data Model | ---> | Physical Data Model | ---> | Optimization Tuning |


### Relational Vs. Apache Cassandra

Simple difference Overview

|  Relational Database                   |  Apache Cassandra                |
|----------------------------------------|----------------------------------|
| Sample relational model methodology    | Cassandra modeling methodology   |
| Data --> Model --> Application         | Application --> Model --> Data   |
| Entities are king                      | Queries are king                 |
| Primary Key for uniqueness             | Primary keys are much more       |
| Often have single point of failure     | Distributed architecture         |
| ACID Complaint                         | CAP Theorem                      |
| Joins and indexes                      | Denormalization                  |
| Referential Integrity enforces         | RI not enforced                  |

#### Relational Data Modeling Methodology

Sample Methodology

CDM --> Mapping --> Relational LDM --> Normalization ---> Normalized Relational and Queries---> Optimization ---> Relational PDM

CMD: Conceptual Data Model <br/>
LDM: Logical Data Model <br/>
PDM: Physical Data Model

#### Transactions and ACID Compilance
Relational databases are ACID compilant

|   Term          |                         Definition                                                                          |
|-----------------|-------------------------------------------------------------------------------------------------------------|
|   Atomicity     | All statements in a transaction succeed (commit), or none of them do (rollback)                             |
|   Consistency   | Transactions can't leave the DB in an inconsistent state.The new DB state must satisfy integrity constraints|
|   Isolation     | Transactions do not interfere with each other                                                               |
|   Durability    | Completed transactions persist in the event of subsequent failure                                           |

#### Transactions in Apache Cassandra

Cassandra does not support ACID transactions

+ ACID causes a significant performance penalty
+ Not required for many use cases
+ However, a single Cassandra write operations demonstrates ACID properties
    + INSERTs, UPDATEs, and DELETEs are atomic, isloated and durable
    + Tunable consistency for data replicated to nodes, but does not handle application integrity constraints
    
#### Apache Cassandra and CAP Theorem
Consistency, Availability and Partition Tolerance

+ By default, Cassandra is an AP database
+ However, this is tunable with consistency level
+ By tuning consistency level, you can make it more CP than AP.
+ However! Cassandra isn't designed to be CA because you can't sacrifice partition tolerance

#### Referential Integrity -- Apache Cassandra
Apache Cassandra does NOT enforce referential integrity

+ Due to performance reasons -- would require a read before a write
+ Not an issue that has to be fixed on the Apache Cassandra side
+ Referential integirty can be enforced in an application design -- more work for developers

OR

+ Run DSE Analytics with Apache Spark to validate that duplicate data is consistent

### Enough CQL to get you started

#### KEYSPACE
Creating your container

+ Top level namespace/container
+ Similar to a relational database schema

```python
CREATE KEYSPACE killrvideo
    WITH REPLICATION = {
        'class': 'SimpleStrategy'
        'replication_factor': 1
     };
```
+ Replication parameters required

#### USE
Accessing your keyspace

+ USE switches between keyspaces
```pthon
USE killrvideo;
```
#### Tables

+ Keyspaces contain tables and tables contain data

```python
CREATE TABLE table1 (
    column1 TEXT,
    column2 TEXT,
    column3 INT,
    PRIMARY KEY (column1)
);

CREATE TABLE users (
    user_ud UUID,
    first_name TEXT,
    last_name TEXT,
    PRIMARY KEY (user_id)
);
```
#### Basic Data Types

+ Text
    + UTF8 encoded string
    + varchar is same as text
+ INT
    + signed
    + 32 bits
    
+ UUID (Universally Unique Identifier)
    + ex: 53b12dj3-12dn-3je3-2j3n-2j3ndj42kns
    + Generate vis ``uuid()``
 
+ TIMEUUID embeds a TIMETSAMP value
    + Ex: 123jdn42-3d43-23d4-12m3-12nd4k5new32
    + Sortable
    + Generate via ``now()``
 
+ TIMESTAMP
    + stores date and time
    + 64-bit integer
    + Milliseconds since Jan 1, 1970 at 00:00:00 GMT
    + Displayed in cqlsh as yyy-mm-dd HH:mm:ssZ
    + As literal in cqlsh is '1979-07-24 08:30:15'
    
+ Blob
    + Arbitrary bytes (no validation), expressed as hexadecimal

+ Boolean
    + Stored internally as true or false

+ Counter
    + 64-bit signed integer-only one counter column is allowed per table

+ Inet
    + IP address string in IPv4 or IPv6 format
    

#### COPY
Works with CSV files

+ Imports/exports CSV 
```python
COPY table1 (column1, column2, column3) FROM 'tabledata.csv';
```
+ Header parameter skips the first line in the file
```python
COPY table1 (column1, column2, column3) FROM 'tabledata.csv'
WITH HEADER=true;
```
#### SELECT
Pulls data from a table

```python
SELECT * FROM table1;

SELECT column1, column2, column3 FROM table1;

SELECT COUNT(*) FROM table1;

SELECT * FROM table1 LIMIT 10;
```

#### TRUNCATE
Removing table data

+ Deletes all data from a table immediately and irreversibly
+ Truncate sends a JMX command to all nodes to delete SSTables that hold the data
+ If any of these nodes are down the command will fail

```python
TRUNCATE table1;
```

#### ALTER TABLE

+ Can change datatype of column, add columns, drop columns, rename columns and change table properties
+ BUT! You cannot alter the ``PRIMARY KEY columns``
```python
ALTER TABLE table1 ADD another_column
text;

ALTER TABLE table1 DROP another_column;
```
#### SOURCE
Executes CQL Scripts

+ Execute a file containing CQL statements
+ Enclose file name in single quotes
+ Output for each statement appears in turn
    + Example:

```python
SOURCE './myscript.cql';
```

### Fundamentals of an APache Cassandra Table

#### Terminology
Terms and definitions to get your head around

+ **Data model**:
    + An abstract model for organizing elements of data
    + In Apache Cassandra this is based on the queries you want to perform
    
+ **Keyspace**:
    + SImilar to relational schema-- outermost grouping of data
    + All tables live inside a keyspace
    + Keyspace is the container for replication
    
+ **Table**:
    + Grouped into keyspace
    + Contain columns
    
+ **Partition**:
    + Row(s) of data that are stored on a particular node in your based on a partitioning strategy

+ **Row**:
    + One or more CQL rows stored together on a partition
 
+ **Column**
    + Similar to a column in a relational database
    + **Primary key**:
    + Used to access the data in a table and guwarantees uniqueness
 
+ **Partition Key**:
    + Defines the node on which the data is stored
 
+ **Clustering Column**:
     + Defines the order of rows within a partition
     
### Collections
Group and store data together in a column

+ Collection columns are multi-values columns
+ Designed to store a small amount of data
+ Retireved in its entirely
+ Cannot nest a collection inside another collection -- unless you use FROZEN

#### SET
Creating and inserting with SET

+ types collection of unique values
+ Stored unordered, but retrieved in sorted ordered

```python
CREATE TABLE users (
    id TEXT PRIMARY KEY,
    fname TEXT,
    lname TEXT,
    emails set<text>
);

INSERT INTO users (id, fname, lname, emails)
    VALUES('cass123', 'Cassandra', 'Dev', {'cass@dev.com', 'cassd@gmail.net'});
    
```
#### LIST
Altering a table and updating using LIST

+ Like SET -- collection of values in same cell
+ Do not need to be unique and can be duplicated
+ Stored in a particular order

```python

ALTER TABLE users ADD freq)des list<text>;

UPDATE users SET freq_dest = ['Berlin', 'London', 'Paris'] WHERE id = 'cass123';
```
#### MAP
Creating and inserting with MAP

+ Types collection of key-value pairs -- name and pair of typed values
+ Ordered by unique keys

```python
ALTER TABLE users ADD todo map<tmestamp, text>;

UPDATE users SET todo = {'2018-1-1': 'create database', '2018-1-2':'load data and test', '2018-2-1': 'move to production' }
WHERE id = 'cass123';
```

#### Using ``FROZEN`` in a Collection

+ If you want to nest datatypes, you have to use ``FROZEN``
+ Using ``FROZEN`` in a collection will serialize multiple components into a single value
+ Values in a ``FROZEN`` collection are treated like blobs
+ Non-frozen types allow updates to individual fields

### User Defined Types (UDTs)
Attach multiple data fields to a column

+ User-defined types group related fields of information
+ User-defined types (UDTs) can attach multiple data fields, each named and types, to a single column
+ Can be any datatype including collections and other UDTs
+ Allows embedding more complex data within a single column

```python
CREATE TYPE address (
    street text, 
    city text,
    zip_code int,
    phones set<text>
);

CREATE TYPE full_name (
    first_name text,
    last_name text
 );
 
 CREATE TABLE users (
    id uuid,
    name frozen <full_name>,
    direct_reports set<frozen <full_name>>,
    addresses map<text, frozen <address>>,
    PRIMARY KEY ((id))
 );
```

### Counter

+ Column used to store a 64-bit signed integer
+ Changed incrementally -- incremented or decremented
+ Values are changed using UPDATE
+ Need specially dedicated tables -- can only have primary key and counter columns

 ```python
 CREATE TABLE moo_counts (
    cow_name TEXT,
    moo_count counter,
    PRIMARY KEY ((cow_name))
);

UPDATE moo_count
SET moo_count = moo_count + 8
WHERE cow_name = 'Betsy';
```

Some things to be aware of

+ Distributed system can cause consistency issues with counters in some cases
+ Cannot INSERT or assign values -- default values is "0"
+ Must be only non-primary key columns(s)
+ Not idempotent
+ Must use UPDATE command -- DataStax Enterprise rejects USING TIMESTAMP or USING TTL to update counter columns
+ Counter columns cannot be indexed or deleted

### UDFs and UDAs

#### User Defined Functions (UDFs)
Creating UDFs

+ Write custom functions using JAVA and JavaScript
+ Use in SELECT , INSERT, and UPDATE statements
+ Function are only available within the keyspace where it is defined

+ Enables by changing the following settings in the ``cassandra.yaml`` file
    + Java: Set ``enable_user_defined_functions`` to true
    + Javascript and other custom languages: Set ``enables_scripted_user_defined_functions`` to true
    
#### User Defined Aggregates (UDAs)

+ Datastax Enterprize allows users to define aggregate functions
+ Functions are applied to data stored in a table as part of query result
+ The aggregate function must be created prior to its use in a SELECT statement
+ Query must only include the aggregate function istdelf -- no additional columns

```python
CREATE OR REPLACE
    FUNCTION avgState ( state tuple<int, float> , val float)
    CALLED ON NULL INPUT
    RETURNS tuple<int, float>
    LANGUAGE java
    AS 'if ( val != null) {
        state.setInt(0, state.getInt(0) + 1);
        state.setFloat(1, state.getFloat(1)+val.floatValue());
    }
    return state;';
    
CREATE OR REPLACE
    FUNCTION avgFinal ( state tuple<int>, float> )
    CALLED ON NULL INPUT
    RETURNS float
    LANGUAGE java
    AS 'float r = 0;
    if (state.getInt(0) == 0) return null;
        r = state.getFLoat(1);
        r /= state.getInt(0);
     return Float.valueOf(r);';
     
CREATE AGGREGATE
    IF NOT EXISTS average ( float )
    SFUNC avgState
    STYPE tuple<int, float>
    FINALFUNC avgFinal
    INITCOND (0,0);
```
#### Querying with a UDF and UDA

+ The state function is called once for each row
+ The value returned by the state function becomes the new state
+ After all rows are processed, the optional final function is executed with the last state value as its arguments
+ Aggregation is performed by the coordinator

```python
SELECT average(avg_rating) FROM videos WHERE release_year = 2002 ALLOW FILTERING;
SELECT average(avg_rating) FROM videos WHERE title = 'Planet of the Apes' ALLOW FILTERING;
SELECT average(avg_rating) FROM videos WHERE mpaa_raing = 'G' ALLOW FILTERING;
SELECT average(avg_rating) FROM videos WHERE genres contains 'Romance' ALLOW FILTERING;
```

### Conceptual Data Modeling
Modelling your domain

+ Abstract view of your domain
+ Technology independent
+ Not specific to any database system

#### Purpose of Conceptual Modeling

+ Understand your data
+ Essential objects
+ Constraints




















********************************************** THE END ************************************************

## DS330: DataStax Enterprise 6 Graph
