_____
**Created**: 15-04-2025 07:39 am
**Status**: In Progress
**Tags**: #Distributed_Systems #Database [[Distributed Database]] [[Database]]
**References**: 
______

There are kind of two options:
1. Local indexes: 
	- Local secondary index would require to query every partition when we need a point/range lookup. Local secondary indexes might be used in full text search use-cases, and are rare in traditional OLAPs.
	- They are often not worth it because if you think about it index makes more sense as a whole and not partitioned. Local indexes mean creating overhead costs on each node, meaning you make poor use of the cache space.
	- The “different indexes on different replicas” idea has been floated for ages but is largely unsound. The problem with that idea is that it gives every node in your distributed system wildly different operational behavior and no reasonable way to balance workload over resources. You can easily find yourself in situations where the entire cluster is running at the speed of whatever node is having the worst time of it based on transient workload characteristics. Since all your nodes are different and the variance is high, it is pretty pathological. If you want your data organized differently, that’s a different instance/system.
2. Global secondary indexes:
	- You are most likely to see a global secondary index.
	- Global secondary indexes can be used when you don't care much for Write performance, it should be evident that maintaining those would cost you write performance.


### Interleaving message from discord
So, doing a bit of simplification (because in reality it's more complex than this, but I think the simplification is still pretty reasonable): Imagine when you're storing rows in your database, instead of each table being in its own structure, you instead store: 

```python
Table0 Row (1) 
-> Table1 Row (1, key1) 
-> -> Table2 Row (1, key1, key2) 
-> Table1 Row (1, key2) 
... Table0 Row(2) 
-> ... 
```

Then, if these are kept together logically, a write that needs to touch two rows in Table2 Row (1, key1, ...) needs to only know where the rows under Table1 Row(1, key1) live. (More explicitly: you know that the rows in Table2 under Table1(1, key1, ...) should be in the same partition / set of partitions, and furthermore they are likely in the same partitions as other rows in Table1 or Table2 within Table0(1)) Now, imagine you want to search on some value column of Table2, and you always know that it'll always be within Table0(x) -- e.g. WHERE Table0.key = x AND Table2.value_column (conditions). So you put an index, interleaved under Table0(x), on (Table2.key (=Table0.key), Table2.value_column -> Table2 key, Table2.key1, Table2.key2). With lookup, you might need to scan all of the partitions under Table0(1) (which are hopefully a single partition, or in the worst case a very small # of them). That's possibly a little worse than a global index lookup in the worst case, but probably the same O(1) lookup if it's all in the same partition. The advantage, though, is in writes. Because when you insert a row into Table2(key, key1, key2), you're likely writing to the same partition under Table0(key), you should be writing to a single partition (or partitions that you know are "close", e.g. managed by the same couple machines, even if they're not the same exact partition) rather than multiple partitions. That makes the write costs of an index significantly lower compared to a global index where you now need to write to two partitions, possibly stored on servers that are far apart.

Basically, you get significantly better write latency ("I don't have to go write to a separate table structure managed by other machines") when compared to a global index. And as long as all of your lookups are /also/ within a single partition, you don't lose any lookup performance compared to a global index since you only need to look into a single partition.