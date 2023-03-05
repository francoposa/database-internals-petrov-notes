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

### Links

We can define several communication protocols, working from least reliable and robust and building up.

#### Fair-Loss Link

* two processes, connected with a link
* processes can send messages across the link
* messages can get lost or delayed

After a message `m` is sent, from the sender's perspective, `m` can be in one of the following states:
* not yet delivered, but will be at some point
* irrecoverably lost during transport
* successfully delivered

Note that the sender does not have a way to find out if the message is already delivered.
This is a fair-loss link.

**Fair-Loss Links** have the following properties:
* **fair loss**: if both sender and recipient are correct and the sender keeps retramnsmitting the message infinitely, it will eventually be delivered
* **finite duplication**: send messages won't be delivered infinitely many times
* **no creation**: a link will not create messages that were never sent

In plain language, we have a link that is not *systematically* losing messages and not creating messages that were never sent.
At the same time, the link is not entirely reliable.

#### Message Acknowledgements

**Acknowledgements**: a way for the recipient to notify the sender that it has received the message

To build acknowledgements, we need bidirectional communication channels and unique identifiers to distinguish between the messages.
Unique identifiers can be simple, like monotonically increasing sequence numbers, or more complex, like hashes, which need to account for collisions and may take more computational power.

Now, with acknowledgements, our possible states for a message change a bit.

Now a message `m(n)`is sent, where `n` is our sequence counter.
After it is sent, the possible states are the same as in the fair-loss link.

However, after the recipient sends an `ack(n)`, the sender is certain message `m(n)` is in a delivered state.

#### Message Retransmits

Message acknowledgements are not enough to make a communication protocol reliable.
Messages may still get lost in transmit, or the remote process may fail before sending an acknowledgement.

**Retransmits** are a way for the sender to retry a potentially failed operation.
We specify *potentially* failed, as for this example we are not using acknowledgements, so the sender cannot be certain whether the message has failed or not.

With retransmits, after a process sends message `m`, it waits until timeout `T`, then tries to send the same message again.

After message `m` is sent, from the sender's perspective, the message is either successfully delivered, or is not *yet* delivered, but will be eventually - assuming any network partitions do not last infinitely long and packet loss is not 100% along the link.

This is called a **Stubborn Link**.
This type of link is highly impractical in the real world.

Retransmits must be combined with acknowledgements to provide a practical solution

#### Problems With Retransmits

Retransmitting a message which may not be yet delivered is only safe if the receiver can process the message as an idempotent operation.

**Idempotent** operations produce a system in the same state no matter how many times the operations are executed, without any additional side effects.

Ex:
* a `PUT` HTTP request processed as a create-or-update operation on a uniquely identifiable resource
  * the first request creates the resource and returns it; subsequent requests return the resource without changing it or creating a new resource
* a remove server shutdown request
  * the first request shuts down the server; subsequent requests do not change the state of the server and it remains shut down

The idempotence of an operation may depend on the context and bounds of the system under analysis.
For example, even though the above examples do not change the state of the main database or server, they may be recorded in a separate logging or eventing system.

In order to achieve idempotence, the receiver must be able to *deduplicate* messages to avoid processing retransmitted messages more than once.
In the `PUT` HTTP request example, the receiver uses a unique identifier to understand that the same resource has been sent more than once, and a new resource does not need to be created.

#### Message Order

Unreliable network links present two problems:
* messages can arrive out of order
* messages may arrive more than once (due to retransmits)

Message sequence numbers allow the receiver know the intended order of messages and ensure FIFO (first in first out) in-order processing.
The receiver can track:
* `n_consecutive` - the highest sequence number up to which the receiver has received all messages
  * messages up through `n_consecutive` can be placed in consecutive order
* `n_processed` - the highest sequence number up to which the receiver has placed into their consecutive order *and processed*
  * this number can be used for deduplication to achieve idempotence

If message `m(n)` is received where `n > n_consecutive + 1`, then there are missing messages, so `m(n)` can be replaced in a reordering buffer until the missing messages are received.
On a fair-loss link, we can assume the missing messages between `n_consecutive` and `n_max_seen` will eventually be delivered.

If a message `m(n)` is received where `n <= n_consective`, then it can be discarded.

Deduplication is accomplished during processing by checking if message `m(n)` has already been processed, using the sequence number `n`.

This is a *Perfect Link*, which provides the following guarantees:
* **reliable delivery**: every message sent at least once will eventually be delivered
* **no duplication**: no message is delivered more than once
* **no creation**: as with other link types, a link will not create messages that were never sent

This simplified model demonstrates properties similar to the TCP protocol, though real-world protocols are necessarily much more sophisticated.

#### Exactly-Once Delivery

**Exactly-Once Delivery** is where a message is only ever sent once, and only ever received once.
This is impossible to guarantee without perfectly reliable network links, which only exist in theory.

As discussed in previous sections, the sender cannot be certain a message is delivered and therefore must retransmit until an acknowledgement is received.
The receiver cannot be certain that the acknowledgement was received, so the receiver cannot assume that the same message will not be sent again.

Instead, network communication protocols generally use **At-Least-Once** delivery, where the sender retransmits until it receives an acknowledgement, and the receiver uses deduplication to avoid processing any messages delivered more than once.

Some communication needs may call for an *unreliable protocol* such as UDP, utilizing **At-Most-Once** delivery, where the sender transmits each message once and does not track whether the message is received.
