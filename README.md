# pg_IdFromDate()

pg_IdFromDate is a set of PostgreSQL functions that you can install on any PostgreSQL databases.

The objective is to be able to efficiently find an ID corresponding to a row with a date; even on very big data tables.

### Installation

* Install all methods from [sql/sql_functions.php](sql/sql_functions.php)
* If you want to use a sample data set, create the table in [sample_table/sample_table.sql](sample_table/sample_table.sql)
* Try it yourself with the examples below

### Use pg_IdFromDate()

pg_IdFromDate() requires 3 inputs: the table name, the name of the date column and the timestamp to search.

For example, if you have the following table:
```sql
gab@gab # SELECT * FROM test LIMIT 5 OFFSET 1000;
  id  |            date            |            some_data             
------+----------------------------+----------------------------------
 7603 | 2014-08-09 13:53:00.786386 | 97d7d73b7495a1d81f002e787462da97
 7604 | 2014-08-09 14:53:00.786386 | 127c29d00f2a563aacd6605a5d62767e
 7605 | 2014-08-09 15:53:00.786386 | 2e9424d87929f035197696134c1bd100
 7606 | 2014-08-09 16:53:00.786386 | 434aca7d7a4616265d291aad94ebb848
 7607 | 2014-08-09 17:53:00.786386 | 56405aa0889b1fe3bc5ad2abec185a58
(5 rows)
```

You might want a simple method to find the ID of the row with a certain date.

pg_IdFromDate() allows you to do that very quickly, even on very large tables with the following query:
```sql
gab@gab # SELECT pg_IdFromDate('test', 'date', '2014-08-09 16:53:00');
 pg_idfromdate 
---------------
          7606
(1 row)
Time: 9.270 ms

gab@gab # SELECT pg_IdFromDate('test', 'date', '2014-08-09');
 pg_idfromdate 
---------------
          7590
Time: 9.270 ms

gab@gab # SELECT pg_IdFromDate('test', 'date', (NOW() - interval '1 week')::timestamp);
 pg_idfromdate 
---------------
         13012
(1 row)
Time: 12.116 ms
```

You can also use the result as a subquery in other queries, for example:
```sql
-- To select all rows where the date is > 2014-08-09
SELECT * FROM test WHERE id > (SELECT pg_IdFromDate('test', 'date', '2014-08-09'));

-- Select all rows where dates are between 2014-08-09 and 2014-10-01
SELECT * FROM test WHERE id > (SELECT pg_IdFromDate('test', 'date', '2014-08-09')) AND id < (SELECT pg_IdFromDate('test', 'date', '2014-10-01'));

-- Delete all rows where the date is < 2014-08-09
DELETE FROM test WHERE id < (SELECT pg_IdFromDate('test', 'date', '2014-08-09'));
```

### Benchmark on a table > 100 million rows

Method                            | "...WHERE date = '[DATE]';" | pg_IdFromDate() | Result                        
----------------------------------|-----------------------------|-----------------|--------------------------------
Querying a timestamp (no index)   | 25,225.50 ms                | 11.08 ms        | pg_IdFromDate() is **2,276x faster**
Querying a date (no index)        | 41,174.18 ms                | 10.74 ms        | pg_IdFromDate() is **3,833x faster**
Querying a timestamp (+ index)    | 12.44 ms                    | 10.22 ms        | pg_IdFromDate() is 2 ms slower
Querying a date (+ index)         | 43,361.46 ms                | 11.05 ms        | pg_IdFromDate() is **3,924x faster**

Two detailed benchmarks (tables > 10 and > 100 million rows) with a detailed list of queries are available on our [Benchmark readme](benchmark/README.md).

### Limitations

* This tool is experimental and there is no guaranteed correct outcome
* If "id" and the "date" column are not strictly in the same chronological order, the output might be an invalid ID (for example if a date is modified and is not above and below the dates of the preceding and following rows).
* The "id" needs to be called "id" and be a unique primary key of the table.

### Author

**Gabriel Bordeaux**

+ [Website](http://www.gab.lc/) 
+ [Twitter](https://twitter.com/gabrielbordeaux)
