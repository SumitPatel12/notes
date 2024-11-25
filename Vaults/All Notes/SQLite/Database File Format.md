_____
**Created**: 23-11-2024 09:40 am
**Status**: In Progress
**Tags**: #SQLite [[SQLite]]
**References**: [SQLite Database File Format](https://www.sqlite.org/fileformat.html)
______

### The Database File
The complete state of an SQLite database is usually contained in a single file on disk called the `main datbase file`. During transactions SQLite stores additional information in a second file called the `rollback journal`, in a write-ahead log file depending on the operational mode.

#### Pages
The main DB file consists of one one or more pages, each of which has a size of power of 2 between 512 and 65536 both inclusive. All pages within the same DB file are the same size. *The page size is determined by the 2-byte integer located at an offset of 16 bytes form the beginning of the database file.* Pages are numbered beginning from 1 and go upto 2$^{32}$  - 2. 

Every page in in the main DB has a single use which is one of the following:
- *A b-tree page*
	- A table b-tree interior page 
	- A table b-tree exterior page 
	- A index b-tree interior page 
	- A index b-tree leaf page 
- *A freelist page*
	- A freelist trunk page
	- A freelist leaf page
- *A payload overflow page*
- *A pointer map page*
- *The lock-byte page*

Al reads from and writes to the main DB file begin at a page boundary and all writes are an integer number of pages in size. 

#### Database Header
The first 100 bytes of the database file comprise the database file header. All of the multibyte fields in the DB file header are stored with the most significant byte first - big-endian.


| Offset | Size (bytes) | Description                                                                                                     |
| ------ | ------------ | --------------------------------------------------------------------------------------------------------------- |
| 0      | 16           | The header string: `SQLite format 3\000`                                                                        |
| 16     | 2            | The DB page size. Power of 2 between 512 and 32768 inclusive, or the value 1 representing 65536                 |
| 18     | 1            | File format write version. 1 for legacy; 2 for [[WAL]]                                                          |
| 19     | 1            | File format read version. 1 for legacy; 2 for [[WAL]]                                                           |
| 20     | 1            | Bytes of unused `reserved` space at the end of each page. Usually 0.                                            |
| 21     | 1            | Maximum embedded payload fraction. Must be 64.                                                                  |
| 22     | 1            | Minimum embedded payload fraction. Must be 32.                                                                  |
| 23     | 1            | Leaf payload fraction. Must be 32.                                                                              |
| 24     | 4            | File change counter. (Big-endian)                                                                               |
| 28     | 4            | Size of the database file in pages. The `in-header DB size`.                                                    |
| 32     | 4            | Page number of the first freelist trunk page.                                                                   |
| 36     | 4            | Total number of freelist pages.                                                                                 |
| 40     | 4            | The schema cookie.                                                                                              |
| 44     | 4            | The schema format number. Supported values are 1, 2, 3, and 4.                                                  |
| 48     | 4            | Default page cache size.                                                                                        |
| 52     | 4            | The page number of the largest root b-tree page when in auto-vacuum or incremental-vacuum modes, or 0 otheriwse |
| 56     | 4            | The DB text encoding. 1 meaning UTF-8, 2 meaning UTF-16le and 3 meaning UTF-16be.                               |
| 60     | 4            | The `user version` as read and set by the [[User Version Program]].                                             |
| 64     | 4            | True(non-zero) for incremental-vacuum mode. False(zero) otherwise.                                              |
| 68     | 4            | The `Application ID` set by the [[PRAGMA Application ID]].                                                      |
| 72     | 20           | Reserved for expansion. Must be 0.                                                                              |
| 92     | 4            | The [[Version-Valid-For number]].                                                                               |
| 96     | 4            | The SQLite Version Number.                                                                                      |
#### Freelist
Unused pages are stored in the **freelist** and are reused when additional pages are required.
The freelist is organised as a linked list of trunk pages with each trunk page containing page numbers for zero or more freelist leaf pages.

*Freelist trunk page* consists of an array of 4-byte big-endian integers. The size of the array is as many integers as will fit in the usable space of a page.
- The first integer on a freelist trunk page is the page number of the next freelist trunk page in the list, or zero if it is the last freelist trunk page.
- The second integer is the number of leaf page pointers to follow.
- The freelist leaf pages contain no information.

![[Freelist Structure.excalidraw|600]]

#### B-Tree Pages
SQLite uses two variations of **b-trees**
1. `Table B-Trees` use a 64-bit singed integer key and store all of the data in the leaves.
2. `Index B-Trees` use arbitrary keys and store no data at all.

Structure of the b-tree:
- K is the number of keys on an interior `b-tree` page.
	- K is at least 2 and in practical cases much more than 2, the upper bound being the number of keys that can fit into the page.
	- The exception being when the page 1 is an interior `b-tree` page. Page 1 has 100 fewer bytes of storage space available due to the database header, and so sometimes it can end up holding just a single key.
	- If the key size is bigger than 1/4th of the page size then it is stored in overflow pages. Guaranteeing that each page will hold **at least 4 keys**.
- A *leaf page* contains keys and in case of the `table b-tree` the associated data.
- The interior page contains K+1 pointers to cild `b-tree` pages. The pointers are just 32-bit unsigned integer page number of the child page.

Within an interior `b-tree` page, the key and pointer to its immediate left are combined into a structure called a `cell`. The right-most pointer is held separately.
A *leaf node* has no pointers but it does use the cell structure to hold keys for the `index b-tree` or keys and content for the `table b-tree`. Data is not contained in the cell.

The `b-tree` corresponding to the `sqlite_schema` table is always a `table b-tree` and always has a root page of 1. It contains the root page number for every other table and index in the DB file.
Each entry in the `table b-tree` consists of a *64-bit* signed integer key and up to `2147483647` bytes of arbitrary data. 
Each entry in the `index b-tree` consists of a an arbitrary key of up to `2147483647` bytes in length and not data.
**Payload** of a cell is the arbitrary length section of the cell.
- For `index b-tree` it is the key itself.
- For `table b-tree`
	- *Interior page* does not have any arbitrary data so it does not have a payload.
	- *Leaf page* contains actual data which is arbitrary and that is the payload.
- When the size of a **payload** crosses a certain threshold, only the first few bytes are stored in the `b-tree` the remaining is stored as a linked list of content overflow pages.

##### Regions of the B-Tree
Regions:
1. 100 byte header *only in the page 1 of a `table b-tree`.*
2. 8 or 12 byte b-tree *page header.*
3. The cell pointer array
4. Unallocated space
5. The cell content area
6. The reserved region

| Offset | Size | Description                                                                                                                                                                                                                        |
| ------ | ---- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0      | 1    | Indicates page type:<br>- 2 (0x02) indicates `interior index` b-tree page<br>- 5 (0x05) indicates `interior table` b-tree page<br>- 10 (0x0a) indicates `leaf index` b-tree page<br>- 13 (0x0d) indicates `leaf table` b-tree page |
| 1      | 2    | Start of the first freeblock on the page.                                                                                                                                                                                          |
| 3      | 2    | Number of cells on the page.                                                                                                                                                                                                       |
| 5      | 2    | Start of the cell content area. 0 value is interpreted as 65536.                                                                                                                                                                   |
| 7      | 1    | Number of fragmented free bytes within the cell content area.                                                                                                                                                                      |
| 8      | 4    | Right most pointer. Appears only in the header of the *interior b-tree* pages and is omitted for all other pages.                                                                                                                  |
![[B-Tree Page Structure.excalidraw|1500]]

#### Pointer Map or Ptrmap Pages
These are extra pages inserted into the DB to make the operations of [[Auto Vacuum|auto vacuum]] and [[Incremental Vacuum|incremental vacuum]] modes more efficient. A `ptrmap page` contains linkage info from child to parent. Ptrmap pages must exist in any DB file which has a non-zero largest *root b-tree* value at offset 52 in the header file. In a DB with `ptrmap pages`, the first of such page is `page 2`. It consists of an array of 5-byte entries. The page provides backlink information for the next N pages (N being the size of the 5 byte integer array).

Each 5-byte ptrmap entry consists of one byte of `page type` and a big-endian `page number`. Following 5 page types are recongnized:
1. *B-Tree root page*. The page number should be 0.
2. *Freelist page.* Page number should be 0.
3. First page of *cell payload overflow chain*. The page number of the b-tree page that contains the cell whose content has overflowed.
4. A page in the *overflow chain* other than the first page. The page number is the prior page of the overflow chain.
5. *Non-root b-tree page*. Page number is the parent b-tree page.

### Schema Layer
