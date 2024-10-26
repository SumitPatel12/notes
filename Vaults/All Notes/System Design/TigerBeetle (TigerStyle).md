_____
**Created**: 22-10-2024 08:12 am
**Status**: In Progress
**Tags**: #SystemDesing [[System Design]]
**References**: [TigerStyle Video](https://www.youtube.com/watch?v=w3WYdYyjek4) [TigerStyle Github](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/TIGER_STYLE.md)
______

### When You Find A Hard Problem Where Do You Start
- Choose stuff (language and technology) for your use-case and not from what is currently in trend 📈.
- Wait before you code and with *TigerSyle*, even when you code its going to be slower than usual.

So what do you look for in TigerStyle:
- See the Whole picture, coding is not the only thing:
	- It's about the design that comes before you start to code.
	- It's the testing, during and after we are done coding.
	- It's about the other team that has to debug in prod.

### The Equation
The following equation sums up what we are trying to look at: **`Time = Design + Coding + Testing + Incidents`**
We ask ourselves:
1. If we ship how good is it going to be?
	1. We could ship fast but then what? How good will it be?
	2. How likely is it to take down production? At what cost?
	3. Will the thing we put together fast achieve what we designed it for?
	4. Will it be valuable?
2. How often as engineers do you think about how your choice of technology could make or break the product a couple year s from now?
	1. Will it make the product thrive?
	2. Will it thrive?
	3. How will it affect marketing?
	4. How will it impact hiring for the future?

#### You look at bigger picture and decide to go fast in some places and slower in others.
- *Design*: 
	- Now here we got ourselves a beast.
	- Design is not just what you will code/build, it is also *what you will not build*, spending time perfecting your design will have exponential impact on the coding phase. You can design code away.
	- Then again it doesn't mean you're wistfully waiting for the best iteration of whatever you are trying to build. This would cost you more time, which is the resource we are trying to conserve. Be hasty with the design and you'll end up with a **bad design**. Problems are going to take longer and longer to fix.
	- You want to have the design roughly right and thats when you can think of starting to code. You want to find a global maxima of the design and not a local one though.
	- *Its better to sketch multiple times in the design phase, sketches are cheap, rewrites are expensive.*
- *Coding*: You can increase the coding speed, type faster, or hire more people but at what cost? The gains might me 2x speed which is marginal at best in the programming world.
- *Testing*: 
	- This dominate the development of complex systems. 
	- Testing these systems take a magnitude of time more than building them. And sometimes bugs are not even reproducible.
	- What if we find a way to reproduce every bug exactly as it occurred? Sort of a time machine.
- *Incidents*: These are likely not within our control. Our hands are tied on this one.


### Four Colours
1. *Network*: 
2. *Disk*: 
3. *Memory*: 
4. *CPU*: 

### Asserions

### The Time Machine
