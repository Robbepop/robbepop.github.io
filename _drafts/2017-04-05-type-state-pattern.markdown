---
layout: post
title : "The type state pattern"
date  : 2017-04-05 12:00:00 +0100
categories: types pattern rust
---

Hello, I am Herobird and this is my first blog post ever.

This post will give you insights into the type state pattern that I recently discovered while doing some stuff in the Rust programming language.  The techniques used here require a basic understanding of generics and trait bounds.

If you are unfamiliar with Rust in general, there are plenty of awesome tutorials, a welcoming community and even two books to get started!
Scenario

Assuming you want to design and implement something that is similar to the well-known builder pattern:
