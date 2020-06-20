## Exercise 2

In this exercise, you will:
+ Create a keyspace for KillrVideo
+ Create a table to store video materials
+ Load the data for the video table from a CSV file

```python
# create a keypsace called killrvideo

CREATE KEYPSACE killrvideo
WITH replication = {
  'class': SimpleStrategy',
  'replication_factor': 1
};

# switch to the newly created keyspace

USE killrvideo;

# create a table videos in killrvideo

CREATE TABLE videos (
  video_id TIMEUUID,
  added_date TIMESTAMP,
  title TEXT,
  PRIMARY KEY (video_id)
);

# insert values in videos table

INSERT INTO videos (video_id, added_date, title) 
	VALUES (1645ea59-14bd-11e5-a993-8138354b7e31, '2014-01-29', 'Cassandra History');

INSERT INTO videos (video_id, added_date, title) 
	VALUES (245e8024-14bd-11e5-9743-8238356b7e32, '2012-04-03', 'Cassandra & SSDs');

INSERT INTO videos (video_id, added_date, title) 
	VALUES (3452f7de-14bd-11e5-855e-8738355b7e3a, '2013-03-17', 'Cassandra Intro');

INSERT INTO videos (video_id, added_date, title) 
	VALUES (4845ed97-14bd-11e5-8a40-8338255b7e33, '2013-10-16', 'DataStax DevCenter');

INSERT INTO videos (video_id, added_date, title) 
	VALUES (5645f8bd-14bd-11e5-af1a-8638355b8e3a, '2013-04-16', 'What is DataStax Enterprise?');

# check value is inserted correclty 

SELECT * 
FROM videos;

# remove the data you inserted

TRUNCATE videos;

# import data into your videostable

COPY videos(video_id, added_date, title)
FROM '~/labwork/data-files/videos.csv'
WITH HEADER=TRUE;

# to verify data loaded correctly

SELECT *
FROM videos;

# the number of imported rows

SELECT COUNT(*)
FROM videos;

# to leave cqlss

QUIT
```

## Exercise 3 - Partitions

In this exercise, you will:
  + Experiment with partitions
 

```python
# start the CQL command shell

/home/ubuntu/node/resources/cassandra/bin/cqlsh

# switch to the killrvideo keyspace

USE killrvideo;

# view the metadata for the video table

DESCRIBE TABLE videos;

# view the partitioner token value for each video id

SELECT token(video_id), video_id 
FROM videos;

# create a table for videos by tag

CREATE TABLE videos_by_tag (
  tag TEXT,
  video_id UUID,
  added_date TIMESTAMP,
  title TEXT,
  PRIMARY KEY ((tag), video_id)
);

# import the videos_by_tag data

COPY videos_by_tag(tag. video_id, added_date, title)
FROM '~/labwork/data-files/videos-by-tag.csv'
WITH HEADER = TRUE;

# verify imported data correctly

SELECT * 
FROM videos_by_tag;

# retrieve all rows tagged with cassandra

SELECT *
FROM videos_by_tag
WHERE tag = 'cassandra'

# retrieve all rows with datastax

SELECT *
FROM videos_by_tag
WHERE tag = 'datastax';

# retrieve the video a title of Cassandra

SELECT *
FROM videos_by_atg
WHERE title = 'Cassandra Intro';  // throws an error
```

## Exercise 4- Clustering Columns

In this exercise, you will:
+ Understand how clustering columns affect the underlying storage mechanism
+ Undertstand how clustering columns affect queries
    
```python
# using killrvideo keyspace

USE killrvideo;

# drop your videos_by_tag table

DROP TABLE vidoes_by_tag;

# create new vidoes_by_tag table

CREATE TABLE videos_by_tag (
    tag TEXT,
    video_id UUID,
    added_date TIMSESTAMP,
    title TEXT,
    PRIMARY KEY ((tag), added_date, video_id)
) WITH CLUSTERING ORDER BY(added_date DESC);

# import the videos_by_tag again via COPY command

COPY videos_by_tag(tag, video_id, added_date, title)
FROM '~/labwork/data-files/videos_by_tag.csv'
WITH HEADER = TRUE;

# perform a SELECT * query

SELECT * 
FROM videos_by_tag;

# execute your wuery again, but list the oldest videos first

SELECT * 
FROM videos_by_tag
ORDER BY added_date ASC; //query will fail. When specifying ORDER BY, you must restrict the partition key

# query to restrict key value to 'cassandra'

SELECT * 
FROM videos_by_tag
WHERE tag = 'cassandra'
ORDER BY added_data ASC;  

Note: Clustering columns allow range queries ( <, \<=, >=).

# query to retrieve videos made in 2013 or later

SELECT *
FROM videos_by_tag
WHERE tag = 'cassandra' and added_date >= '2013-1-1'
ORDER BY added_date ASC;
```

## Exercise 5- Drivers

In this exercise, you will:
+ You'll understand what Apache Cassandra drivers are and their purpose
+ You'll be able to create and read records using s driver

Drivers are the mechanism we use to interact with Apache Cassandra from programming language.

```python
from cassandra.cluster import Cluster

cluster = Cluster(protocol_version = 3)
session = cluster.connect('killrvideo')

# retrieve all records in the videos_by_tag table

for val in session.execute("SELECT * FROM videos_by_tag"):
        print(val[0])
        
# session.execute() returns a sequence of rows "tuples"

print('{0:12} {1:40} {2:5}'.format('Tag', 'ID', 'Title'))
for val in session.execute("SELECT * FROM videos_by_tag"):
       print('{0:12} {1:40} {2:5}'.format(val[0], val[2], val[3]))
       
# insert new video into the database

session.execute(
    "INSERT INTO vidoes_by_tag (tag, added_value, video_id, title)" +
    " VALUES ('cassandra', '2013-01-10', uuid(), 'Cassandra Is My Friend')"
    )

# view your new record
print('{0:12} {1:40} {2:5}'.format('Tag', 'ID', 'Title'))
for val in session.execute("SELECT * FROM videos_by_tag"):
      print('{0:12} {1:40} {2:5}'.format(val[0], val[2], val[3]))
      
# delete your record

session.execute("DELETE FROM videos_by_tag " +
                "WHERE tag = 'cassandra' AND added_date = '2013-01-10' AND +
                "videos_id = INSERT_YOUR_UUID_HERE" )
                
 # execute to verify your record is gone
 print('{0:12} {1:40} {2:5}'.format('Tag', 'ID', 'Title'))
 for val in session.execute("SELECT * FROM videos_by_tag"):
       print('{0:12} {1:40} {2:5}'.format(val[0], val[2], val[3]))

```

## Exercise 6 - Node

In this exercise, you will:
+ Understand what Apache Cassandra nodes are
+ Understand core hardware/software requirements of a node

Nodes are the building blocks of Apache Cassandra clusters. Therefore, it is useful to understand the care and feeding of nodes.

```python

## open the terminal and naviage to your

/home/ubuntu/node/resources/cassandra/bin/folder

## list all possible commands

./nodetool help

./nodetool status
'''
The status command shows information about the entire cluster, particularly the state of each node, 
and information about each of those node: IP address, dataload, number of tokens, total percentage of data saved on each node, 
host ID, and datacenter and rack.
'''

/home/ubuntu/node/bin/dsetool status
'''
Difference between dsetool status and nodetool status.
dsetool works with DataStax Enterprize as a whole (Apache Cassandra, Apache Spark, Apache Solr, Graph) whereas
nodetool is specific to Apache Cassandra
'''
./nodetool info
'''
info command display information about the connected node, which includes token information, hostID, protocol status, dataload,
node uptime, heap memeory usage and capacity, datacenter and rack information, number of errors reported, 
cache usage, and percentage of SSTables that have been incrementally repaired.
'''

./nodetool describecluster
'''
shows the settings that are common across all the nodes in the cluster and the cluster schema a version by each node.
'''

./nodetool getlogginglevels

./nodetool setlogginglevel org.apache.cassandra TRACE
'''
The command setlogginflevel dynamically changes the logging level used by Apache Cassandra without the need for a restart.
You can look at the /var/log/cassandra/system.log afterward to observe the changes.
'''

./nodetool settraceprobability 0.1
'''
Th resulatant value from the settraceprobability command represents a decimal describing the percentage 
of queries being saved, starting from 0(0%) to 1(100. Saved traces can then be viewed in the system_traces keyspace.
'''

./nodetool drain
'''
The drain command stops writes from occuring on the node and flushes all data to disk.
Typically, this command may be run before stopping an Apache Cassandra node.
'''

./nodetool stopdaemon
'''
The stopdaemon command stops a node's execution. Wait for it to complete
'''

/home/ubuntu/node/bin/dse cassandra
'''
Restart your node 
'''

'''
We will now stress the node using a simple tool called Apache Cassandra Stress. Once your node has restarted, navigate to the
/home/ubuntu/node/resources/cassandra/tools/bin directory in the terminal.
Run cassandra-stress to populate the cluster with 50,000 partitions using 1 client thread and without any warmup using:
'''

./cassandra-stress write n=50000 no-warmup -rate threads=1


./nodetool flush
'''
The flush command commits all written (memtables) data to disk.
Unlike drain, flush allows further writes to occur.
'''

```

## Exercise 7 - Ring

In this exercise, you will:
  + Understand the Apache Cassandra token ring
  + Create a two-node cluster
  
One of the secrets to Apache Cassandra performance is the use of a ring that keeps track of tokens. This ring enbales Apache Cassandra to know exactly which nodes contain which partitions. This ring also eliminates any single points of failiure.

```python
#1. shut down the current node
/home/ubuntu/node/resources/cassandra/bin/nodetool stopdaemon

#2. delete the /home/ubuntu/node folder 
cd /hone/ubuntu
rm -rf node

#3. make a two-node cluster
tar -xf dse-6.0.0-bin.tar.gz
mv dse-6.0.0 node1
labwork/config_node 1

tar -xf dse-6.0.0-bin.tar.gz
mv dse-6.0.0 node2
labwork/config_node 2

#4. open the /home/ubuntu/node2/resources/cassandra/conf/cassandra.yaml
vi cassandra.yaml

#5. change initial_token
initial_token: 9223372036854775807

#6. start the first node
/home/ubuntu/node1/bin/dse cassandra

#7. Check status of the node
/home/ubuntu/node1/resources/cassandra/bin/nodetool status

#8. Start second node
/home/ubuntu/node2/bin/dse cassandra

#9. Check for status
/home/ubuntu/node1/resources/cassandra/bin/nodetool status

#10. create table

CREATE KEYSPACE killrvideo
WITH REPLICATION = {
	'class': 'SimpleStrategy',
	'replication_factor': 1
};

USE killrvideo;

CREATE TABLE videos (
	id uuid,
	added_date timestamp,
	title text,
	PRIMARY KEY ((id))
);

COPY videos(id, added_date, title)
FROM '/home/ubuntu/labwork/data-files/videos.csv'
WITH HEADER=TRUE;

CREATE TABLE videos_by_tag (
	tag text,
	video_id uuid,
	added_date timestamp,
	title text,
	PRIMARY KEY ((tag), added_date, video_id ))
	WITH CLUSTERING ORDER BY (added_date DESC);
	
COPY videos_by_tag(tag, video_id, added_date, title)
FROM './home/ubuntu/labwork/data-files/videos-by-tag.csv'
WITH HEADER = TRUE;

#11. let's determine which nodes own which partitions in the videos_by_tag table

SELECT token(tag), tag
FROM videos_by_tag;

#12. check which nodes own which token ranges
/home/ubuntu/node1/resources/cassandra/bin/nodetool ring

#13. check for which partitions resided on which node

/home/ubuntu/node1/resources/cassandra/bin/nodetool getendpoints killrvideo video_by_tag 'cassandra'
/home/ubuntu/node1/resources/cassandra/bin/nodetool getendpoints kilrvideo video_by_tag 'datastax'

