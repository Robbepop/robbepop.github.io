---
layout: post
title : "The type state pattern"
date  : 2017-04-05 12:00:00 +0100
categories: types pattern rust
---

I am Herobird and this is my first blog post ever!

---

This post might give you insights into the type system behind the Rust programming language and what kind of interfaces are possible
to create with it using an implementation technique that I call the **type state pattern**.

[Rust is a systems programming language that runs blazingly fast, prevents segfaults, and guarantees thread safety.](https://www.rust-lang.org/)

Take a look into the awesome rust [documentation](https://doc.rust-lang.org/stable/), especially into the [2nd book](https://library.oreilly.com/book/0636920040385/programming-rust/toc) and meet other rustaceans in the [IRC channel](https://chat.mibbit.com/?server=irc.mozilla.org&channel=%23rust-beginners) if you are unfamiliar with Rust.

In the following, I am assuming that the reader has a basic understanding of Rust, its type system and generics.


## Motivation

The **type state pattern** can be used to substitute program behaviour that can be asserted during compile-time instead of code that models those assertions at run-time.

### Example 1: Human

Given the following example `Human`:

```rust
struct Human {
	is_hungry : bool,
	is_thirsty: bool
	is_sleepy : bool
}

impl Human {
	fn eat(&mut self);   // Should only eat while being hungry.
	fn drink(&mut self); // Should only drink while being thirsty.
	fn play(&mut self);  // Playing makes hungry and thirsty,
	                     // cannot play while being tired.
	fn work(&mut self);  // Working makes tired ...
	fn sleep(&mut self); // Should only sleep while being tired
}
```

In this toy example we constructed a `Human` data type that has got different states and methods that
are associated to those states in a certain way. Some examples are ...

- A human can only eat while being hungry.
- Playing makes hungry and thirsty.

To model this behaviour we could simply use a runtime mechanism and use either `Result`, `panic` or
another error handling strategy - test code to validate the wanted behaviour is also always neat!

A partial example implementation via panics with some pre- and post-conditions for our invariants.

```rust
	fn eat(&mut self) {
		assert!(self.is_hungry); // pre-condition
		println!("Yummi Yummi!");
		is_hungry = false;
		assert!(!self.is_hungry); // post-condition
	}

	fn play(&mut self) {
		// Note: no pre-condition!
		println!("Playing is soooooo fun!");
		is_hungry = true;
		is_thirsty = true;
		assert!(self.is_hungry);  // post-condition #1
		assert!(self.is_thirsty); // post-condition #2
	}
```

Using the type-state-pattern we could model the entire behaviour of our class `Human` to meet all of those invariants at compile-time.
This implies that we no longer would be in need of assertions and wouldn't need tests to check those.

### Example 2: Builder

The [builder pattern](https://en.wikipedia.org/wiki/Builder_pattern) is a commonly used design pattern.
Its main purpose is to lead construction of complex objects in a way that is sane for developers and in a transparent way.

A toy example is following:

The computer data type itself:

```rust
struct Computer {
	/// The owner's name
	owner: String,

	/// Every computer requires a CPU
	cpu: CpuKind,

	/// A computer may have a GPU
	gpu: Option<GpuKind>,

	/// Precomputed MegaHertz
	mhz: usize,

	/// The amount and capacity of the main memory
	rams: Vec<usize>,

	/// The computer's unique mac address
	mac: MacAddress
}
```

Some utility types:

```rust
struct MacAddress { ... }
enum CpuKind { Intel, Amd, ... }
enum GpuKind { Nvidia, Amd, IntelHD, ... }
```

And the builder type for the computer:

```rust
struct ComputerBuilder {
	/// The owner's name
	owner: String,

	/// Every computer requires a CPU
	cpu: Option<CpuKind>,

	/// A computer may have a GPU
	gpu: Option<GpuKind>,

	/// The amount and capacity of the main memory
	rams: Vec<usize>,
}
```

Note that some invariants must be hold for this implementation:

Every computer ...

- stores the name of its owner. (INV 1)
- has one and only one CPU. (INV 2)
- may have a GPU. (INV 3)
- has one or more RAMs, each with at least 1 capacity. (INV 4)

It is easy to model those invariants at run-time with the following implementation:

```rust
impl ComputerBuilder {
	/// This constructor allows us to assert that the owner is always set.
	pub fn with_owner(owner_name: String) -> ComputerBuilder {
		ComputerBuilder{
			owner: owner_name,
			cpu: None, // not set, yet !!
			gpu: None,
			rams: vec![] // no entries, yet !!
		}
	}

	pub fn cpu(mut self, cpu: CpuKind) -> ComputerBuilder {
		self.cpu = Some(cpu);
		self
	}

	pub fn gpu(mut self, gpu: GpuKind) -> ComputerBuilder {
		self.gpu = Some(gpu);
		self
	}

	pub fn add_ram(mut self, ram: usize) -> ComputerBuilder {
		self.rams.push(ram);
		self
	}

	pub fn done(self) -> Computer {
		// calculate other stuff, such as ...
		//   - mac address
		//   - mhz
		// assert that invariants, such as having at least one RAM are met
		Computer{ ... }
	}
}
```

### Common Questions:

- What is our error handling strategy? `panic` vs `Result` ?
- How to handle cases where we set an attribute, such as `builder.cpu(..)`, multiple times?
- What happens if we forget setting `cpu` at all?
- How do we handle the case of zero Rams?

## Let the compiler handle these things! (A first attempt ...)

First let us collect all possible states in which our `ComputerBuilder` may be.

- CPU may be **set** or **unset**
- GPU may be **set** or **unset**
- **zero** or **more** RAMs have been added

A state in which we may call `done` is a state where

- CPU must be **set**
- **more** than zero RAMs must have been added

So to model this behaviour we need to introduce new type states.
Naively this could be done via new named types:

```rust
struct ComputerBuilder_UnsetCPU_UnsetGPU_ZeroRam { ... }
struct ComputerBuilder_UnsetCPU_UnsetGPU_MoreRam { ... }
struct ComputerBuilder_UnsetCPU_SetGPU_ZeroRam { ... }
struct ComputerBuilder_UnsetCPU_SetGPU_MoreRam { ... }
struct ComputerBuilder_SetCPU_UnsetGPU_ZeroRam { ... }
struct ComputerBuilder_SetCPU_UnsetGPU_MoreRam { ... }
struct ComputerBuilder_SetCPU_SetGPU_ZeroRam { ... }
struct ComputerBuilder_SetCPU_SetGPU_MoreRam { ... }
```

The initial constructor could now look like this: (Look at the **returned type** !)

```rust
impl ComputeBuilder {
	pub fn with_owner(owner_name: String) -> ComputerBuilder_UnsetCPU_UnsetGPU_ZeroRam {
		ComputerBuilder_UnsetCPU_UnsetGPU_ZeroRam{
			owner: owner_name,
			cpu: None, // not set, yet !!
			gpu: None,
			rams: vec![] // no entries, yet !!
		}
	}
}
```

With an example implementation of `fn cpu`:

```rust
impl ComputerBuilder_UnsetCPU_UnsetGPU_ZeroRam {
	pub fn cpu(mut self, cpu: CpuKind) -> ComputerBuilder_SetCPU_UnsetGPU_ZeroRam {
		self.cpu = Some(cpu);
		self.into() // here we also need a well-defined transition
		            // from ComputerBuilder_UnsetCPU_UnsetGPU_ZeroRam
		            //   to ComputerBuilder_SetCPU_UnsetGPU_ZeroRam
	}
}
```

This approach is extremely flawed for the following downsides:

- State space explodes exponentially and so we need many many additional types.
- The need to define transitions from type-state to others requires tons of boilerplate code.
- A lot of methods need to be defined for many types for all possible transitions.

That's why I will not continue to focus on this naive approach to deal with this.

## Generics to the rescue



## Real Use-Case (Prophet)

## Conclusion