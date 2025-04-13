_____
**Created**: 12-04-2025 04:54 pm
**Status**: In Progress
**Tags**: #Database #Concurrency [[Database]] [[Concurrency]]
**References**: 
______
### Questions And Things to Look Into
- I'm assuming that `READ UNCOMMITTED` isolation level is not something relevant here. It's commonly used to prevent locking and slowing down thing. 🤷‍♂

### Transactions
*Concurrency* control is the activity of coordinating the actions of processes that operate in parallel, access shared data, and therefore potentially interfere with each other.
*Recovery* is the activity of ensuring that software and hardware failures do not corrupt persistent data.

The main component of the model discussed of in this book is **transaction**. Informally, a transaction is an execution of a program the accesses a shared database. The goal of concurrency control and recovery is to ensure that transactions execute *atomically*, meaning that
1. Each transaction accesses shared data without interfering with other transactions, and
2. If a transaction terminates normally, then all of its effects are made permanent; otherwise it has no effect at all.
#### Database Systems
- A *database* consists of a stet of named *data items*. Each data item has a *value*. The values of the data items at any one time comprise the *state* of the database.
- A data item could be a word of main memory, a page of a disk, a record of a file, or field of a record. The size of the data contained in a data item is called the *granularity* of the data item.
- A *Database System (DBS)'* is a a collection of hardware and software modules that support commands to access the database, called *database operations (or operations)*. The most important operations to consider for us are **Read and Write** operations.
- The DBS executes each operation atomically, meaning it behaves as if it executes *sequentially.* Often it executes *concurrently* but to the user it seems as if they executed sequentially.
- The DBS supports *transaction operations:* Start, Commit, and Abort.
- A program must issue each of its database operations on behalf of a transaction.
- A transaction may be a *concurrent* execution of two or more programs. The last operation however must be a Commit or Abort, any operation issued for the same transaction after it has committed or aborted should be refused to be serviced by the DBS.

#### Transaction Syntax
From the user's viewpoint, a **transaction** is the execution of one or more programs that include database and transaction operations.
Following is an example of the code type used to represent transactions throughout the book. c is just for syntax highlighting in obsidian which I use for note taking.
```c
Procedure Transfer begin
	Start;
	input(fromaccount, toaccount, amount);
	/* This procedure transfers “amount” from “fromaccount” into “toaccount.”*/
	
	temp : = Read(Accounts[fromaccount]);
	
	if temp < amount then begin
		output( “insufficient funds”);
		Abort
	end
	else begin
		Write(Accounts[fromaccount], temp - amount);
		temp : = Read(Accounts[toaccount]);
		WritefAccounts[toaccount], temp + amount);
		Commit;
		output( “transfer completed”);
		end;
	return
end
```

#### Commit and Abort
Transaction is said to be:
- *Committed:* DBS has executed commit operation.
- *Aborted:* DBS has executed abort operation.
- *Active:* DBS has executed the Start operation but not the commit or abort.
- *Uncommitted:* If the transaction is active or aborted.
When a transaction aborts, the DBS will wipe out all of its effects.

The **Commit** operations invocation signifies that a transaction terminated “normally” and that its effects should be permanent. Executing a transaction’s Commit constitutes a guarantee by the DBS that it will not abort the transaction and that the transaction’s effects will survive subsequent failures of the system.

The DBS should guarantee the permanence of Commit under the weakest possible assumptions about the correct operation of hardware, systems software, and application software. That is, it should be able to handle as wide a variety of errors as possible. At least, it should ensure that data written by committed transactions is not lost as a consequence of a computer or operating system failure that corrupts main memory but leaves disk storage unaffected.

#### Messages
- Each transaction is self-contained; no direct communication with other transactions.
- They can communicate indirectly via the database. Storing and retrieving data counts as such.
- DBS mediates each operation that can affect other transactions
- In the model discussed in this book, communication by sending messages between transactions is allowed as long as those messages are stored in the database.
- Two or more processes executing on behalf of the same transaction are free to exchange messages.

### Recoverability
Effects a transaction **T** can have:
1. Anything it wrote to the Database.
2. Other transactions reading the written value(s).

When a transaction aborts, DBS should handle both the above mentioned effects. Say if T1 wrote something that T2 reads then aborting T1 should also abort T2.

We say a transaction **T$_{j}$ reads from T$_{i}$** in an execution if:
1. T$_{j}$ reads x after T$_{i}$  has written into it;
2. T$_{i}$ does not abort before T$_{j}$ reads x; and
3. Every transaction (if any) that writes x between the time T$_{i}$  writes it and T$_{j}$ reads it, aborts before T$_{j}$ reads it.

An execution is **recoverable** if for every transaction T that commits, T's commit follows the commit of every transaction from which T read. Recoverability is required to ensure that aborting a transaction does not change the semantics of committed transactions' operations.

#### Terminal I/O
An execution is recoverable if the DBS is always able to reverse the effects of an aborted transactions on other transactions. The definition of recoverable relies on the assumption that all such effects are through Reads and Writes. Without this assumption, the definition of recoverable does not correspond to its intuition.
Now there are I/O interactions the user can do that interact with a transaction, and what they decide to do with the information and the DBS none the wiser will not abort transactions related as such.
One way to prevent this is for the DB to defer transaction T's output until it has been committed, then the user will only see committed output. This while acceptable a lot of times is not something always viable, but we got to live with that you know, if everything went my way I'd be the richest healthiest and probably the laziest person in the world 🤑.

#### Avoiding Cascading Aborts
Recoverability does not mean no cascading errors, it is in fact essential to have cascading aborts to ensure recoverability.
The prospect of cascading aborts is understandably a headache for the DBS designer since it involves a lot of bookkeeping and the possibility of cascading soooo many transactions - infinitely many if you think about it. Now that is something you want to walk right past and not look back at, DBS wants the same so they try very hard to avoid cascading aborts.

To do this the DBS makes sure that a transaction only reads committed data (there is a concept of isolation levels that let you get around that).

To achieve cascadelessness, the DBS must delay each Read(x) until all transactions that have previously issued a Write(x, val) have either aborted or committed. In doing so, recoverability is also achieved: a transaction must execute its Commit after having executed all its Reads and therefore after all the Commits of transactions from which it read.

#### Strict Executions
One of the problems that can occur is when two transactions write to the same variable and abort.
```c
Write1(x, 2); Write2(x, 3); Abort1; Abort2;
```
The before image of Write,(x, 3) is 2, the value written by T1. However, the value of x after Write,(x, 3) is undone should be 1, the initial value of x (since both updates of x have been aborted). In this case the problem is that the before image was written by an aborted transaction.

To avoid this the execution of `Write(x, val)` should be delayed until all transactions that have previously written `x` are either committed or aborted. Similar to what we had for avoiding cascading aborts that `Read(x)` be delayed until all transaction that had previously written `x` are either committed or aborted.

Executions that satisfy both the above mentioned conditions are called **strict**. Strict execution avoid cascading aborts and are serializable.

### Serializability
#### Concurrency Control Problem