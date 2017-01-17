## CQL

### How to create KeySpace?

```
create keyspace simplestrategyreplication WITH REPLICATION  = {'class': 'SimpleStrategy', 'replication_factor':'3'};
```
