_____
**Created**: 14-12-2024 12:03 pm
**Status**: In Progress
**Tags**: #SQLite #Sand
**References**: 
______

### Rust Structure
**Too much public functions and such.** Get down and come up with a better design. 

- [ ] The module structure was not good. Look into that.
- [ ] Errors and handling with types.
- [ ] Right now the reads were from file type which can be generic.
- [ ] Concrete structures for the SQLite on disk data pages
	- [ ] Table Page
	- [ ] Interior Page
	- [ ] Cells
	- [ ] Serial Types
	- [ ] Maybe WAL?
- [ ] Do `varints` need to return size?
	- [ ] Do I maintain an offset on the structure level and not pass it around as a mutable pointer. (I think yes, passing the offset around is a recipe for disaster).
	- [ ] Maybe I should make the offset shared, like RC or something so changes to it reflect across all of what is using it
- [ ] Look at the buffer pool management part of SQLite.

### SQLite Structure to Look Into
- [ ] File format, I looked at it before but another look won't hurt.
	- [ ] File Header
	- [ ] Page Header
	- [ ] Cell Format
	- [ ] Record format
	- [ ] Page size and Checksums
- [ ] WAL
- [ ] B-tree
- [ ] Parser
- [ ] The code pager and retrieval structure. It's in C++ if I'm not wrong. Maybe look at Turso/Limbo as I'm writing in Rust.