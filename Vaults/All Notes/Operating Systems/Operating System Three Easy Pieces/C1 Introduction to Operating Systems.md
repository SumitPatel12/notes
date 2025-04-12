_____
**Created**: 30-03-2025 03:53 pm
**Status**: Completed
**Tags**: #Operating_System [[Operating Systems]]
**References**: Operating System Three Easy Pieces Book
______

### Virtualizing the CPU
Consider the following program:
```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/time.h>
#include <assert.h>
#include "common.h"
int main(int argc, char *argv[])
{
	if (argc != 2) {
		fprintf(stderr, "usage: cpu <string>\n");
		exit(1);
	}
	char *str = argv[1];
	while (1) {
		Spin(1);
		printf("%s\n", str);
	}
	return 0;
}
```

It simply prints out the provided string until interrupted. `Spin` repeatedly checks the time and terminates after 1 second has elapsed.
If you run this on a single processor machine the output would be obviously infinite prints of that string, but what happens when you try something one the same machine. Consider this: `./cpu A & ./cpu B & ./cpu C & ./cpu D`. The output would be:
```
A
B
C
D
A
B
C
D
...
```

We've got only one processor despite that the instances of the program seem to be running in parallel. It's the *OS* that creates this illusion along with some hardware.
Turning a single CPU (or small set of them) into a seemingly infinite number of CPUs and thus allowing many programs to seemingly run at once is what we call virtualizing the CPU.

to run programs and stop them, and otherwise tell the OS which program to run, there need to be some interfaces/APIs. Plus there are policies in place that drive which program should run if there is some kind of contention.

### Virtualizing the Memory
The model of *physical memory* presented by modern machines is very simple. Memory is but an array of bytes; to *read* memory, one must specify an address to be able to access the data stored there; to *write or update* memory, you need address plus the data that needs to be written there.

Program keeps all of its data structures in memory, and accesses them through various instructions. Each instruction of the program is also in memory; thus memory is accessed on each instruction fetch.
Consider the following program:
```c
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include "common.h"

int main(int argc, char *argv[])
{
	int *p = malloc(sizeof(int)); // a1
	assert(p != NULL);
	printf("(%d) address pointed to by p: %p\n", getpid(), p); // a2
	*p = 0; // a3
	while (1) {
		Spin(1);
		*p = *p + 1;
		printf("(%d) p: %d\n", getpid(), *p); // a4
	}
	return 0;
}
```

It simply allocates some memory, assigns a value, printing, and loops again. The output would be:
``` ts
prompt> ./mem
2134) address pointed to by p: 0x200000
(2134) p: 1
(2134) p: 2
(2134) p: 3
(2134) p: 4
(2134) p: 5
interrupted

// New program execution
prompt> ./mem &; ./mem &
[1] 24113
[2] 24114
(24113) address pointed to by p: 0x200000
(24114) address pointed to by p: 0x200000
(24113) p: 1
(24114) p: 1
(24114) p: 2
(24113) p: 2
(24113) p: 3
(24114) p: 3
(24113) p: 4
(24114) p: 4
```

For multiple instances of the same program you can see that the address assigned was the same. It is as if each running program has its own private memory, instead of sharing the physical memory with other running processes.

And that is what happens as well. The os is *virtualizing the memory*. Each process has access to its own **private virtual address space**, which the OS somehow maps onto the physical memory of the machine. A memory reference within one running program does not affect the address space of other processes; as far as the running program is concerned, it has physical memory all to itself. Ofcourse in reality the address space physically present on the system is shared, and managed by the OS.

### Concurrency
If you've run a program with threads you've likely encountered concurrency issues already. It happens when two threads try to access the same resource leading to contention and sometimes undefined behaviour.

Consider the following program:
```c
volatile int counter = 0;
int loops;

void *worker(void *arg) {
	int i;
	for (i = 0; i < loops; i++) {
		counter++;
	}
	return NULL;
}

int main(int argc, char *argv[]) {
	if (argc != 2) {
		fprintf(stderr, "usage: threads <value>\n");
		exit(1);
	}

	loops = atoi(argv[1]);
	pthread_t p1, p2;
	printf("Initial value: %d\n", counter);

	Pthread_create(&p1, NULL, worker, NULL);
	Pthread_create(&p2, NULL, worker, NULL);

	Pthread_join(p1, NULL);
	Pthread_join(p2, NULL);
	printf("Final Value: %d\n", counter);
	
	return 0;
}
```
Say two threads are going to run this worker function for us, and the value of loops is provided by an argument while running the file. For smaller functions you will observe that the output is 2N (N being the input taken while running the file), but for higher N, the values are likely to be <= 2N, which is odd. That is the issues of concurrency.

The reason we observe this is because of how instructions are executed, which is one at a time. The key part of the program above is where the shared counter is incremented, takes three instructions:
1. Load the value of the counter from memory into a register
2. Increment the value
3. Store it back into memory
The issues we see are because these operations are not atomic i.e. not all 3 run at once.