## CQL ( Cassandra Query Language)

### How to launch cqlsh?
This is a command line tool to execute the CQL statements.  
Approach 1:  
```
cqlsh
```

Approach 2 :   
```
cqlsh --cqlversion='3.3.1'
```

### How to create KeySpace?

```
create keyspace learncassandra WITH REPLICATION  = {'class': 'SimpleStrategy', 'replication_factor':'3'};
```

**DURABLE WRITES:**  
This will skip the commit log entry when data is written to the Cassandra DB.   

```
create keyspace learncassandra WITH REPLICATION  = {'class': 'SimpleStrategy', 'replication_factor':'3'} AND DURABLE_WRITES= false;
```

**ALTER KEYSPACE:**  

```
ALTER keyspace learncassandra WITH REPLICATION  = {'class': 'SimpleStrategy', 'replication_factor':'4'};
```

After the above statement is run, we need to run the **nodetool repair** to update the cluster with this change.  

**DROP KEYSPACE**

```
DROP keyspace learncassandra;
```

### How to check the list of tables in the cassandra Cluster;

```
desc TABLES;
```

### How to create table ?

By default the partition is the primary key of the table.

Approach 1:  
```
use learncassandra;

create table courses (id varchar primary key);
```

Approach 2:  
```
create table learncassandra.courses (id varchar primary key);

```

**ALTER TABLE:**

Add a Column:  
```
ALTER TABLE learncassandra.courses ADD NAME VARCHAR;
```

 Drop a Column:  
 
 ```
 ALTER TABLE learncassandra.courses DROP NAME;
 ```
 
 Removing all the data:  
 
 ```
 TRUNCATE learncassandra.courses;
 ```
 
 **DROP TABLE:**  
 
 ```
 DROP TABLE learncassandra.courses;
 ```
 
 #### Sample Data :  
 
 ```
drop table users; 

CREATE TABLE users (
id varchar,
first_name varchar ,
last_name varchar,
company varchar,

PRIMARY KEY (id)
);


INSERT INTO USERS (id,first_name,last_name,company) 
VALUES ('1','abc','def','XYZ');
 ```
 Lets see what happend if we insert the data with the same primary key.  
 
 ```
 INSERT INTO USERS (id,first_name,last_name,company) 
VALUES ('1','abc','def','PQR');
```

**TABLE PROPERTIES:**  

```
create table learncassandra.courses (id varchar primary key) WITH comment =' A Table of Courses';
```

### How to create a composite primary key ?

The structure of the primary has a significant effect on how many rows a single partition is going to hold.  
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

Compound Key - https://docs.datastax.com/en/cql/3.1/cql/ddl/ddl_compound_keys_c.html  
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

You always need a primary key column in the where statement to deleta a row from the table.  

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


### Static Columns:

These are used when you have a static content that is going to get repeated in a partition.This column will always hold the same value for each row in the partition.    

```
drop table courses; 

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



### DataTypes in Cassandra:

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

### Complex DataTypes in Cassandra:

#### SET:

```
CREATE TABLE collectiontest (
id varchar,
module_id int,
name varchar static,
features set<varchar> static,
PRIMARY KEY (id,module_id)
);
```


SET Insert:  
```
Insert into collectiontest (id, module_id,name,features) VALUES
('node-intro',1,'introduction to node',{'CC'});
```

SET Update Add :  
```
update collectiontest set features=features + {'cd'} where  id ='node-intro';
```

SET Update Remove :  
```
update collectiontest set features=features - {'cd'} where  id ='node-intro';

```

Empty entire SET:  
```
update collectiontest set features={} where  id ='node-intro';

```

#### LIST:

```
drop collectiontest; 

CREATE TABLE collectiontest (
id varchar,
module_id int,
name varchar static,
features list<varchar> static,
PRIMARY KEY (id,module_id)
);
```

LIST Insert:  
Insert using square brackets '['  
```
Insert into collectiontest (id, module_id,name,features) VALUES
('node-intro',1,'introduction to node',['CC']); 
```

LIST Update Add (Prepend) :   
```
update collectiontest set features=  ['cd']  + features  where  id ='node-intro';
```

LIST Update Add (Append) :   
```
update collectiontest set features=  features + ['cd'] where  id ='node-intro';
```

LIST Update Remove (Append) :  

Same string appears multiple times  in the list will also be removed.  

```
update collectiontest set features=  features - ['cd'] where  id ='node-intro';
```

LIST Update by Index  :  


```
update collectiontest set features[0] = 'abc' where  id ='node-intro';
```

Manipulating a LIST by element id (Append) :  

Same string appears multiple times  in the list will also be removed.  

```
update collectiontest set features=  features - ['cd'] where  id ='node-intro';
```
#### MAP:

If the use case is to rememeber previously logged in devices.Map datatype works like a charm in this case.  

```
drop table collectiontest; 

CREATE TABLE collectiontest (
id varchar,
module_id int,
name varchar static,
last_login map<varchar,timestamp> ,
PRIMARY KEY (id,module_id)
);
```

Map Insert:  
Inserting a key value pair is like inserting a Json Object.  

```
Insert into collectiontest (id, module_id,name,last_login) VALUES
('node-intro',1,'introduction to node',{'tab':'2017-01-15 14:57:03'}); 
```

Map update:  

```
update collectiontest set last_login['tab']= '2017-01-15 15:57:03'  where  id ='node-intro';
```

Map Update using TTL:  

```
update collectiontest using TTL 3200 
set last_login['tab']= '2017-01-15 15:57:03'  where  id ='node-intro';

```

Map Add a new Key:  

```
update collectiontest set last_login = last_login + {'laptop':'2017-01-15 14:57:03'}  where  id ='node-intro';
```

Map Delete:  

```
delete last_login['tab']  from collectiontest  where  id ='node-intro';
```
```
update collectiontest set last_login = last_login -  {'laptop'}  where  id ='node-intro';
```

Deleting an entire map:  
```
update collectiontest set last_login = {}  where  id ='node-intro';
 
```

#### Tuples:

Cassandra supports tuples data type.Tuple is a series of data types of different type of particular order.  

Consider the scenario where you would like to know the previously logged in timestamp and logged in ip address.  


```
(varchar,int,int,varchar,timestamp);
```

You must use the frozen keyword when you want to nest the complex data types.  
Nested types are serialized as single blob value.Nested values must be read as a whole.    

```
drop table collectiontest; 

CREATE TABLE collectiontest (
id varchar,
module_id int,
last_login map<varchar,frozen <tuple<timestamp,inet>>> ,
PRIMARY KEY (id)
);

```

Insert a Tuple:  

```
insert into collectiontest(id, module_id,last_login) 
values ('node',1,{'laptop':('2015-06-30 09:02:24','10.2.3.2')});
```

#### User Defined DataType:  

```

drop table collectiontest; 

CREATE TYPE clip (name varchar,duration int);


CREATE TABLE collectiontest (
id varchar,
clips list<frozen <clip>> ,
PRIMARY KEY (id)
);

```

Insert into User Defined Type:  

```
Insert into collectiontest (id, clips)
Values ('nodejs', [{name:'introduction',duration:38}]);
```

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


### Time Series Data:    

**TIMEUUID :**  

It comprises : 
http://docs.datastax.com/en/archived/cql/3.0/cql/cql_reference/uuid_type_r.html?hl=timeuuid  

**Functions:**  

https://docs.datastax.com/en/cql/3.3/cql/cql_reference/timeuuid_functions_r.html

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

### Secondary Indexes :  

Primary indexes are created based on the primary key of the table.  

Cassandra does have support for secondary Indexes. These indexes give more option to query data beyond the primary key fields.  


```

drop table users; 

CREATE TABLE users (
id varchar,
first_name varchar ,
last_name varchar,
company varchar,

PRIMARY KEY (id)
);


INSERT INTO USERS (id,first_name,last_name,company) 
VALUES ('1','abc','def','XYZ');
```


SELECT using index:  

We cannot use a column which is not a primary/cluster/index key in the where statement.You will get the below error message.  

```
InvalidRequest: Error from server: code=2200 [Invalid query] message="Predicates on non-primary-key columns (company) are not yet supported for non secondary index queries"
```
#### Create INDEX:  

The name for the index is not mandatory. 

```
CREATE INDEX users_Company ON users(company);

select * from users where company='XYZ';
```

#### Secondary Indexes on Collection:

Indexes in cassandra are supported for collections too. It supports map, list an set.


Creating an Index without the index name. Cassandra will name it for us.    

```

drop table users; 

CREATE TABLE users (
id varchar,
first_name varchar ,
last_name varchar,
company varchar,
tags set<varchar>,
PRIMARY KEY (id)
);


INSERT INTO USERS (id,first_name,last_name,company,tags) 
VALUES ('1','abc','def','XYZ',{'java'});
```


```
CREATE INDEX ON USERS(tags)
```

Select using collection INDEX:  

```
SELECT * FROM users where tags CONTAINS 'java';
```

By default index created for **MAP** refers to the values in the map.If you want the **where** clause to support keys in the map then index should be created like below.

```
CREATE INDEX ON TABLE(KEYS(<map_column>));

SELECT * FROM users 
WHERE <MAPCOLUMN> CONTAINS KEY <KEY>
```

#### Batches:  

Intended for keeping tables in sync. Not intended for fast loading of data.  

Batch is a group of CQL statements submitted together.As the co ordinator is processing the data the progress is communicated to the other nodes.Just incase if the coordinator node goes down then the other node can take over.   

Batch is not a transaction and there is no rollback feature. Just an assurance that all the data will be processed in the batch.  

![](https://github.com/dilipthelip/LearnCassandra/blob/master/images/Batches.png)

```
BEGIN BATCH

INSERT STATEMENT1;
INSERT STATEMENT2;
.
.
.
INSERT STATEMENTN;

APPLY BATCH;
```

#### UNLOGGED Batch:  

This will not perform multi node logging. This is more appropriate when you are writing multiple rows in to single partition.The batch will be sent to an appropriate node as a SINGLE Write.  

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
