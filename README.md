# LearnCassandra

Cassandra was originally developed at Facebook. Cassandra combines the distributed model of **Dynamo(Amazon)** and the data model of **Bigtable(Google)**.  

Cassandra installation is represented by bunch of nodes. Each node is a representation of a running instance of Cassandra. Cassandra cluster is a masterless and peer to peer system.  

All data stored in cassandra is associated with a **token**. Token ranges are from -2^63 to 2^63 .Nodes are responsible for storing the token values in to the Cassandra DB.When new modes are added , each node takes care of contiguos set of nodes managing the tokens.  

### Virtual Nodes:
- Each node in the cluster has a number of virtual nodes. Each vnode is equivalent to tokens. Lets say a node has 256 tokens then it is nothing but the node has 256 vnodes.  


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

## How to create a node in the cluster ?

Approach 1:  

```
docker run -dt --name=cassandra cassandra:1.0

```

Approach 2: Creating a node with data center and Rack.  

-dc - represents the data center.  
-rack - represents the rack.  

```
docker run -dt --name=cassandra cassandra:1.0 -dc DC1 -rack RAC1

```


## How to check the cassandra node is up and running in docker?

Cassandra has the nodetool which gives lot of status about cassandra cluster that is running inside the docker.  

```
docker exec -it cassandra nodetool status

Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.2  103.51 KiB  256          100.0%            3ebd1b04-d25d-4d9f-b43e-90a73463594d  rack1

```
UN -> U represnts the cluster is **up**, N represents the it is in **normal** status.  
256 -> Total no of V nodes.This is the default.  

## How to display all the token values ?

```
docker exec -it cassandra nodetool ring

Datacenter: datacenter1
==========
Address     Rack        Status State   Load            Owns                Token                                       
                                                                           9081570306984179364                         
172.17.0.2  rack1       Up     Normal  103.51 KiB      100.00%             -9173949671864146016                        
172.17.0.2  rack1       Up     Normal  103.51 KiB      100.00%             -9154750697609864773                        
172.17.0.2  rack1       Up     Normal  103.51 KiB      100.00%             -9068947733955971809 

```

## How to login to the Cassandra cluster?

```
docker exec -it cassandra /bin/bash
```

## How to view the cassandra.yaml?

```
cd /etc/cassandra

cat cassandra.yaml
```

## How to get the IP address of the current Cluster ?

```
docker inspect -f '{{ .NetworkSettings.IPAddress}}' cassandra

172.17.0.2

cassandra -> container name
```

## How to create another node in the cluster?

```
docker run --name cassandra2 -d -e CASSANDRA_SEEDS="$(docker inspect --format='{{ .NetworkSettings.IPAddress }}' cassandra)" cassandra:1.0

Checking the new node in the cluster:  
docker exec -it cassandra nodetool ring

Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.3  108.18 KiB  256          100.0%            96b8e0a2-7c40-4f55-8007-91480487362d  rack1
UN  172.17.0.2  129.96 KiB  256          100.0%            3ebd1b04-d25d-4d9f-b43e-90a73463594d  rack1
```

# Snitch:  
Snitch is what cassandra uses to get the understanding of **Cassandra**.

It used to efficiently route requests and it is consulted when storing multiple copies of the data.

**SimpleSnitch:** - The Simplesnitch is suitable for development and single data center deployments.   
**GossipingPropertyFileSnitch** - This is the protocol cassandra uses to talk to one another and keep everyone up to date on the state of the cluster.This snitch uses a property file in each node to describe that nodes location.Then the nodes gossip with each other.  
**PropertyFileSnitch** - The entire topolgy is defined in a single property file.  

Cloud and Other infrastuctures have their own snitch of their own.   

## Cassandra Terminology:

It is common to store multiple copies of the data across clusters. It increases reliality and performance.  

```
KeySpace->Table->Partition->row
```

- Data in cassandra is organized in to KeySpace(Similar to Oracle/MySQL tableSpace).
- KeySpace has Tables(It is close to its counterpart Oracle Table).
- All data written to cassandra is associated with a **Partition** key.
- This partition key determines where the data is stored in the cluster. All the data in the partition are stored together.
- Partition is primary interaction point when reading or writing data in to cluster.
- Data within a parition represented as one or more rows.
- **Replication Strategy** is determined at the keyspace level.
  - **SimpleStrategy:** This is best used in development or single data center clusters.  
    **Example CQL:**  - 
    ```
    create keyspace simpleStrategyReplication = 
    {'class': 'SimpleStrategy', 'replication_factor':'3'};
    
    ```
    - **Network Topology Strategy:** 
    **Example CQL:**  - 
    ```
    create keyspace simpleStrategyReplication = 
    {'class': 'NetworkTopologyStrategy', 'DC1':'3','DC2':1};
    
    ```
