_____
**Created**: 19-11-2024 08:38 am
**Status**: Completed
**Tags**: #Software_Architecture [[Software Architecture]]
**References**: 
______

### Evolutionary Architecture
An evolutionary architecture supports guided, incremental change across multiple domains.

#### Guided Change
Each project or undertaking will have multiple characteristics. It is the duty of the architect to identify the most important ones, and guide the changes in such a way that these characteristics are protected.

As the architecture evolves, we need mechanisms in place that evaluate how the changes impact the important characteristics we defined during the initial phases. [[Fitness Functions]] are the way to do this.

#### Incremental Change
This encompasses two things:
1. How teams build software incrementally.
2. How they deploy it.
For instance, an example of incremental change can be when we have one shared service that is used by multiple others and now some requirements have changed. We need to modify that one shared service. Incremental way would be that the new and original services can run in parallel and services that depend on those can make the shift at their leisure.

#### Multiple Architectural Dimensions
While designing the architecture the architect must take into account all of the *dimensions of the architecture*. These are the parts that fit together in often orthogonal ways.
Common dimensions to consider:
- *Technical:* The implementation parts of the architecture.
- *Data:* The database schemas, table layouts, data layer, etc.
- *Security:* Defines the policies around security, not just in terms of software but other aspects as well.
- *Operational/System:* How the designed architecture maps to the existing physical or virtual infrastructure we have in place.
To build an evolvable system architects must think about how the system might evolve across all of the important dimensions. And once again to protect the integrity of these dimensions we use [[Fitness Functions]].