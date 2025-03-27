_____
**Created**: 25-09-2024 09:13 am
**Status**: Completed
**Tags**: #Data_Processing [[Data Processing]]
**References**: [[Google Mapreduce.pdf]]
______

### What is MapReduce?
MapReduce is a programming model used for processing large amounts of data. In its implementation we generally specify 
- ***Map Function***: It takes in a set of key/value pairs and generates an intermediate set of key-value pairs.
- ***Reduce(r) Functions***: It takes in the output (intermediate key-value pairs) and merges all the values that are associated with the same key. 
**Note that both of these above defined functions are designed and implemented by the user.**
There is a MapReduce library that groups together the intermediate key-value pairs.

### Programming Model
Consider the following following example for counting the number of occurrences of a word across a large amount of documents.
```python
map(String key, String value):
	#key: document name
	#value: document contents

	for each word w in value:
		EmitIntermediate(w, "1")

reduce(String key, Iterator values):
	#key: a word
	#values: a list of counts

	int result = 0;
	for each v in values:
		result += ParseInt(v)
		Emit(AsString(result))
```
The *mapper* function returns the word with a count one.
The *reducer* function sums all of the occurrences of that word.
On top of these functions the user is required to write a *MapReduce Specification File*. This file includes the name of input, output files and some optional tuning parameters. The specification file obeject is passed to the MapReduce function at the time of invocation.

### Implementation
MapReduce implementations may differ based on the requirements and available resources.
This instance of MapReduce has the following resources:
1. Dual Processor x86 machines, Linux, 2-4 GB of memory.
2. Networking hardware: 100Mb/s or 1Gb/s, slower in actual cases.
3. Clusters contain 100s and 1000s of machines meaning failures are a common occurrence.
4. Storage is cheap IDE disks attached to machines. It leverages a file system that provides features such as replication.
5. For running a task the user sends a request to a scheduling system, it then assigns the job to a set of available machines within the cluster.

#### Execution Overview
The input data is partitioned in M splits and distributed across machines. Each split can be process in parallel by different machines. Similarly, the for the reduce function the intermediate keys are partitioned into R different partitions. 

The execution process is as follows:
1. The MapReduce library in the user program splits the input files into M pieces (16-64MB). Afterwards it starts a copy of the program on a cluster of machines.
2. One machine in the cluster is the master and other are workers. The master is in charge of assigning idle machines either a map or reduce task.
3. For a map task the machine reads its given split, parses the key/value pairs form the input data and passes each pair to a user-defined map function. The intermediate key/value pairs produced during this process are *buffered in memory*.
4. Periodically, the buffered pairs are written to a local disk, partitioned into **R partitions** using the partitioning function. The location of these pairs are then sent to the *master* which in turn sends them to the *reducer* workers.
5. When a reduce worker gets the location of these pairs form the master, it uses [[Remote Procedure Calls (RPCs)]] to read the buffered data from the local disk of the map worker. After reading all of the intermediate data the reduce worker sorts the keys (this groups instances of the same key together).
6. The reduce worker then iterates over the sorted intermediate keys and passed each unique key and its corresponding values to a user defined reduce function. The output of the reduce function is appended to an output file corresponding to the current partition.
7. After all of the map and reduce tasks are done, the master wakes the user program, which in turn returns the output to the use code.

![[Execution_Overview.png|1000]]

#### Master Data Structures
For each map and reduce task the master stores the following
- State: *idle, in progress, completed*.
- *Identity* of the worker machine for non-idle tasks.
- For each completed map task, the master stores the locations and sizes of the *R partitions/regions* created by the map task.

#### Fault Tolerance
##### Worker Failure
Mater sends heartbeat to its workers and if a response is not received within a certain time the worker is marked as failed. Any map tasks *completed* by the worker are marked idle and are available for rescheduling. Same is true for any *in-progress* map or reduce tasks.
The completed map tasks are required to be re-executed because their output is stored in a local disk which is inaccessible due to the worker failure. On the other hand **reduce tasks need not be retried** because they store their output on a *global file system*.
Whenever a map task is re-executed due to a worker failure, all the workers running the reduce task are informed of it. Any reduce task that has not read data form the failed worker will now read data from the worker that is reassigned the map task.

##### Master Failure
In the context of this implementation *master failure* aborts the MapReduce task. Depending on your need you could checkpoint the **data structures** in the master and spawn a new master from the latest persisted checkpoint in case of failure.

##### Semantics in the Presence of Failure
Each in-progress task writes it output to private temporary files. A reduce task produces one such file, while a map task produces R such files (one per reduce task). When the map task completes the worker sends a message to master including the temp file names. The master stores them in its data structure, note that a duplicate message i.e. a message from an already completed map task is discarded.

For most of the cases the map and reduce functions are [[Deterministic]], thus. the MapReduce task executes as if sequential because of the semantics. 

> In the presence of non-[[Deterministic]] operators, the output of a particular reduce task R1 is equivalent to the output for R1 produced by a sequential execution of the [[Non-Deterministic]] program. However, the output for a different reduce task R2 may correspond to the output for R2 produced by a different sequential execution of the [[Non-Deterministic]] program. Consider map task M and reduce tasks R1 and R2. Let e(Ri) be the execution of Ri that committed (there is exactly one such execution). The weaker semantics arise because e(R1) may have read the output produced by one execution of M and e(R2) may have read the output produced by a different execution of M.

#### Locality
The input data (Managed by [[Google File System (GFS)]]) is stored in the local disks on the machines. This helps conserve network bandwidth. [[Google File System (GFS) |GFS]] divides each file into 64MB blocks and has copies of it stored on different machines. The master is aware of the block locations and assigns tasks such that worker that has the data locally or incase such a worker is unavailable a worker closest to it gets assigned the task. This helps reduce network bandwidth as most of the tasks would require local reads only.

#### Task Granularity
The map task is divided into **M** tasks and reduce into **R** tasks. Ideally **M** and **R** would be greater than machines on the cluster to improve load balancing, and speed up recovery in case of failure. 

Tasks master tracks:
```
O(M + R) scheduling decisions.
O(M * R) state in memory.
	The memory usage per map/reduce task pair is close to 1 byte.
```
Note that R is also constrained by the user as output of reduce tasks is divided among separate files. In practice M is chosen such that each individual files is between 16MB to 64MB in size. R is chosen as small multiple of the number of machines estimated to be used.

#### Backup tasks
The MapReduce task is prone to lengthening in case one of the *map or reduce tasks* takes too long to complete. To avoid such situations whenever the MapReduce task is close to completion the master schedules *backup executions* for remaining *in-progress* tasks. The task is marked as complete when one of the primary or the backup tasks complete.

### Refinements
#### Partitioning Function
The user specified the number of output files/reduce tasks they desire *R*. This partitioning is done using a partitioning function on the *intermediate key*. By default the partitioning is done using hash functions i.e. `hash(key) mod R`. In some cases the user may want to provide their own hashing function, this is supported. For e.g. `hash(Source(data)) mod R` would have the data coming form the same source to be in the same output file.

#### Ordering Guarantees
Within a given partition, the intermediate keys are processed in increasing order, resulting in sorted output files per partition. This is useful as in most of the cases users find it useful to have sorted output files.

#### Combiner function
Combiner functional is an optional thing that partially combines the intermediate key/value pairs before passing them over the network to a reducer function. It is executed on each machine performing a map task. Typically the combine and reducer functions use the *same code*. The difference being that combiner function output is written to an intermediate output file which then passed to a reducer function, while the output of the reducer function is written to an output file.
**Partial combining significantly speeds up certain types of MapReduce tasks.**

#### Side Effects
Sometimes the user would want an extra output file to be generated during the MapReduce task, they can do so via the application writer. It generally writes to a temporary file and atomically renames it once it is fully generated. 
Tasks that produce multiple output files with cross-file consistency need to be [[Deterministic]].

#### Skipping Bad Records
There are cases where we want to skip certain records that crash the MapReduce task [[Deterministic |Deterministically]]. MapReduce library provides an optional mode of execution where the library detects such [[Deterministic]] crashes and skips them.
For its implementation each worker has a signal handler that sends a signal with the sequence number in case of segmentation violations or bus errors. The master stores this in a global variable and skips subsequent executions on that record if it has been marked for multiple such signals.

#### Counter
In certain scenarios the user would like to know the count of certain events occurring throughout the map and reduce tasks. To implement this the user creates a named counter and increments it as required in the Map and/or Reduce tasks. The counter values form the individual worker machines  are periodically propagated to master (piggybacked on the ping response). Master aggregates the counter values and makes sure they are not updated for duplicate executions avoiding double counting. 

### Performance
#### Cluster Configuration
All of the performance metrics were tested on a cluster consisting of:
- 1800 machines each with 
	- 2GHz Intel Xeon processors with Hyper Threading enabled
	- 4GB of RAM
	- Two 160GB IDE disks
	- Gigabit Ethernet link
- Machines were arranged in a two level tree-shaped switched network with around 100-200Gbps bandwidth at the root.
- All machines were in the same facility, so the roundtrip time was less than 1ms.
- Out of the 4GB RAM available to each machine around 1-1.5GB was consumed by other processes running on the machine.

#### Grep
Grep program scans through 10$^{10}$  100 byte records searching for a three-character pattern. The input is split into 64MB pieces (M = 15000), output is just one file (R = 1). The program takes around 150 seconds to complete.
**Performance does not seem like a concern right now not including it.**

### Large-Scale Indexing
In the context of the paper the most significant use of this MapReduce was a complete rewrite of the production indexing system that produces the data structures used for the Google web search service.
The indexing system takes a large set of documents that have been retrieved by the crawling system, stored as a set of [[Google File System (GFS)]] files. Raw contents of these files are around 20TB.
The indexing process then runs as a set of 5-10 MapReduce operations. The benefits of using MapReduce are listed below:
- The indexing code has reduced complexity, because a lot of heavy lifting such as fault tolerance, parallelisation and distribution are abstracted by the MapReduce Library.
	- For instance the code for one phase of computation dropped from *3800+ lines to around 700 lines* when expressed as MapReduce task.
- The performance of MapReduce was good enough to have conceptually unrelated computations separate, instead of mixing them together to avoid multiple passes over the data. This makes it easy to change the process.
- Indexing process becomes easier to operate as MapReduce takes care of machine failures and network hiccups. It also makes it easy to improve performance as it would simply need to assign some more machines to the indexing cluster.