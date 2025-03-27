_____
**Created**: 23-02-2025 03:23 pm
**Status**: In Progress
**Tags**: #Data_Structures #Hash_Tables [[HashTables]]
**References**: [ClickHouse Hash Tables](https://clickhouse.com/blog/hash-tables-in-clickhouse-and-zero-cost-abstractions)
______
### Applications in Clickhouse
Before anything, a hash table is a data structure that provides constant **average performance for insert, lookup and delete operations.** Deletion as you can guess is not a concern for the `GRUOP BY` aggregation scenario.

Use-case: *Aggregate huge amounts of data fast.*
The aggregation is done using `GROUP BY` clause. Understand that most of the databases implement hash aggregation algorithm in which the input rows are stored in a hash table with the grouping columns as key. ClickHouse is no exception. 

So, choosing the right kind of hash table becomes the critical task, and the parameters involved make it particularly tricky to select the correct one. It depends on the data-types of the grouping columns, the number of unique keys, their total numbers and factors. 

### Hash Table Design
Following are assumed to be the constituents of the Hash Table, they are by no means exhaustive:
- The Hash Function
- A Collision Resolution Method
- Resizing Policy
- Various Possibilities for Arranging its Cells in Memory

#### Choosing a Hash Function
There are loads of [potential choices](https://clickhouse.com/docs/sql-reference/functions/hash-functions) all of which might appear decent choices. Unless you have an ideal distribution, picking an identity hash function for integer would not work out well. It'd have more collisions than predicted. If you think to use a general purpose hash function then, think again if your dataset is just integers than there are specialised functions that would handle that kind of load. Using it over the general function would result in significantly better performance at scale.

On the other hand you also have to consider whether or not you want to use cryptographic hashes. Using one when you don't need comes at the cost of performance. *Sip Hash Function* - cryptographic - has a throughput of 1GB/s while *City Hash Function* has a throughput of 10GB/s. This is not just a bit slower, but magnitudes slower. 

FNV1a is one of the legacy hash function that you'd find around, **don't use it**. It has been deprecated and fails to achieve the standards the competitors have set.

ClickHouse uses *CRC-32C for integers.* While it has some relatively bad distribution, it is good for hash tables because it uses very little CPU time and is quite fast. It's implemented with [dedicated instructions](https://www.intel.com/content/www/us/en/docs/intrinsics-guide/index.html#ig_expand=1436&text=crc) which use only **two or three** cycles. For strings they use a custom hash function build on top of the one we discussed. [Farm Hash](https://opensource.googleblog.com/2014/03/introducing-farmhash.html) is an alternative you may consider if you don't want a custom function.

#### Collision Resolution
A collision occurs when multiple keys fall into the same slot.
Resolution Methods:
- *Chaining:* The table cell is an array or structure of some sort that can hold more than one value. The colliding key can thus be placed/appended to the cell itself.
	- During lookup both the main cell and its child list are checked for the presence of a key. That's how `unordered_map` is used, and it's not effective because, **its's not cache local**, leading to poor performance.
	- Further on the hot path it would load the allocator very heavily, since calling it would become expensive. As a result most of the modern hash tables use **Open address** method. 
	- It's pros would be that we could use it in any case, it wouldn't matter much what hash function was being used.
	
- *Open Addressing:* You just traverse the table until you find an empty slot to place the key. While traversing/probing we may take one cell at a time, two or varying number of skips. It is an implementation detail on how much each consequent lookup should skip.

- *Cuckoo Hashing:* AKA Two-way Hashing, is a more complex method and rightfully more difficult to implement as well. To include it in the hot path would be including additional lookups leading to degraded performance (slowed lookups as we need more fetches from the memory.)

The blog in question talks about three hash tables worth paying attention to, all of which use open addressing.
1. Google Falt Hash Map
2. Google Abseil Hash Map
3. The HashTable at ClickHouse

ClickHouse and Google hash tables use a load factor of 0.5. Abseil Hash Map uses a load factor of 0.9.

**Google Abseil** takes a slightly different approach to hash tables.

##### Clustering and Locality
In case we chose a bad hash function, clusters form which stick together. That leads to checking of keys that are unrelated - essentially killing cache locality. But, thats not the only thing that can kill cache locality, large enough objects can lead to the same behaviour. To overcome this *large objects* can be serialised somewhere and the pointers to them are stored in the hash table.

##### Resizing
First step is to decide how many times to resize. 
There are two ways:
1. *Resize to power of two:* 
	- It is good because no additional overhead is incurred for division, executing in nanoseconds if the table is present in the cache.
	- Since the table is always the size of power of 2, division operator can be replaced with the cheap *bit shift operator.*
2. *Divide by anything else:*

##### Arranging Cells in Memory
The reason to think about this while implementing **open address** is simple, there is always going to be a collision. We've already discussed what to do in this scenario.
We have to loop through cells deciding if the cell is empty and whether or not we can add the key to that cell or if its been deleted. Assuming the memory is initialised, we need to be able to distinguish between the cell being empty (this is because we might have *Null* values).

One option is to ask the user to give us a value that indicates empty and tombstones. These values are never inserted into the table by the user so we can safely assume that the values can identify the respective states.
This method is used  by Google Hash Maps. The main disadvantage of the method being the user has to choose such values that will never be present in the table. It's oftentimes harder than it seems.

*ClickHouse* uses a more advanced method.They do not keep Null cells in the hash table. The special cell for the Null element is kept separate, before inserting or looking up the hash table, we first check if the value is *NULL* or not and process it accordingly. This comes with the downside of adding a branch to the flow, but modern CPUs handle it very well with the branch predictor.

The newest Google Hash Table implements a rather complicated method.
- They have a place to store metadata, it stores information of whether the cell is empty or deleted. Writing it to the hash table would incur additional memory so we keep it as metadata instead.
	- Since it only needs two bits it would be expensive to spend them on this information. You could use a whole byte but then  what of the remaining bits? 
	- In the Google implementation the the first 53 bits of the hash function are used to search  for cells with metadata, and bottom bits of the hash function are in metadata.
	- This is relevant cause then you could use registers or instructions to quickly check if you need to look at the associated cells.
	