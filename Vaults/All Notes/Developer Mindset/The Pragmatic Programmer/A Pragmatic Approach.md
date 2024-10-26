_____
**Created**: 25-10-2024 09:01 pm
**Status**: Completed
**Tags**: [[The Pragmatic Programmer]]
**References**: 
______

- Good design is easier to change than bad design.
	- When Designing keep *ETC: Easy to Change* in mind
- DRY: Don't repeat yourself.
- Make it easy to reuse.
- Eliminate effects between unrelated things.
- Look for areas that define the system or in your opinion have the highest risk factor and code them first.
- Prototype to learn.
- Program close to the problem domain.
- Estimate to avoid surprises.
### Areas to Look for in Architectural Prototypes
1. Are the responsibilities of the major areas well defined and appropriate?
2. Are the collaborations between the major components well defined?
3. Is the coupling minimised?
4. Do you see possibility of duplication?
5. Are the interfaces well defined with some acceptable constraints?
6. *Does every module have a path to the data it might need access to? Is the data available when the module needs it?*

### What to Keep in Mind for Estimates
- Understand what is being asked.
- Build a model of the system.
- Break that model into components.
- Give each parameter a value.
- Calculate the answers.
- Keep track of your estimation prowess.