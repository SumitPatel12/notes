_____
**Created**: 23-03-2025 08:19 pm
**Status**: In Progress
**Tags**: #Operating_System #System_Performance 
**References**: Research Paper of the Same name.
______

### Introduction
For a *multitasking system*, context switch refers to the switching of the CPU from one process or thread to another. Context switch makes multitasking possible, but with some system overhead.

The cost of context switch comes from several aspects:
- **Direct Cost:**
	- The processor registers need to be saved and restored.
	- The OS kernel code - scheduler - must execute.
	- The [TLB](https://www.scaler.com/topics/tlb-in-os/) entries need to be reloaded
	- The processor pipelines must be flushed.
- **Indirect Costs:** Cache sharing between multiple processes leading to performance degradation in some cases. Cost varies with different memory access behaviours and for different architectures. It might be referred as *cache interference* later on.

### The Measurement Approach
The paper uses pipe communication to implement frequent context switches between two processes.
They measure:
1. Direct time cost per context switch (**c1**) where processes make no data access.
2. Total time cost per context switch (**c2**) where process allocates and accesses an array of floating point numbers. *Note that c2* includes the indirect cost so the *indirect cost can be estimated to be c2 - c1*.
#### The Direct Cost
Ideally they'd have two processes repeatedly sending a single-byte message to each other via two pipes. 
Each round trip communication between the processes include:
- 2 context switches
- One read system call
- One write system call
The time cost of `10,000` round-trip communications *t1*.

Now consider a single process simulating two processes communication by sending a single-byte message to itself via one pipe. It includes:
- One read system call.
- One write system call.
So it does half the work of the above mentioned two processes barring the context switch cost. They measure the time cost of 10,000 simulated round-trip communication (**t2**).

So direct time cost per context switch will be *c1 = t1/20000 - t2/10000*.
#### Total Cost per Context Switch c2
The control flow is similar to the one above just that after each process becomes runnable, it will access an array of floating-point numbers before it writes a message to the other process and then blocks on the next read operation.
The simulation process does the same amount of array accesses as each of the two communicating processes. 
**s1**: Execution time of 10,000 round-trips between 2 processes.
**s2**: Execution time of 10,000 round-trips between 1 process simulating 2 processes.

We get *c2 = s1/20000  - s2/10000*
Different runs of the tests change the *array size* and *size of the strides of access.*

#### Time Measurement
They use a high resolution times that counts the number of cycles the CPU has gone through since startup. On extremely short times the overhead of the timer itself may cause some error, thus they resort to averaging over a large number of context switches.

### Experimental Results
They use IBM eServer with dual 2.0GHz Intel Pentium Xeno CPUs. Each processor has *512KB L2 cache* and *128B cache line.* 
The average direct context switch cost **c1** was around *3.8 microseconds* for them, while **c2** ranged from several microseconds to more than a 1000 microseconds.

#### Effects of Data Size
With the increment of the array size, the three curves - read, write, or read-modify-write - show similar patterns. Each curve can be divided into three regions:
1. *1KB array size to 200KB array size:* Curve is relatively flat with the time of context switch ranging from 4.2 to 8.7 microseconds. This is cause the entire dataset can fit in the L2 cache.
2. *256KB to 512KB array size:* The dataset of the simulation process fits in the L2 cache but the combined dataset of two actual processes does not fit into the L2 cache, the cost increases to rage of 38.6 - 203.2 microseconds.
	- *384KB array size:* The cost starts to differ significantly for the different access types.
		- Since the test program incurs cache misses on the contiguous memory locations, the execution time is mostly bounded by the memory bandwidth.
		- Writes consume twice the bandwidth, hence RMW and write show twice the cost of read.
3. *512KB and above array size:* 
	- Both the simulation and actual processes cannot fit the data in the L2 cache. The cost of context switch is still high compared to region 1 but it no longer increases monotonously with array size.
	- The cost being high still shows that there is some cache interference.
	- Since the dataset is larger than cache, cache misses are bound to happen even in the absence of context switches.

#### Effects of Access Stride
```c
for (i = 0; i < s; i++) {
	for (j = i; j < array_size; j = j + s) {
		array[j]++
	}
}
```
 Once again for array size between 32KB and 128KB there isn't much of a cost difference as the dataset fits in the cache.
 When access stride (**s**) increases the context switch cost increases because of the change in access pattern. Reason being stride affects cache warm-ups , since for contiguous memory access hardware prefetch works well but for higher strides it does not.