# DESIGN A URL SHORTENER

Here are the basic use cases:

1. URL shortening: given a long URL => return a much shorter URL. 
2. URL redirecting: given a shorter URL => redirect to the original URL. 
3. High availability, scalability, and fault tolerance considerations. 


## URL redirecting
Once the server receives a tinyurl request, it changes the short URL to the long URL with 301 redirect.

![image](https://user-images.githubusercontent.com/23625821/133559109-27b61b17-0fcb-4c14-b01d-2b336ac254f3.png)

![image](https://user-images.githubusercontent.com/23625821/133559179-4d1df5bb-4f5c-4c8f-b88e-064025b8d0a3.png)


A 301 redirect shows that the requested URL is “permanently” moved to the long URL. Since it is permanently redirected, the browser caches the response, and subsequent requests for the same URL will not be sent to the URL shortening service. Instead, requests are redirected to the long URL server directly.


A 302 redirect means that the URL is “temporarily” moved to the long URL, meaning that subsequent requests for the same URL will be sent to the URL shortening service first. Then, they are redirected to the long URL server.


Each redirection method has its pros and cons. If the priority is to reduce the server load, using 301 redirect makes sense as only the first request of the same URL is sent to URL shortening servers. However, if analytics is important, 302 redirect is a better choice as it can track click rate and source of the click more easily. 

The most intuitive way to implement URL redirecting is to use hash tables. Assuming the hash table stores <shortURL, longURL> pairs, URL redirecting can be implemented by the following:

- Get longURL: longURL = hashTable.get(shortURL)
- Once you get the longURL, perform the URL redirect.


## URL shortening

Let us assume the short URL looks like this: www.tinyurl.com/{hashValue}. To support the URL shortening use case, we must find a hash function fx that maps a long URL to the hashValue. 

![image](https://user-images.githubusercontent.com/23625821/133559649-0e159bff-7b42-4d7a-8867-0ed578d7fe35.png)


The hash function must satisfy the following requirements:
- Each longURL must be hashed to one hashValue.
- Each hashValue can be mapped back to the longURL.

#### Hash value length

The hashValue consists of characters from [0-9, a-z, A-Z], containing 10 + 26 + 26 = 62 possible characters. To figure out the length of hashValue, find the smallest n such that 62^n ≥ 365 billion. The system must support up to 365 billion URLs based on the back of the envelope estimation.


![1](https://user-images.githubusercontent.com/23625821/133881052-e443e794-3bf6-415e-98ca-1f557d5d702a.png)

When n = 7, 62 ^ n = ~3.5 trillion, 3.5 trillion is more than enough to hold 365 billion URLs, so the length of hashValue is 7.

#### Hash + collision resolution

To shorten a long URL, we should implement a hash function that hashes a long URL to a 7-character string. A straightforward solution is to use well-known hash functions like CR32, MD5, or SHA-1. The following table compares the hash results after applying different hash functions on this URL: https://en.wikipedia.org/wiki/Systems_design.


As shown, even the shortest hash value (from CRC32) is too long (more than 7 characters). How can we make it shorter?

The first approach is to collect the first 7 characters of a hash value; however, this method can lead to hash collisions. To resolve hash collisions, we can recursively append a new predefined string until no more collision is discovered. This process is explained


![image](https://user-images.githubusercontent.com/23625821/133881163-22cea99a-6d2d-441d-9a96-c06a1784fb60.png)

This method can eliminate collision; however, it is expensive to query the database to check if a shortURL exists for every request. A technique called bloom filters can improve performance. A bloom filter is a space-efficient probabilistic technique to test if an element is a member of a set. 


![image](https://user-images.githubusercontent.com/23625821/133916598-8c792065-79e1-4239-8783-75bda357e674.png)

### URL redirecting 

![image](https://user-images.githubusercontent.com/23625821/133916619-4247eba5-7705-4b4d-8390-4dbff343a876.png)

















