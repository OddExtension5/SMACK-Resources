## DS201: DataStax Enterpise 6 Foundation of Apache Cassandra

```
CQL, Partitions, Clustering Columns, Application Connectivity, Node, Ring, Peer to Peer, Vnodes, Gossip, Snitch, Replication, Consistency,Hinted Handoff, Read Repair, Node Sync, Write Path, Read Path, Compaction, Advance Performance
```

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














## DS220: DataStax Enterprize 6 Practical Application Data Modeling with Apache Cassandra

```
Data Modeling Overview, Relational Vs. Cassandra, Cassandra Table, Partition ANd Storage Structure, Clustering Columns, Denormalization, Table Features, Collections, UDTs, Counters, UDFs and UDAs, Conceptual Data Modelings, Application Workflow And Access Patterns, Logical Data Modeling, Analysis And Validation, Write Techniques, Read Techniques, Table/Key Optimization, Data Model Migration, Data Model-Anti Patterns, Data Modeling Use Cases
```





