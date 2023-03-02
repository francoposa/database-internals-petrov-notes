# Chapter 8: Introduction and Overview

## Concurrent Execution

As soon as more than one execution thread is allowed to access the data without any synchronization, the exact outcome of concurrent reads and writes is unpredictable.

Let's say we have a variable `int x = 1` and two processes or threads with read and write access to it.
One process, `adder` will read the variable, add to it, the write the result.
Another process, `multiplier` will read the variable, multiply it, then write the result.

* Scenario 1:
  * adder reads x = 1
  * adder writes (1 * 3) = 3
  * multiplier reads x = 3
  * multiplier writes (3 * 2) = 6
  * **result: x = 6**
* Scenario 2:
  * adder reads x = 1
  * multiplier reads x = 1
  * adder writes (1 + 2) = 3
  * multiplier writes (1 * 2) = 2
  * **result: x = 2**
* Scenario 3:
  * adder reads x = 1
  * multiplier reads x = 1
  * multiplier writes (1 * 2) = 2
  * adder writes (1 + 2) = 3
  * **result: x = 3**
* Scenario 4:
  * multiplier reads x = 1
  * multiplier writes (1 * 2) = 2
  * adder reads x = 2
  * adder writes (2 * 2) = 4
  * **result: x = 4**

So with only two processes, there are many possible outcomes.

**Concurrency** is the first problem of distributed systems.
Concurrent programs, like the above, share properties with distributed systems - a thread or process read shared state, mutate, then write back to the shared state.

**Consistency Models** describe the order in which concurrent operations execute and are made visible to the participants.
Different consistency models offer stricter or looser constraints on the possible states a system can be in.

### Shared State in a Distributed System

In a distributed system, some sort of database takes the place of the shared memory in a concurrent program.

Even if we solve the process of concurrent access to the database, it cannot be guaranteed that all processes reading from and writing to the database are in sync.
A process may not receive a response from a database or another cooperating process for an indefinite amount of time.
The process cannot know what state the overall system is in, and we must make decisions on how to handle these scenarios.

The list of ways in which a distributed system can fail is extensive.

**Failure Models** are used to describe the various ways a system can fail (**failure modes**), in order to decide how a system should handle these scenarios.

**Fault Tolerance** describes if and how a system can continue operating under a given failure mode or modes.

## Fallacies of Distributed Computing

Peter Deutsch defined a famous list of fallacies of distributed computing.
A few of the more immediate ones to consider when designing a system are:

1. The network is reliable
2. Latency is zero
3. Bandwidth is infinite
4. Processing is instantaneous
5. Queues are infinite
6. Clocks are in sync
7. State is in sync


### Network Partitions and Partial Failures

**Network Partitions** are when two or more servers cannot communication with each other.
You may need to differentiate cases where a *single* node cannot communicate with others vs. cases where *groups* of nodes are isolated from each other but can continue communicating within the group.

Network partitions cause different failure modes than regular network unreliability.
General network unreliability can be handled in-process with retries, stateful connection protocols, etc.
Partitions, on the other hand, can produce two or more nodes or groups of nodes proceeding with an algorithm and creating conflicting views of system state.
Further, network partitions may not be symmetric - a process may be able to send but not receive messages.

### Cascading Failures

Failures in one part of a system will not remain isolated.
Without mitigations strategies, problems will propagate throughout the system.

A node coming back online after some time may immediately fail again under load from the *thundering herd* of queued operations which have been waiting.

Corrupted data will be replicated throughout the data if not validated, meaning one disk failure could place other nodes in a corrupted state.

Etc, etc, etc.

## Distributed Systems Abstractions
