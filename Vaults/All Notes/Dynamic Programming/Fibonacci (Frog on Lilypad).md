_____
**Created**: 01-10-2024 09:28 am
**Status**: In Progress
**Tags**: #Coding #Dynamic_Programming [[Dynamic Programming]]
**References**: [Numberphile LilyPad](https://www.youtube.com/watch?v=JBkmR3rzz5M)
______
### Question
There are **n Lilypads** and it can jump only in the forward direction. It can either jump one place or 2 places. How many different way are there through which the frog can reach the *nth lilypad*?

### Intuition
Lets say x$_{target}$ is the number of ways you can reach the target lilypad. 
Now since the frog can only jump 1 or 2 places, to reach x$_{target}$ we must have either jumped one form x$_{target-1}$ or jumped two from x$_{target-2}$. This makes **x$_{target}$ = x$_{target-1}$ + x$_{target-2}$**
**P.S. This is just the fibonacci series/numbers.**

#### Base Cases
x$_{0}$ = 1
x$_{1}$ = 1
x$_{2}$ = 2

### Solution
Just make an array with a\[1\] = 1 and a\[2\] = 2 then using the above formula calculate up till the Nth term.

### Bonus Mathematical Solution