# Learning Path : Apache Cassandra 3.x Administrator Associate Certification

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
+ If data > memory, you = history
+ Disk seeks are the worst

#### Sharding is a Nightmare

+ Data is all over the place
+ No more joins
+ No more aggregations
+ Denormalize all the things
+ Querying secondary indexes requires hiting every shard
+ Adding shards requires manually moving data
+ Schema changes

#### High Availability.. not really

+ Master failover.. who's responsible?
+ Multi-DC is a mess
+ Downtime is frequent
`   + Change database settings (innodb buffer )
    + Driver, power supply failures
    + OS updates

#### Summary of Failure

+ Scaling is a pain
+ ACID is naive at best
+ Re-sharding is a manual process
+ We're going to denormalize for performance
+ High availability is complicated, requires additional operational overhead

### Cassandra Overview

#### What is Apache Cassandra?

+ Fast Distributed Database
+ High Availability
+ Linear Scalability
+ Predictable Performance
+ No SPOF
+ Multi-DC
+ Commodity Hardware
+ Easy to manage operationally
+ Not a drop in replacement for RDBMS

#### Hash Ring

+ No master / slave / replica sets
+ No config servers, zookerper
+ Data is partitioned around the ring
+ Data is replicated to RF=N servers
+ All nodes hold data and can answer queries (both reads & writes)
+ Location of data on ring is determined by partition key

#### CAP Tradeoffs

+ Impossible to be both consistent and highly available during a network partition
+ Latency between data centers also makes consistency impractical
+ Cassandra chooses Availability & Partition Tolerance over Consistency

#### Replication

+ Data is replicated automatically
+ You pick number of servers
+ Called "replication factor" or RF
+ Data is ALWAYS replicated to each replica
+ If a machine is down, missing data is replayed via hinted handoff

#### Consistency Levels

+ Per query consistency
+ ALL, QUORUM, ONE
+ How many replicas for query to respond OK

#### Multi-DC

+ Typical usage: clients write to local DC, replicates async to other DCs
+ Replication factor per keyspace per datacenter
+ Datacenters can be physical or logical

### Choosing a Distribution

#### Write Path
+ Writes are written to any node in the cluster (coordinator)
+ Writes are written to commit log, then to memtable
+ Every write includes a timestamp
+ Memtable flushed to disk periodically (SSTable)
+ New memtable is created in memory
+ Deleted are a special write case, called a "tombstone"

#### What is an SSTable?
+ Immutable data file for row storage
+ Every wirte includes a timetsamp of when it was written
+ Partition is spread across multiple SSTables
+ Same column can be in multiple SSTables
+ Merged through compaction, only latest timestamp is kept
+ Deleted are written as tombstones
+ Easy backups!

#### Read Path
+ Any server may be queried, it acts as the coordinator
+ Contacts nodes with the requested key
+ On each node, data is pulled from SSTables and merged
+ Consistency < ALL performs read repair in background (read_repair_chance)


### CQL Fundamentals
  + Cassandra Query Language (CQL)
  + Mostly similar to SQL
  
#### Keyspaces
  + Top-level namespaces/container
  + Similar to a relational database schema
  
 CREATE KEYSPACE killrvideo
 WITH REPLICATION = {
  'class': 'SimpleStrategy'
  'replication_factor': 1
 };
   + Rreplication parameters required
   
#### USE
  + USE switches between keyspaces
  
 USE killrvideo
 
#### Tables
  + Keypsaces contains tables
  + Tables contains data
  + varchar is sames as text

 CREATE TABLE table1 (
  column1 TEXT,
  column2 TEXT,
  column3 INT,
  PRIMARY KEY (column1)
 );
 
 CREATE TABLE users (
  user_id UUID,
  first_name TEXT,
  last_name TEXT,
  PRIMARY KEY (user_id)
 );
 
#### UUID and TIMEUUID
 Used in place of integer IDs because Cassandra is a distributed database
  
  + Universally Unique Identifier
     + Ex: 34jfnck43- dkc02- ldmcfj-knjd-knndknw
     + Generate via uuid()
      
  + TIMEUUID embes a TIMESTAMP value
     + Ex: 12nsjncjdj-ksnd-jwd34-lanu34dnj
     + Sortable
     + Generate via now()
        
#### INSERT
  Similar to relational syntax
  
  INSERT INTO users (user_id, first_name, last_name)
  VALUES (uuid(), 'Sushil', 'Singh')
  
#### SELECT
  similar to relational syntax
  
  SELECT *
  FROM users
  WHERE user_id = 1j24bb-kqsn243- knx-k12jn3nksn-ksnn;
  
#### COPY
  + Import/exports CSV data
  
 COPY table1 (column1, column2, column3) FROM 'table1data.csv';
  
  + Header parameter skips the first line in the file
 COPY table1 (column1, column2, column3) FROM 'tabledata.csv'
 WITH HEADER=true;
   
   + There are several ways to get data into Apache Cassandra
         + COPY + Apache Spark + Drivers + Etc.,


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
