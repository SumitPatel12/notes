_____
**Created**: 23-03-2025 04:32 pm
**Status**: In Progress
**Tags**: #System_Performance #System_Performance_Methodologies
**References**: 
______
### Terminology
- *IOPS:* Input/output operations per second. Measures the rate of data transfer operations. For disk I/O it just refers to the read and writes per second.
- *Throughput:* The rate of work performed. Changes based on context, for e.g. DBs use operations per second or transactions per second. Communication lingo might refer to it as data rate or bits/bytes per second.
- *Response Time:* The time for an operation to complete. Includes waiting time and the time it takes to transfer the result to the user.
- *Latency:* The amount of time a request waits to be service. In some contexts it might just mean the entire time for an operation. I know thats ambiguous.
- *Utilization:* It's the measure of how much time it was actively working on a task in a given time interval. For storage devices it may just be the amount of storage used.
- *Saturation:* How much work/tasks are queued that cannot be addressed right away.
- *Bottleneck:* Any resource that limits performance.
- *Workload:* Amount of work or load assigned to a system. For DBs it might be the queries it has to handle, for File systems it might be the amount of reads/writes it needs to do along with maintaining open descriptors and connections.
- *Cache:* Just normal cache. Buffer to store frequently accessed data to avoid calling slower storage mediums.

### Models
#### System Under Test (SUT)
Just be aware of the fact that the resulting performance of SUT are affected by interference, this includes events such as system activity, other system users, and any other daemons or workloads running on the same machine as the SUT. This might seem like not that big of a concern for local development since you can ensure nothing interferes with your testing efforts, but on a cloud/serverless system where you share the same space with multiple guest tenants, and with no information of the physical host system.

Other thing with modern environments is that they can be composed of several worked components (think microservices), including load balancers, proxy servers, web servers, application servers, db servers, storage systems, etc. Gauging performance in presence of interference from all of these sources becomes a very challenging task.
#### Queueing System
Model components and/or resources as queueing system, making the task of predicting their performance under different situations easier based on the model.
![[Simple Queuing Model.excalidraw|500]]


### Concepts
1. `Latency:` Simply put latency is the time spent waiting before an operation is performed.
2. `Time Scales:`

| Event                                     | Latency              | Scaled       |
| ----------------------------------------- | -------------------- | ------------ |
| 1 CPU Cycle                               | 0.3ns                | 1s           |
| L1 Cache Access                           | 0.9ns                | 3s           |
| L2 Cache Access                           | 3ns                  | 10s          |
| L3 Cache Access                           | 10ns                 | 33s          |
| Main Memory Access (DRAM, from CPU)       | 100ns                | ymin         |
| Solid-state disk I/O (flash memroy)       | 10-100 micro seconds | 9-90 hours   |
| Rotational Disk I/O                       | 1-10ms               | 1-12 months  |
| Internet: San Francisco to New York       | 40ms                 | 4 years      |
| Internet: San Francisco to United Kingdom | 81ms                 | 8 years      |
| Lightweight hardware virtualization boot  | 100ms                | 11 years     |
| Internet: San Francisco to Australia      | 183ms                | 19 years     |
| OS virtualization system boot             | < 1s                 | 105 years    |
| TCP timer based retransmit                | 1-3s                 | 105-317 yers |
| SCSI command time-out                     | 30s                  | 3 millennia  |
| Hardware virtualization system boot       | 40s                  | 4 millennia  |
| Physical system reboot                    | 5min                 | 32 millennia |

3. `Trade-Offs:` 
	- Common tradeoffs are between *high-performance, on-time and inexpensive*. Often people choose inexpensive and on-time and leave the performance for later.
	- The decisions made early in the product lifecycle can lead to significant performance bottlenecks.
	- Tunable params often come with tradeoffs:
		- *File system record size or blcok size:* 
			- Small record sizes, will perform better from random I/O workloads and utilise the file system cache better.
			- Larger record sizes will improve streaming workloads, including file system backups.
		- *Network buffer size:* Small buffer sizes will reduce the memory overhead per connection, helping the system scale. Large sizes will improve network throughput.
4. `Tuning Efforts:` Example tuning targets
	- Application
	- Database
	- System calls
	- File system
	- Storage