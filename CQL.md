## CQL

### How to create KeySpace?

```
create keyspace simplestrategyreplication WITH REPLICATION  = {'class': 'SimpleStrategy', 'replication_factor':'3'};
```
### How to create table ?

By default the partition is the primary key of the table.
```
use simplestrategyreplication;

create table courses (id varchar primary key);

```

### How to check the consistency Level ?

By default the consistency level is ONE.  

```
consistency;
```
