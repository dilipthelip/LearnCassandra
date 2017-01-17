## CQL ( Cassandra Query Language)

### How to launch cqlsh?
This is a command line tool to execute the CQL statements.  

```
cqlsh
```

### How to create KeySpace?

```
create keyspace simplestrategyreplication WITH REPLICATION  = {'class': 'SimpleStrategy', 'replication_factor':'3'};
```

**DURABLE WRITES:**  
This will skip the commit log entry when data is written to the Cassandra DB.   

```
create keyspace simplestrategyreplication WITH REPLICATION  = {'class': 'SimpleStrategy', 'replication_factor':'3'} AND DURABLE_WRITES= false;
```

**ALTER KEYSPACE:**  

```
ALTER keyspace simplestrategyreplication WITH REPLICATION  = {'class': 'SimpleStrategy', 'replication_factor':'4'};
```

After the above statement is run, we need to run the **nodetool repair** to update the cluster with this change.  

**DROP KEYSPACE**

```
DROP keyspace simplestrategyreplication;
```

### How to create table ?

By default the partition is the primary key of the table.

Approach 1:  
```
use simplestrategyreplication;

create table courses (id varchar primary key);
```

Approach 2:  
```
create table simplestrategyreplication.courses (id varchar primary key);

```

**ALTER TABLE:**

Add a Column:  
```
ALTER TABLE simplestrategyreplication.courses ADD NAME VARCHAR;
```

 Drop a Column:  
 
 ```
 ALTER TABLE simplestrategyreplication.courses DROP NAME;
 ```
 
 Removing all the data:  
 
 ```
 TRUNCATE simplestrategyreplication.courses;
 ```
 
 **DROP TABLE:**  
 
 ```
 DROP TABLE simplestrategyreplication.courses;
 ```

**TABLE PROPERTIES:**  

```
create table simplestrategyreplication.courses (id varchar primary key) WITH comment =' A Table of Courses';
```

### Data Types in Cassandra:

Cassandra Data Type  :   Java Compatible  
**NUMERIC**  
bigint               :      long  
decimal              :    java.math.BigDecimal   
double,float,int     :    java Equivalents  
varint               :    java.math.BigInteger  

**STRING**  
ascii,text,varchar   :    java.lang.String  

ascii -> US ASCII character string.  

**Date**  
timestamp.timeuuid  

**Other**  
boolean,uuid,inet,blob  

**Primary Key and Partition Key :**

Primary key column is also used as a partition key for the column.   

```
CREATE TABLE simplestrategyreplication.courses(
id varchar PRIMARY KEY,
title varchar,
author varchar
);

```

**COMPOSITE PRIMARY KEY:**  
```
CREATE TABLE simplestrategyreplication.courses(
id varchar ,
title varchar,
author varchar
PRIMARY KEY ((id, author));
);
```

With the above table structures there is going to be ony one row present per partition.The primary key determines how many rows a single partition is going to hold.  


### How to check the consistency Level ?

By default the consistency level is ONE.  

```
consistency;
```

### How to change the consistency Level ?

The below command will change the consistency to QUORUM.  

```
consistency QUORUM;
```

### How to insert data into the table ?

```
insert into simplestrategyreplication.courses  (id) values ('cassandra-developers');
```

### How to update data into the table ?

One cannot update the primary key of the table.  

```
update  simplestrategyreplication.courses set author ='java-developers' where id ='cassandra-developers';
```

Multiple partition update is also possible.  
```
update  simplestrategyreplication.courses set author ='java-developers' where id =('cassandra-developers','oracle-developers');

```

### How to select data from the table ?

Approach 1:  
```
select * from simplestrategyreplication.courses;
```

Approach 2:  
Cassandra suppports Alias **AS**  
```
select id as name from simplestrategyreplication.courses;
```

Approach 3:  

**IN** clause.
```
select id as name from simplestrategyreplication.courses where id in ('cassandra-developers');
```

Approach 4:  
Limit the result to a certain number;  
```
select * from simplestrategyreplication.courses LIMIT 100;
```

### How to set Tracing in Cassandra?

```
Tracing on;

-        Co-ordinator Node. ( Execute CQL3 query | 2017-01-17 10:42:43.386000 | 172.17.0.2 | 0 | 127.0.0.1)


activity                                                                                     | timestamp                  | source     | source_elapsed | client
----------------------------------------------------------------------------------------------+----------------------------+------------+----------------+-----------
                                                                           Execute CQL3 query | 2017-01-17 10:42:43.386000 | 172.17.0.2 |              0 | 127.0.0.1
 Parsing insert into courses (id) values ('oracle-developers'); [Native-Transport-Requests-1] | 2017-01-17 10:42:43.397000 | 172.17.0.2 |          19303 | 127.0.0.1
                                            Preparing statement [Native-Transport-Requests-1] | 2017-01-17 10:42:43.398000 | 172.17.0.2 |          20383 | 127.0.0.1
                              Determining replicas for mutation [Native-Transport-Requests-1] | 2017-01-17 10:42:43.399000 | 172.17.0.2 |          21012 | 127.0.0.1
                                                     Appending to commitlog [MutationStage-2] | 2017-01-17 10:42:43.403000 | 172.17.0.2 |          24708 | 127.0.0.1
              Sending MUTATION message to /172.17.0.3 [MessagingService-Outgoing-/172.17.0.3] | 2017-01-17 10:42:43.403000 | 172.17.0.2 |          24745 | 127.0.0.1
              Sending MUTATION message to /172.17.0.4 [MessagingService-Outgoing-/172.17.0.4] | 2017-01-17 10:42:43.403000 | 172.17.0.2 |          24745 | 127.0.0.1
                                                 Adding to courses memtable [MutationStage-2] | 2017-01-17 10:42:43.406000 | 172.17.0.2 |          27736 | 127.0.0.1
           MUTATION message received from /172.17.0.2 [MessagingService-Incoming-/172.17.0.2] | 2017-01-17 10:42:43.410000 | 172.17.0.4 |           1229 | 127.0.0.1
                                                     Appending to commitlog [MutationStage-1] | 2017-01-17 10:42:43.412000 | 172.17.0.4 |           3756 | 127.0.0.1
                                                 Adding to courses memtable [MutationStage-1] | 2017-01-17 10:42:43.412000 | 172.17.0.4 |           4178 | 127.0.0.1
                                          Enqueuing response to /172.17.0.2 [MutationStage-1] | 2017-01-17 10:42:43.413000 | 172.17.0.4 |           4442 | 127.0.0.1
      Sending REQUEST_RESPONSE message to /172.17.0.2 [MessagingService-Outgoing-/172.17.0.2] | 2017-01-17 10:42:43.413000 | 172.17.0.4 |           5005 | 127.0.0.1
           MUTATION message received from /172.17.0.2 [MessagingService-Incoming-/172.17.0.2] | 2017-01-17 10:42:43.423000 | 172.17.0.3 |           3486 | 127.0.0.1
                                                     Appending to commitlog [MutationStage-1] | 2017-01-17 10:42:43.427000 | 172.17.0.3 |           8211 | 127.0.0.1
                                                 Adding to courses memtable [MutationStage-1] | 2017-01-17 10:42:43.427000 | 172.17.0.3 |           8602 | 127.0.0.1
                                          Enqueuing response to /172.17.0.2 [MutationStage-1] | 2017-01-17 10:42:43.427000 | 172.17.0.3 |           8953 | 127.0.0.1
      Sending REQUEST_RESPONSE message to /172.17.0.2 [MessagingService-Outgoing-/172.17.0.2] | 2017-01-17 10:42:43.428000 | 172.17.0.3 |           9472 | 127.0.0.1
   REQUEST_RESPONSE message received from /172.17.0.4 [MessagingService-Incoming-/172.17.0.4] | 2017-01-17 10:42:43.456000 | 172.17.0.2 |          78001 | 127.0.0.1
                                Processing response from /172.17.0.4 [RequestResponseStage-3] | 2017-01-17 10:42:43.458000 | 172.17.0.2 |          78357 | 127.0.0.1
   REQUEST_RESPONSE message received from /172.17.0.3 [MessagingService-Incoming-/172.17.0.3] | 2017-01-17 10:42:43.528000 | 172.17.0.2 |         150098 | 127.0.0.1
                               Processing response from /172.17.0.3 [RequestResponseStage-12] | 2017-01-17 10:42:43.654000 | 172.17.0.2 |         276099 | 127.0.0.1
                                                                             Request complete | 2017-01-17 10:42:43.897540 | 172.17.0.2 |         511540 | 127.0.0.1
```

### How to turn OFF tracing ?

```
Tracing off;
```

### Scenario - Bring one of the node down:

```
docker stop cassandra2;

- Set the consistency to ALL.  

consistency ALL;

- Insert the data in to the DB.

cqlsh:simplestrategyreplication> insert into courses (id) values ('groovy2-developers');
NoHostAvailable:

Since one of the node is down and the inser is not successful

- Selecting a Data

cqlsh:simplestrategyreplication> select * from courses where id ='java-developers';

NoHostAvailable: 

- Change the Consistency

consistency QUORUM;

cqlsh:simplestrategyreplication> insert into courses (id) values ('groovy2-developers');

- Insert was succesful.


```
