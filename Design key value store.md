# Design key value store 


## System components

In this section, we will discuss the following core components and techniques used to build a key-value store:

- Data partition
- Data replication
- Consistency
- Inconsistency resolution

- Handling failures
- System architecture diagram
- Write path
- Read path


### Data partition

For large applications, it is infeasible to fit the complete data set in a single server. The simplest way to accomplish this is to split the data into smaller partitions and store them in multiple servers. There are two challenges while partitioning the data:

- Distribute data across multiple servers evenly.
- Minimize data movement when nodes are added or removed.

Using consistent hashing to partition data has the following advantages:

- Automatic scaling: servers could be added and removed automatically depending on theload.
- Heterogeneity: the number of virtual nodes for a server is proportional to the server capacity. 
- For example, servers with higher capacity are assigned with more virtual nodes.


