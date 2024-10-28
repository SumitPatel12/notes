_____
**Created**: 27-10-2024 03:58 pm
**Status**: Completed
**Tags**: [[The Pragmatic Programmer]]
**References**: 
______

### TLDR
- Listen to your Lizard Brain. (Its prolly a pea brain for some of us 😆).
- When you're sitting in front of an empty buffer, and can't start writing, tell yourself it's just a prototype (its meant to fail, and in the unlikely event it succeeds, we're gonna throw it away anyways).
- Program Deliberately, if the code works then you should know why it works, otherwise when it fails you'll often find yourself praying.
- **If it works don't touch it, is the most idiotic thing you can do.**
- Often you think its close enough, that's a small difference, etc. and let go of things. These things tend to hide in dark corners and haunt you for the rest of your time. *Close enough it not enough*, reach the exact goal and if not find out where it missed the mark.
- We're all guilty of believing in *Phantom Patterns* some time or the other, while they were just coincidences. It's hard to see through them, so you train yourself to ask is this actually a pattern or some series of happy coincidences?
- *Finding an answer that happens to fit is not the same as finding the right answer.*
- Estimate the order of your algorithm. And test them. ⚔.
- Refactors are not the place to introduce new features.
	- Refactor Early, Refactor Often.
	- Make sure you got some good coverage in terms of tests before your refactor.
	- Take small steps and run tests after each step.
- Testing is not just about finding bugs but also defining boundaries and expectations.
- A test is the first user of the code.
- After you are done with your code, analyse it for how it can go wrong and add all of those to your test file. Also if possible add asssetions and trip wires for it across the code.
- 

### How Do You Program Deliberately? 🤔
- Be aware of what you are writing. A measure of it is, can you explain it to a more junior programmer? If not rethink right now.
- Proceed from a plan.
- Try not to depend on assumptions, if you inevitably have to document those assumptions.
- Spend more time on the important parts, they are often the hardest as well.
- Follow the coding patterns in your code, but don't be a slave to them. If you can think of something that serves better bring it up.

### Security Basic Principles
1. Minimise the Attack Surface Area
2. Principle of Least Privilege
3. Secure Defaults
4. Encrypt Sensitive Data
5. Maintain Security Updates