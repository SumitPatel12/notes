_____
**Created**: 27-10-2024 12:36 am
**Status**: Completed
**Tags**: [[The Pragmatic Programmer]]
**References**: 
______
### TLDR
- Make you code easy to replace in the future.
- Use assertions to detect bad data or impossible scenarios, and crash the program/process if necessary (which is more likely than not). Or better put use assertions to prevent the impossible.
- Crash early rather than late, the later you crash the more havoc it can wreck on your system.
- If the language doesn't automatically drop things make sure you do.
- When in doubt it's always better to reduce scope, i.e make that variable local, make that change affect the smallest possible area etc.
- Delocate resources in the opposite order of their allocation, this helps reduce the risk of leaving orphan processes and other problem of delocating out of order.
- When you allocate same resources in different areas of the code, make sure the allocation is in the same order across all instances. Better yet, make the allocation its own function.
- Always take small steps, big steps are likely to lead to more misses.
- Avoid fortune telling, there is little we know for certain about the future, you will always have to make some assumptions, making the smallest assumption is what you need to do.