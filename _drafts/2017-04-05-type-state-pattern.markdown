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


## Background

The **type state pattern** can be used to statically model an entire interface at compile-time.

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

## Real Use-Case (Prophet)

## Conclusion