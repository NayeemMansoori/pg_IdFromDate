# pg_IdFromDate(): Benchmark

## Benchmark with a table > 10 million rows

We searched the same timestamp / dates on the same medium sized table (> 10 million rows).

### Synthesis

Method                            | "...WHERE date = '[DATE]';" | pg_IdFromDate() | Result                        
----------------------------------|-----------------------------|-----------------|--------------------------------
Querying a timestamp (no index)   | 2,264.76 ms                 | 10.04 ms        | pg_IdFromDate() is **225x faster**
Querying a date (no index)        | 3,252.57 ms                 | 10.10 ms        | pg_IdFromDate() is **322x faster**
Querying a timestamp (+ index)    | 3.58 ms                     | 10.97 ms        | pg_IdFromDate() is 3x slower
Querying a date (+ index)         | 3,303.71 ms                 | 10.54 ms        | pg_IdFromDate() is **313x faster**

### Querying a timestamp

```sql
-- Tests without an index
gab@benchmark # SELECT * FROM test WHERE date = '2012-05-09 08:37:21';
   id    |        date         |            some_data             
---------+---------------------+----------------------------------
 9000008 | 2012-05-09 08:37:21 | 80e27e440ff8d570b9a6bbaa72fe6733
(1 row)
Time: 2264.763 ms

gab@benchmark # SELECT * FROM test WHERE id = (SELECT pg_IdFromDate('test', 'date', '2012-05-09 08:37:21'));
   id    |        date         |            some_data             
---------+---------------------+----------------------------------
 9000008 | 2012-05-09 08:37:21 | 80e27e440ff8d570b9a6bbaa72fe6733
(1 row)
Time: 10.044 ms

-- Creating an index
gab@benchmark # CREATE INDEX ON test (date);
CREATE INDEX
Time: 24581.615 ms

-- Tests with an index
gab@benchmark # SELECT * FROM test WHERE date = '2012-05-09 08:37:21';
   id    |        date         |            some_data             
---------+---------------------+----------------------------------
 9000008 | 2012-05-09 08:37:21 | 80e27e440ff8d570b9a6bbaa72fe6733
(1 row)
Time: 3.582 ms

gab@benchmark # SELECT * FROM test WHERE id = (SELECT pg_IdFromDate('test', 'date', '2012-05-09 08:37:21'));
   id    |        date         |            some_data             
---------+---------------------+----------------------------------
 9000008 | 2012-05-09 08:37:21 | 80e27e440ff8d570b9a6bbaa72fe6733
(1 row)
Time: 10.104 ms
```

#### Querying a date

```sql
-- Tests without an index
gab@benchmark # SELECT * FROM test WHERE date::date = '2011-05-08' LIMIT 1;
   id    |        date         |            some_data             
---------+---------------------+----------------------------------
 8471011 | 2011-05-08 00:00:21 | 1954adfd4f5067b11d2ef096d8e98212
(1 row)
Time: 3252.576 ms

gab@benchmark # SELECT * FROM test WHERE id = (SELECT pg_IdFromDate('test', 'date', '2011-05-08'::date));
   id    |        date         |            some_data             
---------+---------------------+----------------------------------
 8471011 | 2011-05-08 00:00:21 | 1954adfd4f5067b11d2ef096d8e98212
(1 row)
Time: 10.976 ms

-- Creating an index
gab@benchmark # CREATE INDEX ON test (date);
CREATE INDEX
Time: 25555.699 ms

-- Tests with an index
gab@benchmark # SELECT * FROM test WHERE date::date = '2011-05-08' LIMIT 1;
   id    |        date         |            some_data             
---------+---------------------+----------------------------------
 8471011 | 2011-05-08 00:00:21 | 1954adfd4f5067b11d2ef096d8e98212
(1 row)
Time: 3303.716 ms

gab@benchmark # SELECT * FROM test WHERE id = (SELECT pg_IdFromDate('test', 'date', '2011-05-08'::date));
   id    |        date         |            some_data             
---------+---------------------+----------------------------------
 8471011 | 2011-05-08 00:00:21 | 1954adfd4f5067b11d2ef096d8e98212
(1 row)
Time: 10.542 ms
```

## Benchmark with a table > 100 million rows

### Synthesis

Method                            | "...WHERE date = '[DATE]';" | pg_IdFromDate() | Result                        
----------------------------------|-----------------------------|-----------------|--------------------------------
Querying a timestamp (no index)   | 25,225.50 ms                | 11.08 ms        | pg_IdFromDate() is **2,276x faster**
Querying a date (no index)        | 41,174.18 ms                | 10.74 ms        | pg_IdFromDate() is **3,833x faster**
Querying a timestamp (+ index)    | 12.44 ms                    | 10.22 ms        | pg_IdFromDate() is 2 ms slower
Querying a date (+ index)         | 43,361.46 ms                | 11.05 ms        | pg_IdFromDate() is **3,924x faster**

### Querying a timestamp

```sql
-- Tests without an index
gab@benchmark # SELECT * FROM test2 WHERE date = '2014-03-09 02:46:43';
    id    |        date         |            some_data             
----------+---------------------+----------------------------------
 99455363 | 2014-03-09 02:46:43 | 1fd9ee3f10aefc095c431c5053bd21f6
(1 row)
Time: 25225.509 ms

gab@benchmark # SELECT * FROM test2 WHERE id = (SELECT pg_IdFromDate('test2', 'date', '2014-03-09 02:46:43'));
    id    |        date         |            some_data             
----------+---------------------+----------------------------------
 99455363 | 2014-03-09 02:46:43 | 1fd9ee3f10aefc095c431c5053bd21f6
(1 row)
Time: 11.087 ms

-- Creating an index
gab@benchmark # CREATE INDEX ON test2 (date);
CREATE INDEX
Time: 227698.756 ms

-- Tests with an index
gab@benchmark # SELECT * FROM test2 WHERE date = '2014-03-09 02:46:43';
    id    |        date         |            some_data             
----------+---------------------+----------------------------------
 99455363 | 2014-03-09 02:46:43 | 1fd9ee3f10aefc095c431c5053bd21f6
(1 row)
Time: 12.447 ms

gab@benchmark # SELECT * FROM test2 WHERE id = (SELECT pg_IdFromDate('test2', 'date', '2014-03-09 02:46:43'));
    id    |        date         |            some_data             
----------+---------------------+----------------------------------
 99455363 | 2014-03-09 02:46:43 | 1fd9ee3f10aefc095c431c5053bd21f6
(1 row)
Time: 10.226 ms
```

#### Querying a date

```sql
-- Tests without an index
gab@benchmark # SELECT * FROM test2 WHERE date::date = '2014-03-09' LIMIT 1;
    id    |        date         |            some_data             
----------+---------------------+----------------------------------
 99455197 | 2014-03-09 00:00:43 | e394f1bf971bf54476f0fe956e847cab
(1 row)
Time: 41174.185 ms

gab@benchmark # SELECT * FROM test2 WHERE id = (SELECT pg_IdFromDate('test2', 'date', '2014-03-09'::date));
    id    |        date         |            some_data             
----------+---------------------+----------------------------------
 99455197 | 2014-03-09 00:00:43 | e394f1bf971bf54476f0fe956e847cab
(1 row)
Time: 10.745 ms

-- Creating an index
gab@benchmark # CREATE INDEX ON test2 (date);
CREATE INDEX
Time: 231331.880 ms

-- Tests with an index
gab@benchmark # SELECT * FROM test2 WHERE date::date = '2014-03-09' LIMIT 1;
    id    |        date         |            some_data             
----------+---------------------+----------------------------------
 99455197 | 2014-03-09 00:00:43 | e394f1bf971bf54476f0fe956e847cab
(1 row)
Time: 43361.469 ms

gab@benchmark # SELECT * FROM test2 WHERE id = (SELECT pg_IdFromDate('test2', 'date', '2014-03-09'::date));
    id    |        date         |            some_data             
----------+---------------------+----------------------------------
 99455197 | 2014-03-09 00:00:43 | e394f1bf971bf54476f0fe956e847cab
(1 row)
Time: 11.050 ms
```
