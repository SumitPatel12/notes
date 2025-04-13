_____
**Created**: 11-04-2025 08:56 pm
**Status**: Completed
**Tags**: #SQLite #Database #Write_Ahead_Logs [[SQLite]] [[Database]]
**References**: [SQLite WAL Doc](https://sqlite.org/wal.html)
______
### Overview
#### Advantage
- Significantly faster in most scenarios
- Provides more concurrency since readers do not block writers and writers do not block readers. Reading and writing can proceed concurrently.
- Disk I/O operations tends to be more sequential using WAL.
- WAL uses may fewer `fsync()` operations and is thus less vulnerable to problems on systems where the said system call is broken.
#### Disadvantages
- All processes using a DB must be on the same host machine; WAL does not work over a network filesystem. This is because WAL requires all processes to share a a small amount of memory and processes on separate host machines cannot share memory 🤷‍♂.
- Transactions that involve changes against multiple [[ATTACHed]] dbs are atomic for each individual db, but are not atomic across al dbs as a set.
- It is not possible to change the `page_size` after entering WAL mode.
- WAL might be slightly slower than traditional rollback-journal approach in applications that do mostly reads and seldom write. 1 or 2% slower.
- There is an additional quasi-persistent `-wal` file and `-shm` shared memory file associated with each database, which can make SQLite less appealing for use as an [application file-format](https://sqlite.org/appfileformat.html).
- There is the extra operation of [checkpointing](https://sqlite.org/wal.html#ckpt) which, though automatic by default, is still something that the application developers need to be mindful of.

### How WAL Works
he original content is preserved in the database file and the changes are appended into a separate WAL file. A `commit` occurs when a special record indicating a commit is appended to the WAL. Thus a commit can happen without ever writing to the original database, which allows readers to continue operating from the original unaltered database while changes are simultaneously being committed into the WAL. Multiple transactions can be appended to the end of a single WAL file.

#### Checkpointing
Checkpointing is moving the WAL file transactions back into the database 
By default SQLite does a checkpoint automatically when the WAL file reaches a threshold size of `1000 pages`; it is customisable by providing a compile time value.

Checkpointing runs with writes when the WAL file is 1000 pages or more , or when the last connection with the DB closes. Also, the WAL file is automatically deleted when the last open connection to the database file closes.

#### Concurrency
At the beginning of a read operation in WAL mode, we remember the location of the **last valid commit** record in the WAL file called `end mark`. Multiple readers can have different `end marks` but each individual reader the `end mark` is unchanged for the duration of the transaction.

When a reader needs a page, it first checks WAL for the page entry prior to the `end mark`, if it is present it's taken otherwise the original database file is queried for the same. Readers can exist in separate processes, so to avoid forcing every reader to scan the WAL for pages, a data structure called the **wal-index** is maintained in *shared memory*. It improves the performance but with the caveat that the readers must exist on the same machine since the **wal-index** is in shared memroy.

There can be only one writer at a time.

A checkpoint operations takes content from the WAL file and transfers it back to the original database file. *Checkpointing can run concurrently with readers,* but it must stop when it reaches a page in the WAL that is past the `end mark` of any current reader. The reason is simple we risk changing the DB content which might be actively used by that reader. The *checkpoint* remembers - in the wal-index - how far it got and will resume transferring form the same point during the next invocation.

Write operation now check for how much progress the *checkpointer* has made, and *if the WAL has been completely synced to the original database file and no readers are making use of the WAL*, then the writer will rewind the WAL back to the beginning and start putting new transactions at the beginning of the WAL.

#### Performance Considerations
Write transactions are fast since they only involve writing the content once and because the writes are all sequential. Further, syncing the content to the disk is not required, as long as the application is willing to sacrifice durability following a power loss or hard reboot.

Read performance deteriorates as WAL file grows in size, the **wal-index** is indeed useful but performance is still impacted as the wal file grows in size.

Checkpointing requires a sync operation in order to avoid possibility of database corruption following a power loss or hard reboot. The WAL must be synced to a persistent storage prior to moving content from the WAL into the database and the database file must be synced prior to resetting the WAL. It also required more `seeking`, this makes checkpoints slower than the normal write transactions.

The *default strategy* is to let the WAL grow to `1000 pages` then run a checkpoint operation for each subsequent **COMMIT** until the WAL is reset to smaller than 1000 pages. By default it runs automatically when the WAL goes over the specified limit, because of this one would observe that most of the **COMMITs** are fast but there are few that are slow because they run the *checkpoint* operation. You can configure SQLite to not run checkpoint during writes and have it run in separate thread or process.

There is a tradeoff between the **average read** and **average write** performance. This is because the read performance requires the WAL to be as small as possible, to do this you would want to run checkpoints frequently, maybe as often as every commit. On the other hand writes wants to amortize the cost of each checkpoint over as many writes as possible, meaning one wants to run checkpoints infrequently and let the WAL grow as large as possible; directly opposing what we want for reads. 

### Implementation Of Shared-Memory For The WAL-Index
The only way they found to guarantee that all the process accessing the same database file use the same shared memory was to create the shared memory by `mmpaping` a file in the same directory as the database itself.

You might think using an ordinary disk file to provide shared memory would be disadvantageous, cause it may cause unnecessary disk I/O, but it's not a concern as the file rarely exceeds 32KiB ins size and is never synced.

### Queries Return SQLITE_BUSY In WAL Mode
I lied when I said reads and writes would carry on concurrently 😝. Following scenarios might give you SQLITE_BUSY:
- If another database connection has the database mode open in [exclusive locking mode](https://sqlite.org/pragma.html#pragma_locking_mode) then all queries against the database will return [SQLITE_BUSY](https://sqlite.org/rescode.html#busy). Both Chrome and Firefox open their database files in exclusive locking mode, so attempts to read Chrome or Firefox databases while the applications are running will run into this problem, for example.
- When the last connection to a particular database is closing, that connection will acquire an exclusive lock for a short time while it cleans up the WAL and shared-memory files. If a separate attempt is made to open and query the database while the first connection is still in the middle of its cleanup process, the second connection might get an [SQLITE_BUSY](https://sqlite.org/rescode.html#busy) error.
- If the last connection to a database crashed, then the first new connection to open the database will start a recovery process. An exclusive lock is held during recovery. So if a third database connection tries to jump in and query while the second connection is running recovery, the third connection will get an [SQLITE_BUSY](https://sqlite.org/rescode.html#busy) error.
