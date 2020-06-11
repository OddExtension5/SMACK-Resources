# Notes of Mastering Cassandra Essentials Course by Tim Berglund

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

```
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

```
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


