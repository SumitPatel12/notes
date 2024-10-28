_____
**Created**: 26-10-2024 02:16 pm
**Status**: Completed
**Tags**: [[The Pragmatic Programmer]]
**References**: 
______
### TLDR
- The command shell is a really good tool to do some general purpose text formatting, getting familiar with [[Regular Expressions]] and [[Grep]] and other such tools will take you a long way.
- Setting up aliases for the most commonly used command-line function you use is always a great idea.
- Learn the commands that make life easier (Neovim BTW).
- Take time to learn you *editor* and *its extension language*. You may often find extensions for your use case, if not maybe build you own one.
- *Debugging:*:
	- Its easy to blame someone for the problem and not focus on a fix, don't do that. Always focus on the fix.
	- While you debug do not panic.
	- It's convenient to fix the symptom, to deploy a quick fix and be done. Well those always come to haunt you again. So, better to fix the bug at the root the first time around.
	- Make sure to have a failing test for the scenario you are trying to fix.
	- If the bug was a result of some bad data being propagated, try to get checks in to prevent it in the future.
	- Always take time to think whether other parts of the system are likely to suffer from the same bugs.

### Debugging Checklist
- [ ] Is the problem a symptom or the actual bug?
- [ ] You have to explain in detail what caused the bug, what and how do you say it?
- [ ] Are other parts of the system being affected by the same bug?