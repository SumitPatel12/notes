_____
**Created**: 17-03-2025 10:57 pm
**Status**: Not Started
**Tags**: #Database #Database_Indexes [[Database]]
**References**: 
______

### Two Basic Index Types
1. *Ordered Index:* Values are stored in some order.
2. *Hash Index:* The values are stored in buckets matching the output of some hash function.

### Evaluation Parameters for Indices
1. *Access Type:* Includes finding a record with a specific search attribute and range queries.
2. *Access Time:* Time taken to find an item of set of items given a specific attribute or range of attributes.
3. *Insertion Time:* Includes time taken to find the point at which the insertion needs to take place plus the time taken to update the index.
4. *Deletion Time:* Includes the time taken to locate the record to be deleted plus the index deletion time.
5. *Space Overhead*

### Ordered Indices
Ordered index stores the value of the **search key** in sorted order, and associates with each search key the records that contain it.

Consider an ordered index for a file that already stores on the disk based on some *sequential order*, if this ordered index has the search keys which also define the sequential order of the file then it is called a *clustering index* or *clustered index.* Note that more often that not the search key for a clustering index is the **primary key** in which case it may also be called a **primary index**. Similarly an index that would define the order different from the sequential order of the file is called a *non-clustering index* or *non-clustered index.*

#### Dense and Sparse Indices
1. **Dense Index:** 
	- An index entry appears for every search-key value in the file.
	- For a clustering dense index, the search-key would have a pointer to the first record match the search-key value, the rest can be accessed sequentially since clustering index would mean that they are indeed stored sequentially.
	- In a dense non-clustering index, we must store a list of pointers to every value the search-key matches.
	- ![[Dense Index.png]]
2. **Sparse Index:**
	- As opposed to dense index, the sparse index would have an entry matching only some of the search-key values.
	- Evidently sparse indices can only be used if the relation is stored in sorted order of the search key i.e. if the index is clustering.
	- ![[Sparse Index.png|700]]

As you can easily observe finding a key in the dense index would be much easier since it has all the possible keys, while in the sparse index we'd have to look up the upper or lower bound on the search key and follow file pointers till we find the desired value.
They both have their pros and cons. Dense indices require more space and have a higher overhead for insertion and deletion. The trade-off must be considered by the DB designer, on basis of the parameters we described before.

A common pattern that you might observe is that there is a sparse index on the first element of each disk block, this is because the cost of bringing a block from the disk to memory is much higher than scanning it after it has been fetched to memory. *A sparse index can thus be used to identify the block in which the desired value resides, scanning the block for the value will come at a negligible cost compared to fetching the block.*

Ofcourse nothing is as simple as we make it out to be, for records for one search key value can occupy several blocks, and this would not sit well with the approach we discussed above. For a more generalised approach we have **Multilevel Indices.**

##### Multilevel Indices
Problems start arising when you start scaling so let's consider a scenario where you have to make an index on a table with 1mil rows. Since index entries are more compact than the actual entries let's say 100 entries for the index are sized at *4KBs*, meaning the index occupies *10,000 blocks*, it's also normal to sometimes have 100mil rows, in which case we'd have an index spanning *1mil blocks* or *4GBs* which in itself would then need to be stored as sequential file because it wouldn't fit into the memory.

Index fitting in memory would be ideal, and lookup times would be minimal for them, but for the scenario we discussed above we'd require fetching blocks from the disk which as established earlier is a costly operation. Since the index is stored sequentially we could use a searching algorithm say *binary search* but that would still be a costly operation. 
Say we use **binary search** on an index spanning **b** blocks, then it'd require **ceil(log2(b))** *random block reads*. Which comes down to 14 reads for the 10000 block index. And this my friends does not include any *overflow pages*. 

The solution is to treat the index as a file to be indexed. The index being indexed is called the *inner index* and the *spare index* being created for the inner index is called the *outer index.* Then locating a record is simple activity of binary searching the outer index for the largest search-key value less than or equal to the one we need to find. Follow its pointer to the original index, fetch the block, scan it for the desired record and follow its pointer to the record.

In the example above 10,000 block index would have an outer index of 100 blocks which can easily fit in memory, in which case we only fetch **1 block as opposed to the 14 blocks** we fetched using binary search on the original index.

`Note that mostly the node size of a B+Tree is chosen to be the same as the size of a disk block.`