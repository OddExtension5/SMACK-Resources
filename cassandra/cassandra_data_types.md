## CASSANDRA DATA TYPES

### COUNTER
The counter type is used to define counter columns. A counter column is a column whose value is a 64-bit signed integer and on which 2 operations are suppoerted: incrementing and decrementing.

Counter have a number of important limitations:

+ They cannot be used for columns part of the ``PRIMARY KEY`` of a table.
+ A table that contains a counter can only contain counters. In other words, either all the columns of a table outside the ``PRIMARY KEY`` have the ``counter`` type, or none of them have it.

### TIMESTAMPS
Values of the timestamp type are encoded as 64-bit signed integers representing a number of milliseconds since the standard base time known as the epoch: January 1 1970 at 00:00:00 GMT.

### COLLECTIONS
CQL supports 2 kinds of collections: ``Maps``, ``Sets`` and ``Lists``. The types of those collections is defined by:

#### Map

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
 
#### Set
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
```

### User-Defined Types (UDT)
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

### Tuples
CQL also supports tuples types (where the elements can be of different types).

```
CREATE TABLE durations (
      event text,
      duration tuple<int, text>,
)

INSERT INTO durations (event, duration) VALUES ('ev1', (3, 'hours'));

```
A tuple is always frozen (without the need of the frozen keyword) and it is not possible to update some elements of a tuple (without updating the whole tuple).
