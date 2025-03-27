_____
**Created**: 23-03-2025 05:10 pm
**Status**: In Progress
**Tags**: #Database #SQLite [[Database]] [[SQLite]]
**References**: [SQLite Architecture](https://www.sqlite.org/arch.html)
______

SQLite compiles the SQL text into the [bytecode](https://www.sqlite.org/opcode.html), then running that [bytecode](https://www.sqlite.org/opcode.html) using a virtual machine. The [sqlite3_prepare_v2()](https://www.sqlite.org/c3ref/prepare.html) and related interfaces act as a compiler for converting to SQL text into bytecode. The [sqlite3_stmt](https://www.sqlite.org/c3ref/stmt.html) object is a container for a single bytecode program that implements a single SQL statement. The [sqlite3_step()](https://www.sqlite.org/c3ref/step.html) interface passes the a bytecode program into the virtual machine, and runs the program until it either completes, or forms a row of result to be returned, or hits a fatal error, or is interrupted.

![[SQLite Components.excalidraw|500]]
### Tokenizer
When a string containing SQL statements is to be evaluated it is first sent to the `tokenizer`, which tokenizes it and passes those tokens one by one to the parser. tokenizer is hand coded in the file [tokenize.c](https://github.com/sqlite/sqlite/blob/master/src/tokenize.c).

### Parser
It assigns meaning to the tokens based on their context. SQLite parser is generated using the [Lemon Parser Generator](https://www.sqlite.org/lemon.html). Lemon generates a parser which is reentrant and thread-safe, and also defines the concept of a non-terminal destructor so that it *does not leak memory when syntax error are encountered.* Grammar file: [parse.y](https://sqlite.org/src/file/src/parse.y)

### Code Generator
After parser generates the parse tree, the code generator runs to analyse the parse tree and generate bytecode that performs the work of the SQL statement. [prepared_statement](https://www.sqlite.org/c3ref/stmt.html) object is a container for this bytecode. The code generator has multiple files that handle the different statements found in SQL. Link to the source folder for these files.: https://github.com/sqlite/sqlite/tree/master/src

### Bytecode Engine
As established earlier the [bytecode](https://www.sqlite.org/opcode.html) is run by a virtual machine. The entirety of which is present within the [vdbe.c](https://sqlite.org/src/file/src/vdbe.c) file. 
- [vdbe.h](https://sqlite.org/src/file/src/vdbe.h) header file defines an interface between the virtual machine and the rest of the SQLite library.
- [vdbeInt.h](https://sqlite.org/src/file/src/vdbeInt.h) which defines structures and interfaces that are private to the virtual machine itself.
- Individual values (strings, integer, floating point numbers, and BLOBs) are stored in an internal object named "Mem" which is implemented by [vdbemem.c](https://sqlite.org/src/file/src/vdbemem.c).
There are more files within the `vdbe*.c` family which are helpers to the virtual machine.

### B-Tree
- The DB itself is maintained on disk using a B-tree implementation, source file: [btree.c](https://sqlite.org/src/file/src/btree.c).
- Separate B-trees are used for each table and each index in the DB.
- All B-trees are stored on the same disk file.
- [[Database File Format|SQLite file Format]]
- The interface to the B-tree subsystem and the rest of the SQLite library is defined by the header file [btree.h](https://sqlite.org/src/file/src/btree.h).

### Page Cache
- B-tree module requests information from the disk in fixed-size pages. The default `page_size` is 4096 bytes but can be any power of 2 between 512 and 65536 bytes. 
- It is responsible for reading, writing and caching pages.
- Provides the rollback and atomic commit abstraction  and takes care of locking the DB file.
- B-tree driver requests pages from the page cache and notifies it in case it wanted to make updates or commit or rollback some change.
- Page cache handles the details of making sure they are handled quickly, safely, and correctly.

The primary page cache implementation is in the [pager.c](https://sqlite.org/src/file/src/pager.c) file. [WAL mode](https://www.sqlite.org/wal.html) logic is in the separate [wal.c](https://sqlite.org/src/file/src/wal.c). In-memory caching is implemented by the [pcache.c](https://sqlite.org/src/file/src/pcache.c) and [pcache1.c](https://sqlite.org/src/file/src/pcache1.c) files. The interface between page cache subsystem and the rest of SQLite is defined by the header file [pager.h](https://sqlite.org/src/file/src/pager.h).

### OS Interface
In order to provide portability across operating systems, SQLite uses an abstract object called the [VFS](https://www.sqlite.org/vfs.html). Each VFS provides methods for opening, reading, writing, and closing files on disk, and for other OS-specific tasks such as finding the current time, or obtaining randomness to initialize the built-in pseudo-random number generator. SQLite currently provides VFSes for unix (in the [os_unix.c](https://sqlite.org/src/file/src/os_unix.c) file) and Windows (in the [os_win.c](https://sqlite.org/src/file/src/os_win.c) file).

### Utilities
Memory allocation, caseless string comparison routines, portable text-to-number conversion routines, and other utilities are located in [util.c](https://sqlite.org/src/file/src/util.c). Symbol tables used by the parser are maintained by hash tables found in [hash.c](https://sqlite.org/src/file/src/hash.c). The [utf.c](https://sqlite.org/src/file/src/utf.c) source file contains Unicode conversion subroutines. SQLite has its own private implementation of [printf()](https://www.sqlite.org/printf.html) (with some extensions) in [printf.c](https://sqlite.org/src/file/src/printf.c) and its own pseudo-random number generator (PRNG) in [random.c](https://sqlite.org/src/file/src/random.c).