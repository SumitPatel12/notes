_____
**Created**: 07-11-2024 09:02 am
**Status**: In Progress
**Tags**: #DataOrientedDesign [[System Design]]
**References**: [Andrew Kelly Practical Data Oriented Design](https://www.youtube.com/watch?v=IroPQ150F6c)
______

### Ramblings while watching the video.
- Multiplication is faster than memory access in many cases.
- The CPU is fast but main memory is slow.
	- You can think of it like this, every access to memory needs to go through a cache line. 
	- Typical cache line is 64 bytes, and having a cache miss will result in fetching a new cache line. 
	- Fetching the new one is generally going to be slower.
- Do more stuff with the CPU and less stuff with the memory.