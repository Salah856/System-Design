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


### Data replication

To achieve high availability and reliability, data must be replicated asynchronously over N servers, where N is a configurable parameter. 

These N servers are chosen using the following logic: after a key is mapped to a position on the hash ring, walk clockwise from that position and choose the first N servers on the ring to store data copies.

![image](https://user-images.githubusercontent.com/23625821/132938370-c850fedd-4830-4676-91ac-ca1401de9e8d.png)

In previous figure, key0 is replicated at s1, s2, and s3.
