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
