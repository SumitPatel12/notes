_____
**Created**: 14-12-2024 12:13 pm
**Status**: Completed
**Tags**: #Database_Internals [[Database Internals]]
**References**: 
______

Data files or primary files are generally implemented as one among the following:
1. `Index-organised tables (IoT)`
	- These store records/data in the index itself.
	- Records are stored in key order, making range scans easier.
	- Storing data in indexes saves at least one disk seek as the end of the index you do not have to look up one more address for the actual record.
2. `Heap-organised tables`
	- Records in this format are not required to follow any particular order, and most of the time are placed in write order.
	- This eliminates the need of file reorganisation when new pages are appended.
	- The downside being they need additional structures pointing to the location of the records for it to be searchable.
3. `Hash-organised tables`
	- Records are stored in buckets.
	- The hash value of the key determines which bucket the record belongs to.
	- Records can be stored in write order or by key order to facilitate lookup speed.

***SQLite is IoT.***