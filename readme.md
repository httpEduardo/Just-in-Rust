# Legion ECS

[![Crates.io](https://img.shields.io/crates/v/legion.svg)](https://crates.io/crates/legion)
[![Documentation](https://docs.rs/legion/badge.svg)](https://docs.rs/legion)
[![CI](https://github.com/httpEduardo/Rust-Game/workflows/CI/badge.svg)](https://github.com/httpEduardo/Rust-Game/actions)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

Legion aims to be a feature-rich, high-performance Entity Component System (ECS) library for Rust game projects with minimal boilerplate.

## Table of Contents

- [Features](#features)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Core Concepts](#core-concepts)
  - [Worlds](#worlds)
  - [Entities and Components](#entities-and-components)
  - [Querying Entities](#querying-entities)
  - [Systems and Scheduling](#systems-and-scheduling)
- [Performance](#performance)
- [Resources](#resources)
- [Contributing](#contributing)
- [License](#license)

## Features

- **High Performance**: Built from the ground up with performance as a primary concern
- **Minimal Boilerplate**: Clean, ergonomic API that gets out of your way
- **Parallel Execution**: Built-in support for parallel system execution
- **Flexible Queries**: Powerful query system with filtering capabilities
- **Serialization**: Optional serialization support via `serde`
- **Type Safety**: Compile-time type checking for components and systems
- **Archetype-based Storage**: Efficient storage model inspired by Unity DOTS
- **Zero-cost Abstractions**: Performance without compromise

## Installation

Add Legion to your `Cargo.toml`:

```toml
[dependencies]
legion = "0.3"
```

### Optional Features

Legion supports several optional features:

```toml
[dependencies]
legion = { version = "0.3", features = ["parallel", "serialize", "codegen"] }
```

- **`parallel`** - Enables parallel system execution (enabled by default)
- **`serialize`** - Adds serialization support via `serde` (enabled by default)
- **`crossbeam-events`** - Event channel support (enabled by default)
- **`codegen`** - Proc-macro utilities for systems (enabled by default)
- **`extended-tuple-impls`** - Extended tuple implementations for more components
- **`wasm-bindgen`** - WASM support via wasm-bindgen
- **`stdweb`** - WASM support via stdweb

## Quick Start

Here's a simple example to get you started:

```rust
use legion::*;

// Define your components
#[derive(Clone, Copy, Debug, PartialEq)]
struct Position {
    x: f32,
    y: f32,
}

#[derive(Clone, Copy, Debug, PartialEq)]
struct Velocity {
    dx: f32,
    dy: f32,
}

fn main() {
    // Create a world to store entities
    let mut world = World::default();

    // Create entities with components
    let entity = world.push((
        Position { x: 0.0, y: 0.0 },
        Velocity { dx: 1.0, dy: 0.5 },
    ));

    // Or insert multiple entities at once (more efficient)
    let entities: &[Entity] = world.extend(vec![
        (Position { x: 0.0, y: 0.0 }, Velocity { dx: 1.0, dy: 0.0 }),
        (Position { x: 1.0, y: 1.0 }, Velocity { dx: 0.5, dy: 0.5 }),
        (Position { x: 2.0, y: 2.0 }, Velocity { dx: 0.0, dy: 1.0 }),
    ]);

    // Query entities
    let mut query = <(&mut Position, &Velocity)>::query();
    for (mut pos, vel) in query.iter_mut(&mut world) {
        pos.x += vel.dx;
        pos.y += vel.dy;
    }
}
```

## Core Concepts

### Worlds

Worlds are collections of entities where each entity can have an arbitrary set of components attached. A world ensures that all entities have unique IDs within that world.

```rust
use legion::*;

let mut world = World::default();
```

### Entities and Components

**Entities** are unique identifiers that represent game objects. The world creates a unique ID for each entity upon insertion.

**Components** are data that can be attached to entities. A component can be any type that is `'static`, `Sized`, `Send`, and `Sync`:

```rust
// Components are just regular structs
#[derive(Clone, Copy, Debug, PartialEq)]
struct Position { x: f32, y: f32 }

#[derive(Clone, Copy, Debug, PartialEq)]
struct Velocity { dx: f32, dy: f32 }

#[derive(Clone, Copy, Debug, PartialEq)]
struct Health(i32);

// Insert a single entity
let entity = world.push((
    Position { x: 0.0, y: 0.0 },
    Velocity { dx: 0.0, dy: 0.0 },
    Health(100),
));

// Insert multiple entities (more efficient for bulk insertion)
let entities = world.extend(vec![
    (Position { x: 0.0, y: 0.0 }, Health(100)),
    (Position { x: 1.0, y: 1.0 }, Health(80)),
    (Position { x: 2.0, y: 2.0 }, Health(120)),
]);
```

You can access entities via `Entry` API, which allows you to:
- Query what component types are attached
- Get component references
- Add or remove components dynamically

```rust
// Access a specific entity
if let Some(mut entry) = world.entry(entity) {
    // Add a component
    entry.add_component(Velocity { dx: 1.0, dy: 1.0 });
    
    // Get a component
    if let Ok(pos) = entry.get_component::<Position>() {
        println!("Position: {:?}", pos);
    }
    
    // Remove a component
    entry.remove_component::<Velocity>();
}
```

### Querying Entities

Queries provide efficient iteration and filtering of entity components. Queries are defined by two parts:

- **Views**: Declare what data to access and how to access it
- **Filters**: Decide which entities to include in the results

```rust
// Query for entities with Position and Velocity components
let mut query = <(&mut Position, &Velocity)>::query();

for (pos, vel) in query.iter_mut(&mut world) {
    pos.x += vel.dx;
    pos.y += vel.dy;
}

// Query with filters
use legion::query::filter::*;

let mut query = <(&Position, &Health)>::query()
    .filter(component::<Velocity>());

for (pos, health) in query.iter(&world) {
    println!("Entity at {:?} has {} health", pos, health.0);
}
```

#### Query Filters

Legion provides powerful filtering capabilities:

```rust
// Filter for entities that have a component
let query = <&Position>::query()
    .filter(component::<Velocity>());

// Filter for entities that don't have a component
let query = <&Position>::query()
    .filter(!component::<Velocity>());

// Combine filters with boolean logic
let query = <&Position>::query()
    .filter(component::<Velocity>() & !component::<Health>());
```

### Systems and Scheduling

Systems are functions that operate on component data. Using the `#[system]` macro, you can easily define systems:

```rust
use legion::*;

#[system]
fn update_positions(world: &mut World) {
    let mut query = <(&mut Position, &Velocity)>::query();
    for (mut pos, vel) in query.iter_mut(world) {
        pos.x += vel.dx;
        pos.y += vel.dy;
    }
}

#[system]
fn print_positions(world: &World) {
    let mut query = <&Position>::query();
    for pos in query.iter(world) {
        println!("Position: x={}, y={}", pos.x, pos.y);
    }
}
```

Systems can be executed in parallel using a `Schedule`:

```rust
use legion::*;

// Create a schedule and add systems
let mut schedule = Schedule::builder()
    .add_system(update_positions_system())
    .add_system(print_positions_system())
    .build();

// Execute the schedule
schedule.execute(&mut world);
```

Legion automatically analyzes system data access patterns and runs systems in parallel when safe to do so, maximizing performance.

## Performance

Legion is designed for high performance:

- **Archetype-based Storage**: Entities with the same component types are stored contiguously in memory for excellent cache locality
- **Parallel Execution**: Systems can run in parallel automatically when their data access doesn't conflict
- **Minimal Overhead**: Zero-cost abstractions mean you only pay for what you use
- **Efficient Queries**: Query iteration is optimized for fast traversal of component data

For benchmarks comparing Legion with other Rust ECS libraries, see the [ECS Bench Suite](https://github.com/rust-gamedev/ecs_bench_suite).

## Resources

- **[Documentation](https://docs.rs/legion)** - API documentation and guides
- **[Legion Book](docs/book/src/introduction.md)** - Detailed book covering Legion's internals
- **[GitHub Repository](https://github.com/TomGillen/legion)** - Source code and issue tracker
- **[ECS Bench Suite](https://github.com/rust-gamedev/ecs_bench_suite)** - Performance benchmarks
- **[Examples](examples/)** - Example projects using Legion

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request. For major changes, please open an issue first to discuss what you would like to change.

### Development Setup

```bash
# Clone the repository
git clone https://github.com/TomGillen/legion.git
cd legion

# Build the project
cargo build

# Run tests
cargo test

# Run benchmarks
cargo bench
```

## License

Legion is distributed under the [MIT License](LICENSE).

```
MIT License

Copyright (c) Thomas Gillen

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

**Built with ❤️ for the Rust game development community**