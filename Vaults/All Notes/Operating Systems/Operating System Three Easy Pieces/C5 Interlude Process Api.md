_____
**Created**: 06-04-2025 02:23 pm
**Status**: Completed
**Tags**: #Operating_System [[Operating Systems]]
**References**: 
______

UNIX system process creation is among the more intriguing ways of creating a new process. It does so with a pair of system calls:
1. `fork()`
2. `exec()`
There is a third routine called `wait()`, used by a process wishing to wait for a process it has created to complete.

### fork() System Call
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main (int argc, char *argv[]) {
	printf("hello world (pid: %d)\n", (int) getpid());

	int rc = fork();
	if (rc < 0) {
		// fork failed
		fprintf(stderr, "fork failed\n");
		exit(1);
	} else if (rc == 0) {
		// child process
		printf("Hello, I'm the child process (pid: %d)\n", (int) getpid());
	} else {
		// parent goes down this path (main)
		printf("Well, hello there, I'm sure you didn't forget about the parent process, (rc: %d) (pid: %d)\n", rc, getpid());
	}
}
```

Well the code is simple enough:
- We first print `hello world` along with the processId.
- Then we fork the current process. What it does is that it creates an **exact copy of the calling process** (as far as the end user is concerned of course).
- To the OS this would look like there are two copies of the program p1 running, and both are about to return from the `fork()` system call. 
- The newly created process, otherwise known as the *child process*, doesn't start running at main; rather it just comes into life as if it had called fork itself.
- Well now comes in the key difference, for the parent the fork returns the **PID** of the newly created child process, for the child process it returns `0`. This enables us to handle the different scenarios of what to call for the child and what for the parent.

### exec() System Call
The `fork()` system call is strange; its partner in crime, exec(), is not so normal either. What it does: given the name of an executable (e.g., wc), and some arguments (e.g., p3.c), it loads code (and static data) from that executable and overwrites its current code segment (and current static data) with it; the heap and stack and other parts of the memory space of the program are re-initialized. Then the OS simply runs that program, passing in any arguments as the argv of that process. Thus, it does not create a new process; rather, it transforms the currently running program (formerly p3) into a different running program (wc). After the exec() in the child, it is almost as if p3.c never ran; a successful call to exec() never returns.