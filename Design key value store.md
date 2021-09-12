# Design key value store 

## System components

We will discuss the following core components and techniques used to build a key-value store:

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

### Consistency

Since data is replicated at multiple nodes, it must be synchronized across replicas. Quorum consensus can guarantee consistency for both read and write operations. Let us establish a few definitions first.

N = The number of replicas. 

W = A write quorum of size W. For a write operation to be considered as successful, write operation must be acknowledged from W replicas.

R = A read quorum of size R. For a read operation to be considered as successful, read operation must wait for responses from at least R replicas.

![image](https://user-images.githubusercontent.com/23625821/132938428-0a0c0f34-414a-4a2a-983b-3f4b4cb1c8fe.png)

The configuration of W, R and N is a typical tradeoff between latency and consistency. If W = 1 or R = 1, an operation is returned quickly because a coordinator only needs to wait for a response from any of the replicas. If W or R > 1, the system offers better consistency; however, the query will be slower because the coordinator must wait for the response from the slowest replica.


If W + R > N, strong consistency is guaranteed because there must be at least oneoverlapping node that has the latest data to ensure consistency.

How to configure N, W, and R to fit our use cases? Here are some of the possible setups:

If R = 1 and W = N, the system is optimized for a fast read.
If W = 1 and R = N, the system is optimized for fast write.

If W + R > N, strong consistency is guaranteed (Usually N = 3, W = R = 2).
If W + R <= N, strong consistency is not guaranteed.

Depending on the requirement, we can tune the values of W, R, N to achieve the desired level of consistency.


### Consistency models

Consistency model is other important factor to consider when designing a key-value store. A consistency model defines the degree of data consistency, and a wide spectrum of possible consistency models exist:

- Strong consistency: any read operation returns a value corresponding to the result of the most updated write data item. A client never sees out-of-date data.
- Weak consistency: subsequent read operations may not see the most updated value.
- Eventual consistency: this is a specific form of weak consistency. Given enough time, all updates are propagated, and all replicas are consistent.


Strong consistency is usually achieved by forcing a replica not to accept new reads/writes until every replica has agreed on current write. This approach is not ideal for highly available systems because it could block new operations. Dynamo and Cassandra adopt eventual consistency, which is our recommended consistency model for our key-value store.

## Inconsistency resolution: versioning

Replication gives high availability but causes inconsistencies among replicas. Versioning and vector locks are used to solve inconsistency problems. Versioning means treating each data modification as a new immutable version of data. Before we talk about versioning, let us use an example to explain how inconsistency happens:


Both replica nodes n1 and n2 have the same value. Let us call this value the original value. Server 1 and server 2 get the same value for get(“name”) operation.

![image](https://user-images.githubusercontent.com/23625821/132976362-6d3e12b0-caf2-4a8a-a079-7123f1f1d493.png)

Next, server 1 changes the name to “johnSanFrancisco”, and server 2 changes the name to “johnNewYork”. These two changes are performed simultaneously. Now, we have conflicting values, called versions v1 and v2.

![image](https://user-images.githubusercontent.com/23625821/132976406-db04866d-c5f3-4fcb-a52a-30d1a4790513.png)

In this example, the original value could be ignored because the modifications were based on it. However, there is no clear way to resolve the conflict of the last two versions. 

To resolve this issue, we need a versioning system that can detect conflicts and reconcile conflicts. A vector clock is a common technique to solve this problem. Let us examine how vector clocks work.





