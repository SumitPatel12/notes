_____
**Created**: 11-01-2025 03:22 pm
**Status**: In Progress
**Tags**: #Database_Internals[[Database Internals]]
**References**: 
______

WAL consists of log records. Each record has a unique, monotonically increasing *log sequence number (LSN)*. Their contents are generally cached in the *log buffer* and are flushed on disk in a *force* operation. *Force* happens as the log buffer fills up, and can be requested by the transaction manager or the page cache. All logs are to be flushed on the disk in the **LSN order.**

