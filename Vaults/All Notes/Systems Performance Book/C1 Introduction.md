_____
**Created** : 22-03-2025 07:29 pm
**Status**: Completed
**Tags**: #System_Performance
**References**: 
______

*Definition:* Systems performance studies the performance of an entire computer system, including all major software and hardware components. Anything in the data path from storage devices to application software, is included, because it can affect performance.

### Idealish Steps in the Lifecycle of Project Development
List of Activities that are also some ideal steps in the life cycle of the software project from conception to development and to production deployment:
1. Setting performance objectives and performance modelling for a future product.
2. Performance characterisation of prototype software and hardware.
3. Performance analysis of in-development products in a test environment.
4. Non-regression testing for new product versions.
5. Benchmarking product releases.
6. Proof-of-concept testing in the target production environment.
7. Performance tuning in production.
8. Monitoring of running production software.
9. Performance analysis of production issues.
10. Incident reviews for production issues.
11. Performance tool development to enhance production analysis.

Some of the common methods of relating to performance:
1. *Canary Testing:* Testing new software on a single machine or instance with a fraction of the production workload.
2. *Blue-green Deployment:* Move traffic gradually to new pool of instances while leaving the old pool online as a backup.
3. *Capacity Planning:* 
	- During design it pertains to studying the resource footprint of the software to check if it meets the needs.
	- After deployment, monitoring resource usage, user patterns etc.
	- Some bigger products might include having Site Reliability Engineers (SRE) looking after whole sites or cloud infra.

### Perspectives
Performance roles can be viewed from two perspectives (now there can be more, we're gonna look at two):
1. *Workload Analysis:* App Devs look after this one since it includes the performance of the delivered app.
2. *Resource Analysis:* System Admins are the people generally looking at this perspective as they are responsible for the system resources.

### Performance is Challenging
The real task is not finding an issue; it's identifying which ones matter the **most**.
To achieve that we must *quantify* the magnitude of the issue, which in itself is a challenging task. Some issues may not apply to your workload or fixing it might bring such a marginal change the effort might not be worth the gain.

### Latency
Latency is  a measure of the amount of time spent waiting, and is essential performance metric.
As a metric latency can allow maximum speedup to be estimated. Consider a DB query that takes 100ms *(which is the latency)*, during which it spends 80ms on disk reads and 20ms on CPU. Eliminating Disk read can potentially lead to 20ms query time making it 5x faster.

### Observability
#### Counters, Statistics, and Metrics
Apps and kernels typically provide data on their state and activity: operations counts, byte counts, latency measurements, resource utilisation, and error states. Some of which are implemented as integer variables called **counters** that are hard-coded in the software. These counters can then be read at different time for **statistics** such as change over time, average, nth-percentiles, etc.
A **metric** is  a statistic that has been selected to evaluate or monitor a target. Most companies use monitoring agents to record selected stats ant a regular interval and chart them.

#### Profiling
*Profiling* for system performance generally refers to using tools to sample  a subset of measurements to get a rudimentary understanding of the target, CPUs and server instances being one the most common profiling targets.
An effective visualisation of CPU profiles is the *flame graph*. They can help you ind more performance compared to most other methods.

#### Tracing
Tracing is just recording/capturing data to be used for later user or on-the-fly for analytics or other related actions.
##### Static Instrumentation
It describes hard-coded software instrumentation points added to the source code. For *Linux* technology for kernel static instrumentation is called **tracepoints**. For user-space software the same is called **user statically defined tracing (USTD)**. It is used by libraries (like libc) for instrumenting library calls and by many applications for instrumenting service requests.
##### Dynamic Instrumentation
It creates instrumentation points after the software is running by modifying in\memory instructions to insert instrumentation routines. You can think of it as being similar to what a debugger does when introducing a breakpoint in the software, just that debuggers pass the execution flow to an interactive debugger when the breakpoint is hit, whereas dynamic instrumentation runs a routine and then continues the target software.

##### BPF
BPF, which originally stood for Berkeley Packet Filter, is powering the latest dynamic tracing tools for Linux. It originated as a mini in-kernel virtual machine for speeding up the execution of tcpdump expressions. It has since evolved to become an generic in-kernel execution environment, one that provides *safety and fast access to resources.*

### Experimentation
Most of the benchmarking tools would fall under this category. They perform an experiment by applying some synthetic workload to the system for measuring performance.
- *Macro-Benchmark:* Tools that simulate real-world workload such as clients making application request.
- *Micro-Benchmarks:* Tools that test specific component for instance network, load-balancers, CPUs etc.

### Methodologies
These are ways to document the recommended steps for performing various tasks in system performance. 


### Case Studies
#### Slow Disks
**Premise:** You are a system admin and you were given a ticket stating problems with `slow disks` on one of their DB servers.
**Solution:**
- Your first task, lean more about the issue and form a problem statement. The ticket says `slow disks` but it's your responsibility to find out if it is causing a database issue or not since that is not explicitly mentioned on the ticket. So, you revert back by requesting/asking the following:'
	- Is there currently a DB performance issue? How is it measured?
	- How long has this issue been present?
	- Has anything changed with the DB recently?
	- Why were the disks suspected?
- *Reply:* "We've got a log for queries that are slower than 1000ms. These are rare occurrences but as of last week they've been dime a dozen. `AcmeMon` showed that disks were busy."
- You confirm from the response that there is indeed some DB issue, but you further gather that the disk mention is likely a guess and not a certainty. So you decide to check the disks but also look up on the other resources quickly in case the initial hunch was wrong.
- `AcmeMon` is the company's basic monitoring service, providing historical performance graphs based on standard OS metrics.
- You log into `AcmeMon` and see that the disk usage is high around **80%** while other resources show a much lower amount of utilisation. Historical data shows that the disk utilisation has been steadily increasing over the past week while other resources show no such trend.
- You then decide to log into the server and check from there.
	- You check /sys for disk error counters and find out it's `zero`.
	- You run `iostat` with an interval of 1sec and watch utilisation and saturation metrics over time. You observe that the utilisation fluctuates, often hitting 100% and causing levels of saturation and increased disk I/O latency.
- To confirm that this indeed is blocking the DB and is not asynchronous, you use BCC/BPF tracing tool called `offcputime` to capture stack traces whenever the database was descheduled by the kernel, along with the time spent off-CPU. The stack trace show that the DB is often blocking during a file system read, during a query. That is enough evidence for you.
- The next question you ask is `Why?`. The disk performance stats appear to be consistent with high load. 
- So you run `iostat` again but this time to measure IOPS, throughput, average disk I/O latency and the read/write ratio. This presented you with enough reliable information to conclude that the problem was not slow disks but `high disk loads`.
- You report your findings. However, the disks appear to be acting normally for the load. So you ask if the DB load has increased, to which the answer was `no`.
- You now think about other causes. One of which is file system fragmentation - occurs when the file system approaches 100% capacity - which in this case was still only 30%.
- You now check for cache, because significant cache misses can also cause the issues you see. You fin out that the current cache hit ratio is *91%* which is good but you have no idea what it used to be before. You now log onto the other DB servers and notice that their metric is *98%* and also that the `cache size` is significantly larger on other DB server compared to the one the issues are observed on.
- You finally find what was causing the issue after looking at the file system cache size and server memory usage. A **development project** has a prototype app running on the server consuming increasing amount of memory, even though it isn't under production load yet. This memory takes from the available memory for the file system cache leading to more misses and disk reads.