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






































