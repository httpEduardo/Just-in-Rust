# Introduction

Welcome to the Legion book! This book is intended to be a summary overview of legion, including: 
- An overview of how to use it
- Some examples of different use case scenarios
- how it is different than other Entity-Component-Systems in the rust ecosystem
- Overviews of some pertinent internals

This book assumes a general understanding of the concepts of the Entity-Component-System design and data composition 
as a design pattern. If you need a summary of what an ECS is, please see the [Wikipedia article on ECS].

## Design

Legions internal architecture is heavily inspired by the new Unity ECS architecture [^1], while the publicly facing 
API is strongly built upon specs [^2], while expanding on it and learning from many of the faults found in that API.

### Quick Version
The core concept of Legion's design is based around the concept of `Entities`, `Archetypes`, and `Chunks`. 
These core concepts are the building blocks of legion.

#### Worlds
Worlds are collections of `Entities` and `Archetypes`. Each world has it's own collection of entities. A world ensures
all of its entities to have an unique id. While IDs are unique per world, they may not be across different worlds.

#### Entities
Entities are strictly ID's, allocated within a given `World` of legion, which allows uniquely referenced component
instances. Entities may have an arbitrary collection of components attached. The entities in a world may be efficiently
searched and iterated via `Queries` or the `system-macro`.

#### Archetypes
An Archetype is considered a "Grouping of Components and Tags". Entities may have varying numbers and types of 
components; any combination of these tags and components is considered an `Archetype`. In legion, entity storage 
and parallelization of system execution are all centered on this concept of Archetypes. 


#### Queries
Queries provide efficient iteration and filtering of entity components in a world, and thus giving access to them.
Queries are defined by two parts; "views" and "filters". Views declare what data to access, and how to access it. 
Filters decide which entities are to be included in the results.


#### Systems
Systems are essentially an abstraction over queries. Systems can be executed (in parallel) by a `Schedule`.
They are constructed by writing a function with the `system` macro on top of. The `system` macro appends '_system' 
to the function and constructs the complete system on compile time.


## Other resources
* [ECS Bench Suite][ebs]: A suite of benchmarks designed to test and compare Rust ECS library performance across a 
    variety of challenging circumstances.
* [Legion Docs][ld]: the docs for legion

[ebs]: https://github.com/rust-gamedev/ecs_bench_suite 
[ld]: https://docs.rs/legion/0.3.0/legion/

[^1]: https://docs.unity3d.com/Packages/com.unity.entities@0.1/manual/ecs_core.html
[^2]: https://github.com/amethyst/specs

[Wikipedia article on ECS]: https://en.wikipedia.org/wiki/Entity_component_system
