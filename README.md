# LearnCassandra

Cassandra was originally developed at Facebook. Cassandra combines the distributed model of **Dynamo(Amazon)** and the data model of **Bigtable(Google)**.  

Cassandra installation is represented by bunch of nodes. Each node is a representation of a running instance of Cassandra. Cassandra cluster is a masterless and peer to peer system.  

All data stored in cassandra is associated with a **token**. Nodes are responsible for storing the token values in to the Cassandra DB.When new modes are added , each node takes care of contiguos set of nodes managing the tokens.  

## How Cassandra stores Data ?  
- Cassandra is a **partitioned row store**.Data are read and written with a partition key.
- When data is inserted the hash(partition key) is hashed in to the token.  
- Token value specifies which V node and physical node responsible for the data.  

## How to install Cassandra ?

```
brew install cassandra

```

## How to check Cassandra Version?

```
Type "cqlsh"

show VERSION
```

## How to download the docker file of Cassandra ?

clone thos repo and build the docker image based in your cassandra version.

```
https://github.com/docker-library/cassandra/tree/3ca0a18a575ae318f753ab1ecf01d54c93192681
```

## How to build the docker image?

```
docker build -t cassandra:1.0 .

```

## How to run the docker image ?

```
docker run -dt --name=cassandra cassandra:1.0

```



