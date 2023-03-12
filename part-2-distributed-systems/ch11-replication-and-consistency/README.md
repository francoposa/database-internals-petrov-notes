# Chapter 11: Replication and Consistency

**Fault Tolerance** is a property of a system that can continue operating correctly in the presence of failures of its components.

Creating fault-tolerant systems is difficult; adding fault tolerance to existing systems is even more difficult.

The primary goal to increase fault tolerance is to remove single points of failure from the system and ensure redundancy in mission-critical components.
Usually the goal is to make redundancy transparent to the user.

**Data Replication** introduces redundancy by maintaining multiple copies of data in a system.

However, updating multiple copies of data atomically is a problem equivalent to consensus - nodes must all receive the data and agree that they have all received the data before allowing subsequent read operations to see the data.
This is very expensive to do for every write operation.

Various system architectures have been developed to provide ways to appear consistent to a user without all nodes having all of the most recent data all the time.

When data records are modified, their redundant copies must also be updated.
When handling replication, we care about three events: **write**, **replica update**, and **read**.

## Achieving Availability

Availability is one goal of a building a distributed system: from a user's perspective, the system should continue operating as normal despite failures of individual nodes.

## CAP Theorem

**Availability** is the ability of a system to respond to every request successfully.
Theoretically, availability includes the ability to respond eventually on some infinite timeline, but in reality there are quite non-infinite limits to how long we can tolerate waiting on a response.

**Consistency** has many different theoretical and practical definitions, but here we talk about **Linearizability**, a.k.a. **Atomic Consistency**.
The academic definition of linearizability is quite dense, but essentially it means that the operations in a system are committed in an ordered manner.

Further definition and examples of linearizability will follow, but you can think of it at a very high level like this:

* variable x has an initial value `x(0)`
* operation A writes a new value to x, `x(1)`
* if operation A is completed/committed, then any read operation beginning after A commits must not see the old value `x(0)`

**CAP Theorem** describes tradeoffs between **Consistency** and **Availability** in the event of a Network **Partition** - meaning some nodes in the system cannot communicate with each other.

In the event of a network partition, a partitioned node will not receive all updates.
If a partitioned node services a read without have received all writes, it will respond with data that may be inconsistent with the rest of the system.

As such, a system must be designed to prioritize one or the other in the event of a partition:

* **Consistent and Partition Tolerant (CP)**: prefer to fail to serve responses rather than potentially serve inconsistent data
* **Available and Partition Tolerant (AP)**: prefer to serve potentially inconsistent data rather than failing to serve a response

**PACELC** extends CAP to state that even when operating normally without network partitions (**E**lse), systems must choose between **C**onsistency and **L**atency.
A system takes a nonzero amount of time to establish consistency, and that time is the latency before the operation is committed and subsequent reads can see the update.
Users have limited thresholds they will tolerate for this latency - from the user's perspective, long-enough latency will have a similar effect as the system being unavailable.
