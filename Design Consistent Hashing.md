# DESIGN CONSISTENT HASHING

To achieve horizontal scaling, it is important to distribute requests/data efficiently and evenly across servers. Consistent hashing is a commonly used technique to achieve this goal. 

## The rehashing problem

If you have n cache servers, a common way to balance the load is to use the following hash method:

    serverIndex = hash(key) % N, where N is the size of the server pool.


![1](https://user-images.githubusercontent.com/23625821/132457051-a0466301-bd1e-4579-a38a-19ae82874049.png)

To fetch the server where a key is stored, we perform the modular operation f(key) % 4. For instance, hash(key0) % 4 = 1 means a client must contact server 1 to fetch the cached data.

![2](https://user-images.githubusercontent.com/23625821/132457879-ca5c9d86-ab3f-4e29-abec-ad91d1a32d9b.png)


This approach works well when the size of the server pool is fixed, and the data distribution is even. However, problems arise when new servers are added, or existing servers are removed. For example, if server 1 goes offline, the size of the server pool becomes 3. Using the same hash function, we get the same hash value for a key. But applying modular operation gives us different server indexes because the number of servers is reduced by 1. 


![1](https://user-images.githubusercontent.com/23625821/132457499-dc2de4fb-370e-4a84-bbcb-c72e400c9ebb.png)


![1](https://user-images.githubusercontent.com/23625821/132457859-107839ee-5e35-4634-ad82-1f7f3b4e19be.png)


Most keys are redistributed, not just the ones originally stored in the offline server (server 1). This means that when server 1 goes offline, most cache clients will connect to the wrong servers to fetch data. This causes a storm of cache misses. Consistent hashing is an effective technique to mitigate this problem.

### Consistent hashing


Quoted from Wikipedia: "Consistent hashing is a special kind of hashing such that when a hash table is re-sized and consistent hashing is used, only k/n keys need to be remapped on average, where k is the number of keys, and n is the number of slots.

### Hash space and hash ring

Now we understand the definition of consistent hashing, let us find out how it works. Assume SHA-1 is used as the hash function f, and the output range of the hash function is: x0, x1, x2, x3, …, xn. In cryptography, SHA-1’s hash space goes from 0 to 2^160 - 1. 

That means x0 corresponds to 0, xn corresponds to 2^160 – 1, and all the other hash values in the middle fall between 0 and 2^160 - 1.


![image](https://user-images.githubusercontent.com/23625821/132458366-48f5b2fd-e448-4f7d-a20d-95bf6c2bca46.png)


By collecting both ends, we get a hash ring like following: 

![image](https://user-images.githubusercontent.com/23625821/132458458-8066f343-380c-49ff-8cd1-3292d0881fd6.png)

### Hash servers

Using the same hash function f, we map servers based on server IP or name onto the ring.

![image](https://user-images.githubusercontent.com/23625821/132458552-817519aa-64e6-47a6-bd1c-849dedd49252.png)

### Hash keys

One thing worth mentioning is that hash function used here is different from the one in “the rehashing problem,” and there is no modular operation. As shown, 4 cache keys (key0, key1, key2, and key3) are hashed onto the hash ring: 

![image](https://user-images.githubusercontent.com/23625821/132644590-dc63e626-e02b-4442-b91b-a38b97275a15.png)

### Server lookup
To determine which server a key is stored on, we go clockwise from the key position on the ring until a server is found.

![image](https://user-images.githubusercontent.com/23625821/132644801-0591abbf-02e5-4cf3-9349-776d9800f97d.png)

### Add a server

Using the logic described above, adding a new server will only require redistribution of a fraction of keys. Let us take a close look at the logic. Before server 4 is added, key0 is stored on server 0. Now, key0 will be stored on server 4 because server 4 is the first server it encounters by going clockwise from key0’s position on the ring. The other keys are not redistributed based on consistent hashing algorithm.

![image](https://user-images.githubusercontent.com/23625821/132645081-005c1d41-683c-46e1-bd33-d61d60176356.png)

### Remove a server

When a server is removed, only a small fraction of keys require redistribution with consistent hashing. When server 1 is removed, only key1 must be remapped to server 2. The rest of the keys are unaffected.

![image](https://user-images.githubusercontent.com/23625821/132645323-4ed47092-e099-42bc-a869-dee2251534b7.png)


##### The basic steps are:

- Map servers and keys on to the ring using a uniformly distributed hash function.
- To find out which server a key is mapped to, go clockwise from the key position until the first server on the ring is found.

###### Two problems are identified with this approach. 

- First, it is impossible to keep the same size of partitions on the ring for all servers considering a server can be added or removed. 
- A partition is the hash space between adjacent servers. 
- It is possible that the size of the partitions on the ring assigned to each server is very small or fairly large.


For example, if s1 is removed, s2’s partition (highlighted with the bidirectional arrows) is twice as large as s0 and s3’s partition.

![image](https://user-images.githubusercontent.com/23625821/132645996-9ba4b3fe-519f-457c-b3d3-2e7ee845d4f3.png)

### Virtual nodes

A virtual node refers to the real node, and each server is represented by multiple virtual nodes on the ring.


