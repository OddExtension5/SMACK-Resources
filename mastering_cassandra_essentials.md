# Mastering Cassandra Essentials Course by Tim Berglund

## Course Outline

  + Why Cassandra?
  + Data Model
  + Distributed Architecture
  + Read and Write Path
  + JAVA API

## Why Cassandra?

1. Scale: Cassandra is a database that works fundamentally at scale
2. Cassandra is distributed database, which gives highly available
3. A lot of data, too much for one server, maybe a priority on uptime, and data that changes rapidly or we might say, transactional data
4. Cassandra is an online, OLTP kind of a database

### Cassandra History

+ Open sourced from Facebook in July, 2008
+ Apache Incubator in March, 2009
+ Apache top-level in February, 2010
+ 0.8 in June, 2011 (**CQL**)
+ 2.0 in Sept, 2013 (**LWT**) Light Weight Transaction
+ 3.0 in Nov, 2015 (Materialized Views, SASI: SSTable Attached Secondary Indexes)

### Cassandra Status

+ Open Source, APL 2.0
+ Apache Software Foundation Project
+ Realted to DataStax Enterprise

## Data Model

### Cassandra Tables

+ A lot like relational tables
+ Narrow, arbitrarily tall
+ No joins
+ No foreign key constraints
+ Always exist within a keyspace

```python
CREATE KEYSPACE essentials 
WITH REPLICATION = {
        'class': 'SimpleStrategy',
        'replication_factor': 1
                   };
                   
CREATE TABLE movies (
        movie_id UUID,
        title TEXT,
        relaease_year INT<
        PRIMARY KEY ((movie_id))
                    );
                    
INSERT INTO movies (movie_id, relaease_year, title) values (uuid(), 2011, 'Trr of Life');
INSERT INTO movies (movie_id, relaease_year, title) values (uuid(), 2016, 'La La Land');
INSERT INTO movies (movie_id, relaease_year, title) values (uuid(), 2011, 'Birdman');

SELECT * FROM movies    //wrong 
                    
```

### Cassandra Keys

+ All table accesses are by key
+ Primary keys identigy rows
+ Composed of partition key and clustering key

```python
CREATE KEYSPACE essentials
WITH REPLICATION = {
        'class': 'SimpleStrategy',
        'replication_factor': 1
                  };
                  
 USE essentials;
 
 CREATE TABLE movies_by_actor (
      actor TEXT,
      release_year INT,
      movie_id UUID,
      title TEXT,
      genres SET<TEXT>,
      rating FLOAT,
      PRIMARY KEY ((actor), release_year, movie_id)
) WITH CLUSTERING ORDER BY (release_year DESC, movie_id ASC);

INSERT INTO movie_by_actor (actor, release_year, movie_id, title, generes, rating) 
VALUES ('Emma Stone', 1qjs234-edru21-2j34023-e3045kddkeu2, 'La La Land', {'musical', 'drama'}, 10);

INSERT INTO movie_by_actor (actor, release_year, movie_id, title, generes, rating) 
VALUES ('Emma Stone', 1qasd234-e12w21-2j2kend-3e1jw345kkeu2, 'Tree of Life', {'drama'}, 10);

SELECT * FROM movie_by_actor WHERE actor='Emma Stone'
SELECT * FROM movie_by_actor WHERE actor='Emma Stone' AND release_year > 2015
SELECT * FROM movie_by_actor WHERE actor='Emma Stone' AND release_year > 2015 and rating = 10   // it will not work because rating is not in PRIMARY KEY

```

### Cassandra Data Types
https://cassandra.apache.org/doc/latest/cql/types.html

+ text, int, bigint, float, double, boolean
+ decimal, varint
+ uuid ( uuid()) , timeuuid ( now() )
+ inet
+ tuple
+ timestamp  { not guarantee to be unique like timeuuid )
+ counter
+ set
+ list
+ map
+ user define data types

#### COUNTER
The counter type is used to define counter columns. A counter column is a column whose value is a 64-bit signed integer and on which 2 operations are suppoerted: incrementing and decrementing.

Counter have a number of important limitations:

+ They cannot be used for columns part of the ``PRIMARY KEY`` of a table.
+ A table that contains a counter can only contain counters. In other words, either all the columns of a table outside the ``PRIMARY KEY`` have the ``counter`` type, or none of them have it.

#### TIMESTAMPS
Values of the timestamp type are encoded as 64-bit signed integers representing a number of milliseconds since the standard base time known as the epoch: January 1 1970 at 00:00:00 GMT.

#### COLLECTIONS
CQL supports 2 kinds of collections: ``Maps``, ``Sets`` and ``Lists``. The types of those collections is defined by:

##### Map

```python
CREATE TABLE users (
      id text PRIMARY KEY,
      name text,
      favs map<text, text>  // A map of text keys, and text values
 );
 
 
INSERT INTO users (id, names, favs)
            VALUES ('jsmith', 'John Smith', { 'fruit': 'Apple', 'band': 'Beatles' } );
             
 // Replace the existing map entirely.
 UPDATE users SET favs = {'fruit': 'Banana' } WHERE id = 'jsmith';
 UPDATE users SET favs['author'] = 'Ed Poe' WHERE id = 'jsmith';
 UPDATE users SET favs = favs + { 'movie': 'Cassablanca', 'band': 'ZZ Top' } WHERE id = 'jsmith';
 
 DELETE favs['author'] FROM users WHERE id = 'jsmith';
 UPDATE users SET favs = favs - { 'movie', 'band' } WHERE id = 'jsmith';
 
 ```
 
##### Set
A set is a (sorted) collection of unique values. You can define and insert a map with:

```python
CREATE TABLE images (
      name text PRIMARY KEY,
      owner text,
      tags set<text> // A set of text values
);

INSERT INTO images (name, onwer, tags)
            VALUES ('cat.jpg', 'jsmith', {'pet', 'cute'});
            
// Replace the existing set entirely
UPDATE images SET tags = { 'kitten', 'cat', 'lol' } HWERE name = 'cat.jpg';
 
// adding one or multiple elements
UPDATE images SET tags = tags + { 'gray', 'cuddly' } WHERE name = 'cat.jpg';
 
// removing one or multiple elements
UPDATE images SET tags = tags - { 'cat' } WHERE name = 'cat.jpg';

#### Lists
A list is a (sorted) collection of non-unique values where elements are ordered by there position in the list.

```
```python
CREATE TABLE plays (
    id text PRIMARY KEY,
    game text,
    players int,
    scores list<int>  // A list of integers
);

INSERT INTO plays (id, game, players, scores )
            VALUES ('123-afde', 'quake', 3, [17, 4, 2]);

// Replace the existing list entirely
UPDATE plays SET scores = [ 3, 9, 4] WHERE id = '123-afde';

// appending and prepending values to a list
UPDATE plays SET players = 5, scores = scores + [14, 21 ] WHERE id = '123-afde';
UPDATE plays SET players = 6, scores = [3] + scores WHERE id = '123-afde';

//setting the value at a particular position in the list,
UPDATE plays SET scores[1] = 7 WHERE id = '123-afde';

//removing an element by its position in the list
DELETE scores[1] FROm plays WHERE id = '123-afde';

//deleting all the occurrences of a particular values in the list (if a particular elelemt doesn't occur at all in the list, it is simply ignored and no error is thrown):

UPDATE plays SET scores = scores - [12, 21] WHERE id = '123-afde';

```

#### User-Defined Types (UDT)
Creating a new user-defined type is done using a ``CREATE TYPE`` statement.

```python
CREATE TYPE phone (
        country_code int,
        number text,
)

CREATE TYPE address (
        street text,
        city text,
        zip text
        phones map<text, phone>
)

CREATE TABLE user (
      name text PRIMARY KEY,
      addresses map<text, frozen<address>>
)

```
Once a used-defined type has been created, value can be input using a UDT literal.

```
INSERT INTO user (name, addresses )
            VALUES ('z3 Pr3z1den7', {
                'home': {
                      street: '1600 Pennsylvania Ave NW',
                      city: 'Washington',
                      zip: '20500',
                      phones: { 'cell': { country_code: 1, number: '202 456-1111' },
                                'landline' : { country_code: 1, number: '...' } }
                  },
                  'work': {
                        street: '1600 Pennsylvania Ave NW',
                        city: 'Washington',
                        zip: '20500',
                        phones: { 'fax': { country_code: 1, number: '..' } }
                })
                
```

#### Tuples
CQL also supports tuples types (where the elements can be of different types).

```python
CREATE TABLE durations (
      event text,
      duration tuple<int, text>,
)

INSERT INTO durations (event, duration) VALUES ('ev1', (3, 'hours'));

```
A tuple is always frozen (without the need of the frozen keyword) and it is not possible to update some elements of a tuple (without updating the whole tuple).


### Secondary Indexes

+ Not like relational secondary indexes
+ Good for large partitions
+ SASI ( SSTable Attached Secondary Index)
+ http://www.doanduyhai.com/blog/?p=2058

```python
CREATE CUSTOM INDEX title ON movies_by_actor (title) USING
'org.apache.cassandra.index.sasi.SASIIndex' WITH OPTIONS = { 'mode': 'CONTAINS'};

```

Using CQL, you can create an index on a column after defining a table. You can also index a collection column.
Secondary indexes are used to query a table using a column that is not normally queryable.

Secondary indexes are tricky to use and can impact performance greatly. The index table is stored on each node in a cluster, so a query involving a secondary index can rapidly become a performance nightmare if multiple nodes are accessed.

``A general rule of thumb is to index a column with low cardinality of few values.``

```python
CREATE TABLE cycling.rank_by_year_and_name (
    race_year int,
    race_name text,
    cyclist_name text,
    rank int,
    PRIMARY KEY ((race_year, race_name), rank)
);

// both race_year and race_name must be specified as these columns comprise the partition key
SELECT * FROM cycling.rank_by_year_and_name WHERE race_year=2015 AND race_name='Tour of Japan';

//this query will fail, because the table has a composite partition key
SELECT * FROM cycling.rank_by_year_and_name WHERE race_year=2015;

//An index is created for the race year, and the query will succceed.
//An index name is optional and must be unique within a keyspace. If you don not provide a name, Cassandra will assign a name like race_year_idx

CREATE INDEX ryear ON cycling.rank_by_year_and_name (race_year);

SELECT * FROM cycling.rank_by_year_and_name WHERE race_year=2015;  // now it will work!

//A clustering column can also be used to create an index. An index is created on rank, and used in a query
CREATE INDEX rrank ON cycling.rank_by_year_and_name (rank);
SELECT * FROM cycling.rank_by_year_and_name WHERE rank = 1;

```

**When not to use an index**
+ On high-cardinality columns for a query of a huge volumne of records for a small number of results.
+ In tables that use a counter column
+ On a frequently updated or deleted column
+ To look for a row in a large partition unless narrowly queried.


### Query-First Modeling

+ We are used to modeling domains
+ Now we model queries as well
+ Dictated by key design and index behaviour
+ harder? Easier? Inevitable?

### Materialized Views

+ Materialized Views are suited for high cardinality data
+ The data in a materialized view is arranged serially based on the view's primary key.
+ Secondary Indexes are suited for low cardinality data.
+ Queries of high cardinality columns on secondary indexes require Cassandra to access all nodes in a cluster, causing high read latency.

**Restrictions for materialized views**:
  + Include all of the source table's primary keys in the materialized view's primary key
  + Only one new column can be added to the materialized view's primary key, Static column are not allowed
  + Exclude rows with null values in the materialized view primary key column

**Static column**: A special column that is shared by all rows of a partition

```python
CREATE TABLE cyclist_mv (
      cid UUID PRIMARY KEY,
      name text,
      age int,
      birthday date,
      country text
);

CREATE MATERIALIZED VIEW cyclist_by_age AS
        SELECT age, birthday, name, country
        FROM cyclist_mv
        WHERE age IS NOT NULL AND cid IS NOT NULL
        PRIMARY KEY (age, cid);
        
SELECT age, name, birthday FROM cyclist_by_age WHERE age = 18;

CREATE MATERIALIZED VIEW cyclist_by_birthday AS
         SELECT age, birthday, name, country
         FROM cyclist_mv
         WHERE birthday IS NOT NULL AND cid IS NOT NULL
         PRIMARY KEY (birthday, cid);
         
CREATE MATERIALIZED VIEW cyclist_by_country AS
          SELECT age, birthday, name, country
          FROM cyclist_mv
          WHERE country IS NOT NULL AND cid IS NOT NULL
          PRIMARY KEY (country, cid);
          
SELECT age, name, birthday FROM cyclist_by_country WHERE country = 'India';

SELECT age, name, birthday FROM cyclist_by_birthday WHERE birthday = '1987-09-04';

```
+ Cassandra can only write data directly to source tables, not to materialized views. Cassandra updates a materilaized view asynchronously after inserting data into the source table, so the update of materialized view is delayed. Cassandra performs a read repair to a materialized view only after updating the source table.

## Cassandra Distributed Architecture

### Nodes and Clusters

+ Remember: one computer is not enough
+ Every node is a peer
+ Ring Structure
+ Tokens and hashing
+ Elastic Scaling

Partition Key -----> Hashing ----> Token ID ----> Then It decide which node to go to

### Replication

+ Only one copy of data would be madness
+ Hardware failure is certain
+ Replication factor
+ Replica Placement Strategy

### Consistency

+ How many replicas is enough?
+ Introducing the coordinator
+ Tunable consistency levels
+ **R + W > N**

CL = ALL, Quorum (Majority Vote), ONE

I wanted to write ---> Pick a random node, chosed one node called Coordinator ---> Cordinator will hash the partition key and check the token for finding the node --> This will provide path to appropriate node for data write based on data replication

```R + W > N```
R: No. of node we can read <br/>
W: No. of nodes that we can write <br/>
N: Replication Factor <br/>

### Multiple Data Centers

+ Requires at business scale
+ Builds on consistency model
+ Tunable: **EACH_QUORUM** , **LOCAL_QUORUM**

### Virtual Nodes (vnodes)

+ Token assignment is work
+ Elastic scale is a burden on one node
+ num_tokens = 256

Vnodes distribute data across nodes at a finer granularity that can be easily achieved if calculated tokens are used.
Vnodes simplify many taska in Cassandra:
  + Tokens are automatically calculated and assigned to each node.
  + Rebalancing a cluster is automatically accomplished when adding or removing nodes. When a node joins the cluster, it assumes responsibility for an even portion of data from the other nodes in the cluster. If a node fails, the load is spread evenly across other nodes in the cluster.
  + Rebuilding a dead node is faster because it involves every node in the cluster.
  
**Note**: DataStax recommends using **8** vnodes (tokens). Using 8 vnodes distributes the workload between systems with a ~10% variance and has minimal impact on performance.

![Image](https://docs.datastax.com/en/cassandra-oss/3.0/cassandra/images/arc_vnodes_compare.png)

### Gossip

Gossip is a peer-to-peer communication protocol in which nodes periodically exhange state information about themselves and about other nodes they know about. The gossip process runs every second and exchanges state messages with up to three other nodes in the cluster. The nodes exchange information about themselves abd about the other nodes that they have gossiped about, so all nodes quickly learn about all other nodes in the cluster.

A gossip message has a version associated with it, so that during a gossip exachnge, older information is overwritten with the most current state for a particular node.

[Read More](https://www.linkedin.com/pulse/gossip-protocol-inside-apache-cassandra-soham-saha/)

## Read and Write Path in Cassandra

### The Write Path

+ Log-structured merge tree
+ Commit all writes to a log immediately [ONE per Node]
+ Buffer writes in the memtable [Per Table]
+ Flush sequentially to immutable SSTables 


**Coordinator** sending a write ---> sent to **COMMIT LOG** [one per node] ---> Put this into **Mem Table** [one per table] ---> If written succesfully on memtable then we get ACK from MemTable to coordinator  ---> If mem table fills up then flush data into the **SSTable** (disk).

![Image2](https://docs.datastax.com/en/cassandra-oss/2.1/cassandra/images/dml_write-process_12.png)

+ Memtables and SSTables are maintained per table.
+ SSTables are immutable, not written to again the memtable is flushed.

### The Read Path

+ A process of assembling the rows and columns we need
+ First look in the memtable
+ Then look through the SSTables
+ Key cache and bloom filters
+ Commit doesn't participate in read path

**Coordinator** sending a read ---> Do we have what we need in **Memtable** ---> If we don't find in Memtable then go to **SSTable** ---> **Bloom filter** helps to find out the in which partition of SStable my data resides (probabilistics) ---> If Bloom Filter says yes it may reside in this partition then check on the **Key Cache** if its previously read if not ---> Check on SSTable partition


``Coordinator --> Memtable --> Bloom FIlter ---> Key Cache --> SSTable partition``

![image34](https://docs.datastax.com/en/archived/cassandra/3.0/cassandra/images/dml_caching-reads_12.png)

### Compaction

+ Wait, we just keep writing SSTables forever?
+ Compaction two SSTables
+ Compaction and tombstones
+ Size-tiered compaction ---> Write heavy workload
+ Leveled COmpaction ---> Read heavy workload

Picking the right compaction strategy for your workload will ensure the best performance for both querying and for compaction itself.

1. Size Tiered Compaction Strategy <br/>
      The default compaction strategy. Useful as fallback when other strategies don't fit the workload.
      Most useful for non pure time series workloads with spinning disks, or when the I/O from lCS is too high
      
2.  Leveled Compaction Startegy <br/>
      LCS is optimised for read heavy workloads, or workloads with lots of updates and deletes. It is not good choice for immutable time series data
      
3. Time Window Compaction Strategy <br/>
      Time Window Compaction Startegy is designed for TTL'ed, mostly immutable time series data.
      
``Type of Compaction``

The concept of compaction is used for different kinds of operations in Cassandra, the common thing about these operations is that it takes one or more sstables and output new sstables. The types of compactions are:

**Minor compaction** <br/>
    triggered automatically in Cassandra

**Major compaction** <br/>
a user executes a compaction over all sstables on the node.

**User defined compaction** <br/>
a user triggers a compaction on a given set of sstables.


## The Java Driver in Cassandra

### Java Driver

```java
build.gradle

apply plugin: 'java'
apply plugin: 'idea'

repositories {
    jcenter(0
}

dependencies {
    compile 'com.datastax.cassandra:cassandra-river-core:3.2.0'
    compile 'com.datastax.cassandra:cassandra-driver-mapping:3.2.0'
}

task cassandra(type: javaExec) {
  classpath = sourceSets.main.runtimeClasspath
  main = 'Cassandra'
}
```
```java
import com.datastax.driver.core.*;
import java.util.UUID;

public class Cassandra {

  public static void main(STring args[]) {
       Cluster cluster;
       Session session;
       
       cluster = Cluster.builder().addContactPoint("127.0.0.1").build();
       session = cluster.connect("essentials");
       
       session.execute("INSERT INTO movies (movie_id, title, release_year) VALUES (UUID(), 'Blade RUnner', 2017););
       
       ResultSet results = session.execute("SELECT * FROM movies");
       for ( Row row: results) {
       System.out.format("%s %s %s\n", row.getUUID("movie_id"), row.getString(:title"), row.getInt("release_year"));
       }
       
     }
  }
  
```

