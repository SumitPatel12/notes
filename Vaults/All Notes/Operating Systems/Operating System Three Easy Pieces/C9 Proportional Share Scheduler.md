_____
**Created**: 12-04-2025 03:51 pm
**Status**: Completed
**Tags**: #Operating_System [[Operating Systems]]
**References**: 
______
Instead of optimizing for response time and turnaround times, the scheduler instead tries to guarantee that each job obtains a certain percentage of CPU time. 

### Tickets
#### Tickets Represent Your Share
Each process is assigned **Tickets** and these tickets would then determine what percentage of the resources each process is to get. The working is simple, a lottery is held to decide who runs, the process with more tickets is more likely to win the lottery and thus have more run time and share of the resources.

You can achieve the lottery system by random generation, keep in mind this would then be probabilistic and not deterministic. If speed is the goal such an implementation is a good idea.

#### Ticket Mechanisms
Lottery scheduling provides a concept called **ticket currency**. It allows the user with a set of tickets to allocate tickets among their own jobs in whatever currency they like; the system would then automatically convert said currency to the correct global value.

**Ticket Transfer** allows a process to temporarily hand off its tickets to another process. It's useful in client/server settings where a client process sends a message to a server asking it to do some work on the client's behalf. Client can pass the tickets to the server to boost performance and the server would return it back after its done.

**Ticket Inflation** lets a process temporarily raise or lower the number of tickets it owns. It is only useful when all processes trust one another otherwise you could end up having a scenario where one greedy process hogs all of the resources.

#### Implementation
```c
// counter: used to track if we’ve found the winner yet
int counter = 0;
// winner: use some call to a random number generator to get a value, between 0 and the total # of tickets

int winner = getrandom(0, totaltickets);

// current: use this to walk through the list of jobs
node_t *current = head;
while (current) {
	counter = counter + current->tickets;
	if (counter > winner)
		break; // found the winner
	current = current->next;
}
// ’current’ is the winner: schedule it...
```

To make the process most efficient, it is generally best to keep the list of processes in descending order of tokens possessed.

#### Stride Scheduling
This is the deterministic fair-share scheduler. 
Each job in the system has a stride, which is inverse proportion to the number of tickets it has. Every time a process runs we increment its pass value by the stride to track its global progress. 
The scheduler then uses the stride and pass to determine which process should run next. The basic idea is simple: at any given time, pick the process that has the lowest pass value to run; when you run a process, increment its pass by the stride value.

Example:
- Consider jobs A, B, and C with tickets 100, 50, 250 respectively.
- To get the stride value lets divide a large number - say 1000 - with ticket count of each process. It results in A, B, C having 100, 200, 40 as stride value.
Not writing the dry run you can figure it out yourself 😝.

### Linus Completely Fair Scheduler (CFS)
#### Basic Operation
Instead of a fixed time slice, CFS operates on a counting-based technique known as **virtual runtime (vruntime)**. As each process runs it accumulates `vrtuntime`, the proportion with physical real time is left to the implementer. When making a scheduling decision,CFS will pick the process with the lowest `vruntime`.

How the scheduler decides when to stop a running process and start the next one is crucial. This is because of the trade-off we need to make. 
- If the CFS switches too often it will come at the cost of performance since **context switching** overhead will increase.
- If it switches less often then, near-term fairness will be compromised.

Well this is managed through various control parameters.
- *sched_latency:* 
	- Used to determine how long one process should run before considering a switch. Typical value is `48ms`.
	- CFS divides this value by the number of processes running on the CPU to determine the time slice for a process, ensuring that over this period of time it is completely fair.
- *min_granularity:*
	- Usually set to `6ms`.
	- CFS will never set the time slice lower than this value, ensuring that too much time is not spent in context switching and other scheduling overhead.

#### Weighting (Niceness)
CFS enables control over process priority. This is done through tickets, but through a mechanism known as **nice** level of the process. It can be set between `-20` to `+19` with a default of `0`, well hold you horses, you wouldn't have expected this, *negatives* mean higher prio and *positives* mean lower prio.
Signed integers strike back ⚔.
```c
static const int prio_to_weight[40] = {
	/* -20 */ 88761, 71755, 56483, 46273, 36291,
	/* -15 */ 29154, 23254, 18705, 14949, 11916,
	/* -10 */ 9548, 7620, 6100, 4904, 3906,
	/* -5 */ 3121, 2501, 1991, 1586, 1277,
	/* 0 */ 1024, 820, 655, 526, 423,
	/* 5 */ 335, 272, 215, 172, 137,
	/* 10 */ 110, 87, 70, 56, 45,
	/* 15 */ 36, 29, 23, 18, 15,
};
```

Now the weights can be factored in when calculating which process to run next.
$$
time\_slice_{k} = \frac{x}{\sum_{i=0}^{n-1}{weight_{i}}}.schedule\_latency
$$
#### Using Red-Black Trees
- CFS keeps processes in a Red-Black Tree, it has the benefit of operations having logarithmic times and not linear, with the caveat that it has to do a little extra work to maintain low depths.
- CFS does not keep all processes in this structure, only running or runnable ones, if a process sleeps or is completed it is removed and tracked elsewhere if needed.
- The processes are ordered in the tree by `vruntime`, and most operations such as deletion and insertion are logarithmic in time, when n is in thousands this makes a significant difference.

#### Dealing with I/O and Sleeping Processes
One problem with picking the lowest `vruntime` to run next arises with jobs that have gone to sleep for a long period of time. Imagine two processes, A and B, one of which (A) runs continuously, and the other (B) which has gone to sleep for a long period of time (say, 10 seconds). When B wakes up, its `vruntime` will be 10 seconds behind A’s, and thus (if we’re not careful), B will now monopolise the CPU for the next 10 seconds while it catches up, effectively starving A.

CFS handles this case by altering the `vruntime` of a job when it wakes up. Specifically, CFS sets the `vruntime` of that job to the minimum value found in the tree (remember, the tree only contains running jobs). In this way, CFS avoids starvation, but not without a cost: jobs that sleep for short periods of time frequently do not ever get their fair share of the CPU