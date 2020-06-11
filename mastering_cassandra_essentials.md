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

```
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

```
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


#### User-Defined Types (UDT)
Creating a new user-defined type is done using a ``CREATE TYPE`` statement.

```
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

```
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
