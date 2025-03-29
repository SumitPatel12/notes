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

![[Pasted image 20250329144714.png]]

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
