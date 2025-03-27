_____
**Created**: 14-12-2024 12:55 pm
**Status**: In Progress
**Tags**: #Database_Internals  [[Database Internals]]
**References**: 
______

Index is a structure that organises data records on disk in a way that facilitates efficient retrieval operations. They are organised as structures that map keys to locations in the data files where the records identified by these keys or primary keys.

An index on a primary file is the primary index, all other indexes are secondary index.

Secondary indexes can point directly to the data record, or simply store the primary key. A pointer to the data record can hold an offset to a heap file or index-organised table. Multiple secondary indexes can point to the same record and a single key may hold multiple entries in a secondary record.

`Clustered Index:` If the order of the data records follow the search key order, the index is a clustered index. Data records are generally stored in the same file.
`Non-clustered Index:` If data is stored in a separate file, and does not follow key order then it is a non-clustered index.