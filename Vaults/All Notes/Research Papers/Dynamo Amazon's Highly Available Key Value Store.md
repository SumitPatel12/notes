_____
**Created**: 28-03-2025 10:52 pm
**Status**: In Progress
**Tags**: #Database #Key_Value_Store [[Database]]
**References**: [[Amazon Dynamo.pdf]]
______
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
