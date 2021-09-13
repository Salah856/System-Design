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

A vector clock is a [server, version] pair associated with a data item. It can be used to check if one version precedes, succeeds, or in conflict with others.

Assume a vector clock is represented by D([S1, v1], [S2, v2], …, [Sn, vn]), where D is a data item, v1 is a version counter, and s1 is a server number, etc. 

If data item D is written to server Si, the system must perform one of the following tasks.

- Increment vi if [Si, vi] exists.
- Otherwise, create a new entry [Si, 1].

Even though vector clocks can resolve conflicts, there are two notable downsides. First, vector clocks add complexity to the client because it needs to implement conflict resolution logic.

Second, the [server: version] pairs in the vector clock could grow rapidly. To fix this problem, we set a threshold for the length, and if it exceeds the limit, the oldest pairs are removed. This can lead to inefficiencies in reconciliation because the descendant relationship cannot be determined accurately. 

However, based on Dynamo paper, Amazon has not yet encountered this problem in production; therefore, it is probably an acceptable solution for most companies.


## Gossip protocol works as follows:
- Each node maintains a node membership list, which contains member IDs and heartbeat counters.
- Each node periodically increments its heartbeat counter.
- Each node periodically sends heartbeats to a set of random nodes, which in turn propagate to another set of nodes.
- Once nodes receive heartbeats, membership list is updated to the latest info.
- If the heartbeat has not increased for more than predefined periods, the member is considered as offline.


![image](https://user-images.githubusercontent.com/23625821/132976753-a62831a7-f758-418e-af36-93316085a410.png)

### Handling temporary failures

After failures have been detected through the gossip protocol, the system needs to deploy certain mechanisms to ensure availability. In the strict quorum approach, read and write operations could be blocked as illustrated in the quorum consensus section.

A technique called “sloppy quorum” is used to improve availability. Instead of enforcing the quorum requirement, the system chooses the first W healthy servers for writes and first R healthy servers for reads on the hash ring. Offline servers are ignored.

If a server is unavailable due to network or server failures, another server will process requests temporarily. When the down server is up, changes will be pushed back to achieve data consistency. This process is called hinted handoff. Since s2 is unavailable, reads and writes will be handled by s3 temporarily. When s2 comes back online, s3 will hand the data back to s2.

![image](https://user-images.githubusercontent.com/23625821/132976820-8ec8fdfb-e349-4564-b2e8-da4ba7a43983.png)

### Handling permanent failures

Hinted handoff is used to handle temporary failures. What if a replica is permanently unavailable? To handle such a situation, we implement an anti-entropy protocol to keep replicas in sync. Anti-entropy involves comparing each piece of data on replicas and updating each replica to the newest version. A Merkle tree is used for inconsistency detection and minimizing the amount of data transferred.

A hash tree or Merkle tree is a tree in which every non-leaf node is labeled with the hash of the labels or values (in case of leaves) of its child nodes. Hash trees allow efficient and secure verification of the contents of large data structures

Step 1: Divide key space into buckets (4 in our example). A bucket is used as the root level node to maintain a limited depth of the tree.

![image](https://user-images.githubusercontent.com/23625821/132976937-09153501-7c32-4cb4-af23-4f13d65ed86f.png)

Step 2: Once the buckets are created, hash each key in a bucket using a uniform hashing method

![image](https://user-images.githubusercontent.com/23625821/132976949-c3a12897-0de4-4e21-998f-aa60b3712f1a.png)

Step 3: Create a single hash node per bucket

![image](https://user-images.githubusercontent.com/23625821/132976954-7e546860-49db-4a1f-a77c-0355f0a876bd.png)

Step 4: Build the tree upwards till root by calculating hashes of children. 

![image](https://user-images.githubusercontent.com/23625821/132976963-fbc4b019-c9c4-456a-b82b-50b67f1155a6.png)

To compare two Merkle trees, start by comparing the root hashes. If root hashes match, both servers have the same data. If root hashes disagree, then the left child hashes are compared followed by right child hashes. 

You can traverse the tree to find which buckets are not synchronized and synchronize those buckets only. Using Merkle trees, the amount of data needed to be synchronized is proportional to the differences between the two replicas , and not the amount of data they contain. 

In real-world systems, the bucket size is quite big. For instance, a possible configuration is one million buckets per one billion keys, so each bucket only contains 1000 keys.



## Reference materials

[1] Amazon DynamoDB: https://aws.amazon.com/dynamodb/

[2] memcached: https://memcached.org/

[3] Redis: https://redis.io/

[4] Dynamo: Amazon’s Highly Available Key-value Store: https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf

[5] Cassandra: https://cassandra.apache.org/

[6] Bigtable: A Distributed Storage System for Structured Data: https://static.googleusercontent.com/media/research.google.com/en//archive/bigtable-osdi06.pdf

[7] Merkle tree: https://en.wikipedia.org/wiki/Merkle_tree

[8] Cassandra architecture: https://cassandra.apache.org/doc/latest/architecture/

[9] SStable: https://www.igvita.com/2012/02/06/sstable-and-log-structured-storage-leveldb/

[10] Bloom filter https://en.wikipedia.org/wiki/Bloom_filter


https://awsbuilder.hashnode.dev/how-to-build-key-value-store




