# DESIGN A UNIQUE ID GENERATOR

Let's assume that requirements of the system are: 

- IDs must be unique.
- IDs are numerical values only.
- IDs fit into 64-bit.
- IDs are ordered by date.
- Ability to generate over 10,000 unique IDs per second.

Multiple options can be used to generate unique IDs in distributed systems. The options we considered are:

- Multi-master replication
- Universally unique identifier (UUID)
- Ticket server
- Twitter snowflake approach

## Multi-master replication

![image](https://user-images.githubusercontent.com/23625821/133215703-d8fb4ea3-83f7-46f0-964b-25fc6c65c7b8.png)


This approach uses the databases’ auto_increment feature. Instead of increasing the next ID by 1, we increase it by k, where k is the number of database servers in use. Next ID to be generated is equal to the previous ID in the same server plus 2. This solves some scalability issues because IDs can scale with the number of database servers.

However, this strategy has some major drawbacks:
- Hard to scale with multiple data centers
- IDs do not go up with time across multiple servers.
- It does not scale well when a server is added or removed.


## UUID

A UUID is another easy way to obtain unique IDs. UUID is a 128-bit number used to identify information in computer systems. UUID has a very low probability of getting collusion. Quoted from Wikipedia, “after generating 1 billion UUIDs every second for approximately 100 years would the probability of creating a single duplicate reach 50%”

![image](https://user-images.githubusercontent.com/23625821/133216432-7145f6bd-3cdf-4e44-b014-73a7d0ac5f09.png)

In this design, each web server contains an ID generator, and a web server is responsible for generating IDs independently.

#### Pros:
- Generating UUID is simple. No coordination between servers is needed so there will not be any synchronization issues.
- The system is easy to scale because each web server is responsible for generating IDs they consume. ID generator can easily scale with web servers.

#### Cons:
- IDs are 128 bits long, but our requirement is 64 bits.
- IDs do not go up with time.
- IDs could be non-numeric.



## Ticket Server

Ticket servers are another interesting way to generate unique IDs. Flicker developed ticket servers to generate distributed primary keys. It is worth mentioning how the system works.

![image](https://user-images.githubusercontent.com/23625821/133216916-d3379b10-3fc5-4a90-b023-cc62654e2727.png)

The idea is to use a centralized auto_increment feature in a single database server (Ticket Server). 

#### Pros:
- Numeric IDs.
- It is easy to implement, and it works for small to medium-scale applications.

#### Cons:
- Single point of failure. 
- Single ticket server means if the ticket server goes down, all systems that depend on it will face issues. 
- To avoid a single point of failure, we can set up multiple ticket servers. 
- However, this will introduce new challenges such as data synchronization.


## Twitter snowflake approach
Approaches mentioned above give us some ideas about how different ID generation systems work. However, none of them meet our specific requirements; thus, we need another approach. Twitter’s unique ID generation system called “snowflake” is inspiring and can satisfy our requirements. Divide and conquer is our friend. Instead of generating an ID directly, we divide an ID into different sections.

![image](https://user-images.githubusercontent.com/23625821/133381427-14e35715-96f1-4676-9089-4b3612bfa64c.png)


Each section is explained below:

- Sign bit: 1 bit. It will always be 0. This is reserved for future uses. It can potentially be used to distinguish between signed and unsigned numbers.
- Timestamp: 41 bits. Milliseconds since the epoch or custom epoch. We use Twitter snowflake default epoch 1288834974657, equivalent to Nov 04, 2010, 01:42:54 UTC.
- Datacenter ID: 5 bits, which gives us 2 ^ 5 = 32 datacenters.
- Machine ID: 5 bits, which gives us 2 ^ 5 = 32 machines per datacenter.
- Sequence number: 12 bits. For every ID generated on that machine/process, the sequence number is incremented by 1. The number is reset to 0 every millisecond.

Datacenter IDs and machine IDs are chosen at the startup time, generally fixed once the system is up running. Any changes in datacenter IDs and machine IDs require careful review since an accidental change in those values can lead to ID conflicts. Timestamp and sequence numbers are generated when the ID generator is running.










































