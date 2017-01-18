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

### How to create a composite primary key ?

Approach 1:  

In the below scenario it is a combination of **Primary key(partition_key, clustering_key)**. These together make a composite primary key.  

```
CREATE TABLE COURSES (
ID VARCHAR ,
NAME VARCHAR,
AUTHOR VARCHAR,
AUDIENCE INT,
DURATION INT,
CC BOOLEAN,
RELEASED TIMESTAMP,
MODULE_ID int,
MODULE_NAME varchar,
MODULE_DURATION int,
PRIMARY KEY(id,module_id)
) WITH COMMENT = 'A Table of Courses and modules';
```

Approach 2:  
```
PRIMARY KEY (P_KEY, C_key1,...Ckey_N);
```
Approach 3:  
```
PRIMARY KEY ((P_KEY1,...P_KEY_N), C_key1,...Ckey_N);

```

Approach 4:
```
PRIMARY KEY ((P_KEY1,...P_KEY_N))
```

### Static Columns:

These are used when you have a static content that is going to get repeated in a partition.This column will always hold the same value for each row in the partition.    

```
CREATE TABLE COURSES (
ID VARCHAR ,
NAME VARCHAR,
AUTHOR VARCHAR STATIC,
AUDIENCE INT,
DURATION INT,
CC BOOLEAN,
RELEASED TIMESTAMP,
MODULE_ID int,
MODULE_NAME varchar,
MODULE_DURATION int,
PRIMARY KEY(id,module_id)
) WITH COMMENT = 'A Table of Courses and modules';

```

### How to check the list of tables in the cassandra Cluster;

```
desc TABLES;
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
insert into simplestrategyreplication.courses  (id) values ('cassandra-developers','xyz','author1');
```

### How to update data into the table ?

One cannot update the **primary key** of the table.  

```
update  simplestrategyreplication.courses set author ='java-developers' where id ='cassandra-developers';
```

Multiple partition update is also possible.  
```
update  simplestrategyreplication.courses set author ='java-developers' where id =('cassandra-developers','oracle-developers');

```

### How to delete data from the table ?

**DELETE ROW:**  

```
delete from simplestrategyreplication.courses where id ='cassandra-developers';
```

**DELETE COLUMN**    

```
DELETE author from simplestrategyreplication.courses where id ='cassandra-developers';
```

```
update  simplestrategyreplication.courses set author ='null' where id =('cassandra-developers');
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

#### Distinct Keyword:

The Disinct keyword should only be applied to partiotin key columns and static columns.  

The primary key column should always be a part of the select statement.  
author is a static column in the below statement an it can be part of it.  

```
select distinct id, author from courses;
```

### Time Series Data:    

**TIMEUUID :**  

http://docs.datastax.com/en/archived/cql/3.0/cql/cql_reference/uuid_type_r.html?hl=timeuuid  

```
CREATE TABLE COURSE_PAGE_VIEWS(
COURSE_ID VARCHAR,
VIEW_ID TIMEUUID,
PRIMARY KEY (COURSE_ID,VIEW_ID)
) WITH CLUSTERING ORDER BY (VIEW_ID DESC);

--SAMPLE INSERTS

INSERT INTO COURSE_PAGE_VIEWS(COURSE_ID,VIEW_ID)
VALUES ('NODE-INTRO', now());

INSERT INTO COURSE_PAGE_VIEWS(COURSE_ID,VIEW_ID)
VALUES ('NODE-INTRO', now());

INSERT INTO COURSE_PAGE_VIEWS(COURSE_ID,VIEW_ID)
VALUES ('NODE-INTRO', now());

```


**DATE:**  

```
select dateOf(view_id) from COURSE_PAGE_VIEWS;

 system.dateof(view_id)
---------------------------------
 2017-01-18 14:37:06.021000+0000
 2017-01-18 14:37:04.160000+0000
 2017-01-18 14:37:04.156000+0000
```

**maxTimeUUID and minTimeUUID:**  

**>=** and **>** have the same effect.  

```
select dateOf(view_id) from COURSE_PAGE_VIEWS 
where COURSE_ID = 'NODE-INTRO' 
and view_id >= maxTimeuuid('2017-01-18 00:00+0000')
and view_id <= minTimeuuid('2017-01-19 00:00+0000') ;

```


### When the data was inserted?  

```
select id , writetime(author) from simplestrategyreplication.courses;

 id                   | writetime(author)
----------------------+-------------------
 cassandra-developers |  1484686747503022
```
The unix time of the date was written is returned. 

### Token value of the row ?

```
cqlsh:learncassandra> select id, token(id) from courses;

 id                          | system.token(id)
-----------------------------+----------------------
       AngularJS-Get-Started | -1307076543347084026
                  node-intro |  1230314183812605664
 raspberry-pi-for-developers |  1510057717685901069
                 advanced-JS |  3078562638948418255
         docker-fundamentals |  3390654573743950459
```

### Time to Live (TTL): 

The TTL value determines the time it is going to live in the database.The value corresponds to seconds.    

**SET ROW LEVEL TTL**  

APPROACH 1:  

```
insert into simplestrategyreplication.courses  (id,author) values ('cassandra-developers','xyz') USING TTL 32400;
```

APPROACH 2:  

The below statement is to have the value set for few seconds as mentioned in the TTL and expires after that.   

```
UPDATE USERS USING TTL 120 SET RESET_TOKEN='abc123' where id ='John1';

SELECT TTL(RESET_TOKEN) from USERS where id ='John1';

cqlsh:learncassandra> SELECT TTL(RESET_TOKEN) from USERS where id ='John1';

 ttl(reset_token)
------------------
              114
```



**SET TABLE LEVEL TTL**  

```
CREATE TABLE simplestrategyreplication.courses(
id varchar PRIMARY KEY,
title varchar,
author varchar
) WITH default_time_to_live = 10800;

```
### How to Expand the resultset in Cassandra?

```
expand on;
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
