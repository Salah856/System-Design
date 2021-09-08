# DESIGN CONSISTENT HASHING

To achieve horizontal scaling, it is important to distribute requests/data efficiently and evenly across servers. Consistent hashing is a commonly used technique to achieve this goal. 

## The rehashing problem

If you have n cache servers, a common way to balance the load is to use the following hash method:

    serverIndex = hash(key) % N, where N is the size of the server pool.


![1](https://user-images.githubusercontent.com/23625821/132457051-a0466301-bd1e-4579-a38a-19ae82874049.png)

To fetch the server where a key is stored, we perform the modular operation f(key) % 4. For instance, hash(key0) % 4 = 1 means a client must contact server 1 to fetch the cached data.

![image](https://user-images.githubusercontent.com/23625821/132457322-82e9bbe2-aec3-4644-a22c-90bed3edd97d.png)




