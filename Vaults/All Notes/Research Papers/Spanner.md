_____
**Created**: 30-08-2024 08:24 pm
**Status**: Completed
**Tags**: #DistributedSystem [[Database]] [[File System]] [[Distributed Database]] [[Transactions]]
**References**: [[Google Spanner.pdf|Google Spanner]]
______

### Introduction
Spanner is a globally distributed database built and designed and deployed at Google. It uses replication for global availability and geographic locality. Clients automatically failover between replicas. Spanners main focus lies in managing cross-datacenter replicated data. 

Spanner has evolved from a [[Big Table]] like versioned key-value store to a temporal multi-versioned database. Spanner stores data as schematised semi-relational tables. The data is versioned and is timestamped with its commit times automatically. Garbage collection policies are put in place and are configurable to handle deletion and flushing of older versions of data. It provides an SQL based query language and supports general purpose transactions. It provides externally consistent reads and writes, and provides globally-consistent reads across the database at a timestamp.

### Implementation
Spanner deployment is called *universe*. Spanner is organised as a set of *zone*; each zone is the rough analog of a deployment of [[Big Table]]. Zones are the unit of administrative deployment. They indicate the set of locations across which data can be replicated.

The spanner deployment structure image:
![[Spanner_Server_Organisation.png|500]]
A zone consists of a *zonemaster* and *several* spanservers. The zonemaster is tasked with assigning data to a spanserver, which in turn is tasked with serving this data to the clients.
The *universemaster* and *placement* master have only one instance per deployment. Universemaster gives the status of the deployment while *placement driver* as the name suggests handles the moving of data.

#### Spanserver Software Stack
Spanserver manages data in the form of `tablets`. Tablets saves data in the format `(key:string, timestamp:int64) → string`. Assigning a timestamp to the data is what enables spanserver to have versioning. The tablet state is stored in a B-Tree like files and a write-ahead-log on Colossus. Each tablet implements a Paxos state machine which stores logs and metadata. Implements a consistent replicated bag of mappings. The Paxos applies writes in order. Paxos group is just a set of replicas.

Replicas will have a leader and each spanserver within that leader implements a lock table (contains state for [[Two-Phase Locking]]) for concurrency control and a transaction manager for supporting [[Distributed Transactions]].

![[Spanserver_Stack.png|500]] 

#### Directories and Placement
A *`directory`* is a set of contiguous keys that share a prefix, also giving users a semblance of control over the locality of the keys. *`directory`* is the unit of data placement, all data within a directory share. the same replication configurations. *`Directory`* is also the smallest unit an application can configure. To make this a bit more clear Administrators are given control over two things:
1. Number and types of replicas to create
2. Where the replicas are stored geographically
If a directory grows too big we may shard it into multiple fragments across one or more Paxos groups.

#### Datat Model
Spanner exposes the following three:
- Data model based on schematised semi-relational tables
- Query language
- General purpose transactions

An application (universe) has one or more databases. Each database in turn contains unlimited number of schematised tablets. 
Every table is required to have an ordered set of at least one primary-key column.

#### True Time
For more details refer to [[True Time]].

| Method       | Returns                              | What it does                                                                                            |
| ------------ | ------------------------------------ | ------------------------------------------------------------------------------------------------------- |
| TT.now()     | TTinterval: \[earliest, latest\]     | Returns a TTinterval that is guaranteed to contain the absolute time during which TT.now() was invoked. |
| TT.after(t)  | true if t has definitely passed.     | Checks whether we've passed a certain timestamp t; takes the error bounds into account.                 |
| TT.before(t) | true if t has definitely not arrived | Checks whether we've a certain timestamp t is yet to occur; takes the error bounds into account.        |
### Concurrency Control

| Operation                                | Concurrency Control | Replica Required                           |
| ---------------------------------------- | ------------------- |:------------------------------------------ |
| Read-Write Transaction                   | pessimistic         | leader                                     |
| Read-only Transaction                    | lock-free           | leader for timestamp; any replica for read |
| Snapshot Read, client-provided timestamp | lock-free           | any                                        |
| Snapshot Read, client-provided bound     | lock-free           | any                                        |
To elaborate a snapshot read a read in the past. 
Transactional read writes use [[Two-Phase Locking]]. Inside of each, Paxos group Spanner assigns timestamps to Paxos writes in monotonically increasing order, this is true even across leaders. To enforce this across leaders, the leader may assign a timestamp only if the timestamp is within its lease period. 

#### Reads at a Timestamp
Each replica has a timestamp called t$_{safe}$ which is the maximum timestamp at which a replica is up-to-date, implying that a read can be processed if t <= t$_{safe}$ .

A read only transaction executes in two phases:
1. Assign a timestamp s$_{read}$ 
2. Execute the transaction's read as snapshot reads at s$_{read}$.

If t$_{safe}$ has not advanced sufficiently then it blocks the read.

#### Read Write Transactions
The writes uncommitted transactions are buffered at the client until they are committed, and they are not assigned a timestamp until then. Reads require the data read to have a timestamp, implying that uncommitted write data would not be read by it. 

In read-write transactions, the reads use [[Wound Wait]] to avoid deadlocks. During the client-transaction, the client sends keep-alive messages to the participant leaders to avoid them timing out the transaction. Once all the reads have completed and writes buffered it initiates the [[Two Phase Commits]]. The client chooses a coordinator group and sends relevant information (coordinator Id, buffered reads, commits) etc. to the involved leaders. Non coordinator leaders first acquire write locks and choose a prepare timestamp (larger than any previous ones assigned to previous transaction, this helps preserve monotonicity).

The coordinator leader also first acquires the write locks and skips the preparer phase, instead it chooses a timestamp for the transaction after hearing form all other participating leaders. This timestamp once again has to be greater than or equal to all the prepare timestamps of the leaders, greater than `TT.now().latest` and greater than any timestamp it has assigned to previous transactions to preserve monotonicity. 

The coordinator waits until `TT.after(transaction_timestamp)` before committing to ensure that the *transaction_timestamp* was indeed in the past. After this wait it(coordinator) sends the *transaction_timestamp* to all the participants which then logs the outcome of the commit through Paxos. All participants apply the same **timestamp** then release their acquired locks.

#### Schema-Change Transactions
Schema-Change transactions are non-blocking variant of the standard transaction. The transaction is given a timestamp in the future in the prepare phase to avoid affecting any concurrent activity. The reads and writes that depend on the schema will check time t and wait if t if after the selected transaction time for schema change.
