_____
**Created**: 28-03-2025 10:52 pm
**Status**: In Progress
**Tags**: #Database #Key_Value_Store [[Database]]
**References**: [[Amazon Dynamo.pdf]]
______
### Things To Look Into
- Hierarchical Namespaces
- Peer to Peer Networks
- Multi-hop routing
- Consistent Hashing

### Initial Read Notes
- It sacrifices *consistency under certain failure scenarios* in favour of high-availability.
- Amazon uses highly decentralized, loosely coupled, service oriented architecture consisting of hundreds of services, which requires a highly available storage technology.
- Dynamo provides a `primary-key` only interface to meet requirements of certain services.
- Data is partitioned and replicated using `consistent hashing`, consistency is facilitated by object versioning. 
- The consistency among replicas during updates is maintained by a quorum-like techique and a decentralized replica synchronisation protocol.
- It uses a gossip based distributed failure detection membership protocol.
- It is completely decentralized system and nodes can be added and removed from Dynamo without requiring any manual partitioning or redistribution.
- Each service that uses Dynamo runs its own Dynamo instance.
- **`System Assumptions And Requirements:`**
	- *Query Model:* Simple read and write operations to a data item that is uniquely identified by a key. State is stored ins `blobs` and no operation spans multiple data items.
	- *ACID Properties:* Dynamo targets applications that operate with weaker consistency if it results in higher availability. Dynamo **does not provide any isolation guarantees and permits only single key updates.**
	- *Efficiency:* Services should be able to configure Dynamo such that they consistently achieve their latency and throughput requirements. The tradeoffs are in:
		- Performance
		- Cost efficiency
		- Availability
		- Durability Guraantees

#### Service Level Agreements
Include's the client’s expected request rate distribution for a particular API and the expected service latency under those conditions.

![[Pasted image 20250329144714.png|500]]

SLAs are expressed and measured at the 99.9th percentile of the distribution. The choice for 99.9% over an even higher percentile was made after taking into consideration the cost-benefit analysis which indicated that above this there was significant cost overhead for very small performance gains.

One of the main design considerations for Dynamo is to provide the services control over their system properties such as durability and consistency, and to let them make their own tradeoffs between functionality, performance and cost-effectiveness.

#### Design Considerations
Dynamo is designed to be an eventually consistent data store.
Conflict resolution for eventually consistent can be done at two places, either read or write. The creators of Dynamo go with conflict resolution on reads, since their requirements put heavy emphasis on `always writable`. Consider not being able to update your cart contents because the conflict resolution kept failing amidst network or server failures vs not being able to view some items. 🤷‍♂

Dynamo primarily leaves the conflict resolution to the application since it is more aware of the data schema and can decide best what works for it's specific use case. If the application dev does not want to write a conflict resolution strategy then it can push it to the Data store to decide which method to use (it generally chooses a simple strategy such as last write wins or something.)

- `Incremental Scalability:` Dynamo should e able to scale out tone storage host at a time with minimal impact on both the operator of the system and the system itself.
- `Symmetry:` Every node in Dynamo should have the same set of responsibilities as its peers; there should be no distinguished node or nodes that take special roles or extra set of responsibilities.
- `Decentralization:` Favour decentralized peer-to-peer techniques over centralized control. 
- `Heterogeity:` The work distribution must be proportional to the capabilities of the individual servers.

*Kind of TLDR;*
- Dynamo is targeted mainly at applications that need **always writable** data store.
- It is built for an infrastructure within a single administrative domain where all nodes are assumed to be trusted.
- Applications that use Dynamo do not generally require hierarchical namespaces or complex relational schemas.
- Built to provide consistent performance at the 99.9th percentile, read and writes for these percentiles to be within a couple 100 ms. To meet this they had to avoid routing requests through multiple nodes.
- It can be characterised as a zero-hop DHT, where each node maintains enough routing information locally to route a request to the appropriate node directly.

#### System Architecture

| Problem                            | Technique                                               | Advantage                                                                                                         |
| ---------------------------------- | ------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| Partitioning                       | [[Consistent Hashing]]                                  | Incremental Scalability                                                                                           |
| High Availability for writes       | Vector Clocks with reconciliation during reads          | Version size is decoupled from updates                                                                            |
| Handling temporary failures        | Sloppy Quorum nad hinted handoff                        | Provides high availability and durability guarantee when some of the replicas are not available.                  |
| Recovering from permanent failures | Anti-entropy using Merkle trees                         | Synchronizes divergent replicas in the background.                                                                |
| Membershiop and failure detection  | Gossip-based membership protocol and failure detection. | Preserves symmetry and avoids having a centralized registry for storing membership and node liveness information. |

##### System Interface
Dynamo stores objects associated with the key through a simple interface for which it exposes two operations:
1. `get(key):` 
	- Locates the object (which the key represents) across all replicas in the storage system.
	- Returns a single value or list of conflicting values and a context.
2. `put(key, context):` 
	- Determines where the replicas of the object should be placed based on the associated key, and writes the replicas to disk.
	- *Context* encodes system metadata about the object that is opaque to the caller and includes information such as the version of the object.
	- The context info is stored along with the object so that the system can verify the validity of the context object supplied in the put request.
Dynamo treats both key and object supplied by the caller as an opaque array of bytes and applies a MD5 hash on the key to generate a 128-bit identifier, which is used to determine the storage nodes that are responsible for serving the key.

##### Partitioning Algorithm
They use consistent hashing to distribute the load across multiple storage hosts.
The single position assignment of each node for consistent hashing did not sit well with the SLAs and heterogeneity of their system, so they decided to use a variant of consistent hashing:
- They introduced the concept of `virtual node`.
- Whenever a physical/actual node is added to the Dynamo system it gets assigned multiple positions/tokens in the ring.
- It presents the following advantages:
	- If a node becomes unavailable for some reason, it is evenly dispersed across the remaining available nodes.
	- When it gets back up again, or a new node is added in its place, or the system adds a new node, it accepts a roughly equivalent amount of load from each of the other available nodes.

##### Replication
- Each data item is replicated at N hosts; N is configured before spawning the instance. 
- Each key k, is assigned to a coordinator node which is in charge of the replication the data items that fall within its range.
- It persists the key to itself and N-1 preceding nodes in clockwise direction. Essentially each node is getting values between its Nth predecessor and itself.
- The list of nodes that are responsible for storing a particular key is called the preference list.
	- To account for node failures, the preference list contains more than N nodes.
	- Since they use virtual nodes it is possible that the first N successor positions for a particular key may be owned by less than N distinct physical nodes.
	- To address that the preference list skips positions in the ring, ensuring that only distinct physical nodes make up the list.

##### Data Versioning
- As established earlier, Dynamo is an eventually consistent data store, meaning updates are to be propagated to all replicas asynchronously.
- A `put` call may return before the update is applied to all replicas.
- Meaning subsequent `get` call can possibly return a previous version of the data. There are bounds on update propagation times but under certain circumstances updates may be delayed for extensive period of times.
- It treats the result of each modification as a new immutable version of the data.
	- New versions subsume older version(s) for the most part, but there are times when branching may occur. 
	- This can happen when there is a failure with concurrent updates, leading to conflicting versions, for thees the system cannot reconcile the multiple versions of the same object and the client is required to perform the reconciliation to collapse the multiple branches back to one.
	- *Merging* different versions of a shopping cart is one of the examples. This means no data is ever lost on the `add to cart` but deleted items might make a new appearance.
- *Network partitioning paired with node failures* can result multiple different versions of an item. This means the system has to acknowledge the possibility as well and guard around that.
- To do this Dynamo uses vector clocks. They capture the causality between different versions of the same object.
	- A vector clock is effectively a list of `(node, counter)` pairs, one vector clock is associated with every version of every object. 
	- This means we can determine if two versions of an object are parallel or causal by comparing the vector clocks.
		- If the counters of the first object's clock are <= all of the nodes in the second clock, then the first is an ancestor of the second and can be forgotten.
		- Otherwise the two versions are considered to be conflicting.
- When updating the context needs to be passed which tells us which version to update. The context is obtained from a prior read and contains the vector clock info.
- When processing a read request Dynamo has access to multiple branches that cannot be syntactically reconciled, it will return all the objects return all the objects at the leaves, with the corresponding version information in the context. And update using this context is considered to have reconciled the divergent versions and the branches are collapsed into a single new version.

##### Execution of get() and put()
There are two strategies a client can use to select a node:
1. Route its request through a generic load balancer that will select a node based on load information.
2. Use a partition-aware client library that routes requests directly to the appropriate coordinator nodes.
First has the benefit of the application not needing to to include any Dynamo related code, second one of-course gives better latency.

Normally first among the top N nodes in the *preference list* would serve as a coordinator. If the request are received through a load balancer, requests to access a key may be routed to any random node nit the ring. In that case the node who received the request does not coordinate it if it is not in the top *N* of the keys preference list. Instead the at node will forward it to the first among the top *N* nodes in the preference list.

Dynamo uses a quorum like protocol for consistency. It has two key configurable: *R* and *W*; R being the minimum number of nodes that must participate in a successful read operation and W being the minimum number of nodes that must participate in a write operation. Setting R and W such that **R + W > N** leads to a quorum-like system.
In this model the latency of get operation is dictated by the slowest of the R replicas. For this reason, T and W are usually configured to be less than N, to provide better latency.

For put:
- Coordinator generates the vector clock for  the new version and writes it locally.
- Then its sends the new version to the N highest-ranked reachable nodes.
- If at least W-1 respond then the write is considered successful.

For get:
- Coordinator requests all existing versions of dat for that key from the N highest-ranked reachable nodes in the preference list, then waits for R responses before returning the result to the client.
- In case of multiple versions it returns all the versions it deems to be causally unrelated.
- The divergent versions are then reconciled and that version superseding the current version is written back.

##### Handling Failures: Hinted Handoff
It uses a `sloppy quorum`; all rads and writes are performed on the first *N healthy* nodes from the preference list, not always the first N nodes encountered while walking the consistent hashing ring.

Say node A fails and a write operation that should have been routed to A is instead routed to D. Then the request to D will have a hint in the metadata that would tell it that the request was originally meant for node A. So, it will keep that reqs content in a separate local store which is scanned periodically. When node A gets back up, the scan will try to send the relevant replicas to A and on successful transfer the data on D would be safe to delete.

Dynamo is configured to replicate across multiple data centers, to avoid down time in case a data center is down for any reason. To do this the preference list of a key is constructed such that the storage nodes are spread across multiple data centers.

##### Handling Permanent Failures: Replica Synchronization
There are times when the hinted replicas become unavailable before the replica can make it back to the intended node. An anti-entropy protocol is deployed to combat such cases.

To detect inconsistencies between replicas faster and to minimize the amount of Data transferred, it uses **Merkle trees**. It is a has tree where leaves are hashes of the values of individual keys. Parent nodes higher in the tree are hashes of their respective children. The advantage being each branch can be checked independently without requiring nodes to download the entire tree or the entire data set. It also helps reducing the amount of data transfer for checking inconsistencies since if the hash values of root of two trees are equal then it means the leaf values are also equal, meaning no synchronization is required.

Each node maintains a separate Merkle tree for each key range it hosts. This allows easy comparisons for whether key rage are up-to-date. On different root values the tree is traversed to find the differences. The downside being when a node joins the system the tree would need to be recalculated.

##### Membership and Failure Detection
###### Ring Membership
When a node starts for the first time, it chooses a set of tokens/virtual nodes and maps nodes to their respective token sets. The mapping is persisted on disk and initially contains only the local node and token set. The mappings stored at different Dynamo nodes are reconciled during the same communication exchange that reconciles the membership change histories. Thus, partitioning and placement information also propagates via gossip-based protocol and each storage node is aware of the token ranges handled by its peers.

###### External Discovery
The mechanism described above could temporarily result in a logically partitioned Dynamo ring. For example, the administrator could contact node A to join A to the ring, then contact node B to join B to the ring. In this scenario, nodes A and B would each consider itself a member of the ring, yet neither would be immediately aware of the other. To prevent logical partitions, some Dynamo nodes play the role of seeds. Seeds are nodes that are discovered via an external mechanism and are known to all nodes. Because all nodes eventually reconcile their membership with a seed, logical partitions are highly unlikely. Seeds can be obtained either from static configuration or from a configuration service. Typically seeds are fully functional nodes in the Dynamo ring.

###### Failure Detection
Node A may consider node B failed if node B does not respond to node A's messages, (even if B is responsive to node C's messages). In the presence of a steady rate of client request generating internode communication in the Dynamo ring, a node A quickly discovers that a node B is unresponsive when B fails to respond to a message; Node A then uses alternate nodes to service requests that map to B's partitions; A periodically retries B to check for the latter's recovery. IN the absence of client requests to drive traffic between two nodes, neither node really needs to know whether the other is reachable and responsive.

##### Adding/Removing Storage Nodes
When a new node is added to the system, it gets assigned a number of tokens that are randomly scattered on the ring. For every key range that is assigned to node X, there may be a number of nodes that are currently in charge of handling keys that fall within its token range. Due to that allocation to X, some existing nodes no longer have that key range so those keys are transferred to X.
Similarly when a node is removed it sends the keys it was managing to relevant nodes that would handle the key range when it leaves.

#### Implementation
Each node has the following three main components:
1. Request Coordination
2. Membership and failure detection
3. Local Persistence Engine

The local persistence component allows for different storage engines to be plugged in, like Berkeley DB Transactional Data Store, MySQL, and an in-memory buffer with persistent backing tree. These choices are made based on the applications access patterns and object sizes.

The request coordination component is built on top of an event- driven messaging substrate where the message processing pipeline is split into multiple stages similar to the SEDA architecture. All communications are implemented using Java NIO channels. The coordinator executes the read and write requests on behalf of clients by collecting data from one or more nodes (in the case of reads) or storing data at one or more nodes (for writes). Each client request results in the creation of a state machine on the node that received the client request. The state machine contains all the logic for identifying the nodes responsible for a key, sending the requests, waiting for responses, potentially doing retries, processing the replies and packaging the response to the client. Each state machine instance handles exactly one client request. For instance, a read operation implements the following state machine: (i) send read requests to the nodes, (ii) wait for minimum number of required responses, (iii) if too few replies were received within a given time bound, fail the request, (iv) otherwise gather all the data versions and determine the ones to be returned and (v) if versioning is enabled, perform syntactic reconciliation and generate an opaque write context that contains the vector clock that subsumes all the remaining versions. For the sake of brevity the failure handling and retry states are left out.

During read if stale versions were returned the the coordinator node updates those nodes with the more recent copy thus repairing them.

Write requests can be handled by any of the top N nodes in the preference list. Since each write usually follows a read operation, the coordinator for a write is chosen to be the node that replied fastest to the previous read operation which is stored in the context information of the request. This optimization enables them to pick the node that has the data that was read by the preceding read operation thereby increasing the chances of getting `read-your-write` consistency.

#### Experiences and Lessons Learned
Main patterns in which Dynamo is used:
1. *Business logic specific reconciliation*
2. *Timestamp based reconciliation*
3. *High performance read engine*

The main advantage of Dynamo is that its client applications can tune the values of N, R and W to achieve their desired levels of performance, availability and durability. For instance, the value of N determines the durability of each object. A typical value of N used by Dynamo’s users is 3.
The common (N,R,W) configuration used by several instances of Dynamo is (3,2,2). These values are chosen to meet the necessary levels of performance, durability, consistency, and availability SLAs.

##### Balancing Performance and Durability
Typical SLA required of services leveraging Dynamo is that the 99.9th percentile read and writes execute within 300ms. 

While this level of performance is acceptable for a number of services, a few customer-facing services required higher levels of performance. For these services, Dynamo provides the ability to trade-off durability guarantees for performance. In the optimization each storage node maintains an object buffer in its main memory. Each write operation is stored in the buffer and gets periodically written to storage by a writer thread. In this scheme, read operations first check if the requested key is present in the buffer. If so, the object is read from the buffer instead of the storage engine.
Obviously, this scheme trades durability for performance. In this scheme, a server crash can result in missing writes that were queued up in the buffer. To reduce the durability risk, the write operation is refined to have the coordinator choose one out of the N replicas to perform a “durable write”. Since the coordinator waits only for W responses, the performance of the write operation is not affected by the performance of the durable write operation performed by a single replica.

##### Client-driven or Server-driven Coordination
Dynamo has a request coordination component that uses a state machine to handle incoming requests. Client requests are uniformly assigned to nodes in the ring by a load balancer. Any Dynamo node can act as a coordinator for a read request. Write requests on the other hand will be coordinated by a node in the key’s current preference list. This restriction is due to the fact that these preferred nodes have the added responsibility of creating a new version stamp that causally subsumes the version that has been updated by the write request. Note that if Dynamo’s versioning scheme is based on physical timestamps, any node can coordinate a write request.

An alternative approach to request coordination is to move the state machine to the client nodes. In this scheme client applications use a library to perform request coordination locally. A client periodically picks a random Dynamo node and downloads its current view of Dynamo membership state. Using this information the client can determine which set of nodes form the preference list for any given key. Read requests can be coordinated at the client node thereby avoiding the extra network hop that is incurred if the request were assigned to a random Dynamo node by the load balancer. Writes will either be forwarded to a node in the key’s preference list or can be coordinated locally if Dynamo is using timestamps based versioning.