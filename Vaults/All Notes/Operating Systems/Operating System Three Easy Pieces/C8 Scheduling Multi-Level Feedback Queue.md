_____
**Created**: 11-04-2025 08:51 pm
**Status**: Completed
**Tags**: #Operating_System [[Operating Systems]]
**References**: 
______
### Multi-Level Feedback Queue (MLFQ)
It learns from the past to predict the future, and such approaches are common in OSes. 
#### Basic Rules
In the treatment of the book, MLFQ has a number of distinct **queues**, each assigned a **priority level** At any given time, a job that is ready to run is on a single queue. MLFQ uses priorities to decide which job should run at a given time; higher priority runs first. If there are more than one job in a queue i.e. same priority then we will use round-robin scheduling among those jobs.

Rules:
1. If `Priority(A)` > `Priority(B)`, A runs B doesn't.
2. If `Priority(A)` = `Priority(B)`, A and B run in round robin fashion.
3. When a job enters the system, it is placed at the highest priority.
4. Once a job uses up its time allotment at a given level (regardless of how many times it has given up the CPU), its priority is reduced.
5. After some period S, move all jobs in the system to the topmost queue/priority.

Rather than assigning a fixed priority to each job, MLFQ *varies* the priority of jobs based on *observed behaviour.* If a job repeatedly relinquishes the CPUT while waiting for input from the keyboard its priority is high. If, instead it uses CPU intensively for long periods of time its priority is adjusted to be lower.

#### Tuning MLFQ And Other Issues
One big question is how to **Parameterize** such a scheduler. How many queues should there be? How big should the time slice be pr queue? HOw often should priority be boosted in order to avoid starvation and account for changes in behaviour?

As the operating system rarely knows what is best for each and every process of the system, it is often useful to provide interfaces to allow users or administrators to provide some hints to the OS. We often call such hints advice, as the OS need not necessarily pay attention to it, but rather might take the advice into account in order to make a better decision. Such hints are useful in many parts of the OS, including the scheduler (e.g., with nice), memory manager (e.g., madvise), and file system (e.g., informed prefetching and caching