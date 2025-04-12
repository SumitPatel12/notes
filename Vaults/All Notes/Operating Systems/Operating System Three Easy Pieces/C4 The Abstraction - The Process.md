_____
**Created**: 05-04-2025 06:05 pm
**Status**: Completed
**Tags**: #Operating_System #Virtualization [[Operating Systems]]
**References**: 
______
The OS runs one process, then stops it and then runs another, and so forth, thus promoting the illusion that many virtual CPUs exist when in fact only one or - for modern hardware - a few are available. This is one of the very basic techniques and is known as `time sharing`. 

### The Abstraction: A Process
The abstraction provided by the OS of a running program is something we will call a **process**.
To understand what constitutes a process we have to understand its machine state. One obvious component of the machine is *memory.* Another part are *registers*; many instructions explicitly read or update registers and thus clearly they are important to the execution of the process. 

There are some particularly special registers that form part of this machine state. E.g., the *program counter (PC) sometimes called the instruction pointer or (IP)*, tells us which instruction of the program will execute next; similarly a *stack pointer* and associated *frame pointer* are used to manage the stack for function parameters, local variables, and return addresses.

Finally, programs often access persistent storage devices too, thus information regarding the open files and other such things might be included in the machine state as well.

### Process API
- **Create:** A method to create a new process.
- **Destroy:** Something to kill/destroy an existing process forcefully.
- **Wait:** Wait for a process to stop running.
- **Miscellaneous Control:** Some method to suspend and resume a process in execution.
- **Status:** Get info on a process like how long it has run for, what state it is in, etc.

### Process Creation: A Little More Detail
The first thing that the OS must do to run a program is to **load** its code and any static data associated with the program into memory. Programs initially resided on a persistent storage medium in some kind of **executable format.** The OS thus reads those bytes from the persistent storage and places them in memory.
![[Loading A Program To Process.excalidraw|700]]

Once the code and static data are loaded into memory, there are a few other things the OS needs to do before running the process. A certain amount of memory must be allocated for the program's **run-time stack (or just stack).** Stack holds the local variables, function, return addresses, and other such things.

The OS further allocates some memory for the **heap**. Heap generally used for dynamic allocation for instance when a running program generates some data whose size you could not tell at compile time, it goes to the heap. *Rust vectors and Box* are heap allocated.

The OS is still not done, there are some other initialization tasks, particularly related to the I/O. For example in UNIX systems, each process by default has three open *file descriptors*, for standard input, output, and error; these descriptors let programs easily read input from the terminal and print output to the screen.

After all this the OS has finally set the stage for the program execution. The only thing left to do is to start the execution which for different programs is different, like for `C` and `Rust` its the main function. After that OS transfers control of the CPU to the newly created process and so the process begins execution.

### Process States
- **Running:** Process is running on a processor i.e. executing instructions.
- **Ready:** The process is ready to run, but the OS has chosen not to run it at this given time.
- **Blocked:** The process has performed some kind of operation that makes it not ready to run until some other event takes place.
![[Process State Transitions.excalidraw|500]]

### Data Structures
Well the process is likely to keep some kind of *process list* for all processes with additional metadata on which ones are running, which are blocked, which are ready to execute. 

Code block below shows some information the OS tracks about a process. The **register context** will hold, for a stopped process, the contents of its registers. When a process is stopped, its registers will be saved to this memory location; by restoring these registers (i.e., placing their values back into the actual physical registers), the OS can resume running the process.
```c
// The registers xv6 will save and restore to stop and subsequently restart a process
struct context {
	int eip;
	int esp;
	int ebx;
	int ecx;
	int edx;
	int edi;
	int ebp;
};

enum proc_state {
	UNUSED,
	EMBROY,
	SLEEPING,
	RUNNABLE,
	RUNNING,
	ZOMBIE
};

// The nformation xv6 tracks about each porcess including its register context and state
struct proc {
	char *mem;                    // Start of the process memory
	unit sz;                      // Size of the process memory
	char *kstaack;                // Bottom of the kernel stack for this process
	enum proc_state state         // Process State
	int pid;                      // Process Id
	struct proc *parent           // Parent Process
	void *chan;                   // If not zero, sleeping on chan
	int killed;                   // If not zero, has been killed
	struct file *ofile[NOFILE];   // Open files
	struct inode *cwd;            // Current working directory
	struct context context;       // Switch here to run process
	struct trapframe *td;         // Trap frame for the current interrupt
}
```

### Key Terms
- The **process** is the major OS abstraction of a running program. At any point in time, the process can be  described by its state: The contents of memory in its **address space**, the contents of CPU registers (including the **program counter** and **stack pointer**, among others), and information about I/O.
- The **process API** consists of calls programs can make related to processes. Typically, this includes creation, destruction, and other useful calls.
- Processes exist in one of many different **process states**, including running, ready to run, and blocked. Different events transition a process from one of these states to the other.
- A **process list** contains information about all processes in the system. Each entry is found in what is sometimes called a **process control block (PCB)**, which is really just a structure that contains information about a specific process.