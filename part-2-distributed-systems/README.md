# Part 2: Distributed Systems

**vertical scaling**: running the same software on a single, larger, higher-performance machine

**horizontal scaling**: running software on multiple machines connected over a network and working as a single logical entity

## Basic Definitions

Distributed systems have multiple **participants** a.k.a. processes, nodes, replicas, actors, workers, etc.

Each participant has its own local **state**.

Participants exchange **messages** over communication **links**.

Processes access the time through a **clock**, which can be a logical or physical (a.k.a. wall) clock.
Logical clocks include monotonic clocks; wall clocks report on time as it exists in the physical world.

The core difficulties of distributed systems stem from the fact that the components are not physically located in the same place, and communicate through links which are expected to be nondeterministically slow and unreliable.
Communication may be delayed, reordered, fail completely, etc.; processes may be slow, hung, crashed or unresponsive.

Distributed systems shares concepts in common with concurrent programming, though concurrent programming cannot be applied directly to distributed systems as concurrent programming is assumed to be distributed across the cores of a single CPU, whose links are presumed to be reliable and low-cost to communicate across.

Distributed systems algorithms coordinate local and remote state, transitioning between states (a.k.a. steps or phases) which account for the expectation of unreliable networks and system components failures.

Distributed algorithms serve various purposes:

* **Coordination**: supervising the actions and behaviors of other processes
* **Cooperation**: multiple processes relying on each other to complete their tasks
* **Dissemination**: processes cooperating to reliably spread information to all interested parties
* **Consensus**: reaching agreement among multiple processes
