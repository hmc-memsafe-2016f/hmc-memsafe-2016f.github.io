---

layout: assignment
title: "The Final Project"

---

## The Motivation

[Graphs][graph] are the mathematical structure for encoding relationships. They
typically consist of a set of vertices connected by edges that may be directed
and/or have weights. In this assignment you'll be building a data structure for
representing weighted, directed graphs. You'll also demonstrate the strength of
your data structure by using it to implement two graph algorithms.

## Project Overview

To complete your project you must complete the following:

   1. A review of the API of `petgraph`.
   1. A specification of your own API
   1. A description of the algorithms you will implement, including how they
      work
   1. An implementation of your data structure
   1. An implementation of the algorithms
   1. A demonstration that the algorithms work (tests cases and/or a GUI)

Items 1 through 3 are due Tuesday, 6 December 2016 at 4:15 pm. Items 4 through 6 are due
Thursday, 15 December 2016 at 10:00 pm.

Submissions should be made by submitting a pull-requests against
[this](https://github.com/hmc-memsafe-2016f/final-project) (empty) repository.

## Groups & Grading

You may do this project with someone else. The advantage of doing so is that two
of you can do a more awesome project with your combined wits and manpower. The
disadvantage is that you'll have to coordinate with them and make sure that you
both contribute fairly to the completion of the project. The choice is yours.

There will also be opportunities for bonuses. I'd be excited to hear about
stretch goals that you think would make good bonuses. The following will
definitely be bonuses:

   * Allow the user to store generic (type-safe) data at nodes and edges. That
     is, make your graph generic over the type attached to nodes and edges. The
     allows you to express generalized weights at nodes and edges.
   * Implementing an API that does a particularly good job of catching errors at
     compile-time. A good example of this would being able to implement a graph
     search without any `unwrap`s.
   * Implement a GUI for one or both of your graph algorithms
   * Implement a parallel graph algorithm. You need not achieve any particular
     speedup.
   * Implement a particularly challenging or complex graph algorithm (although
     you need to clear the algorithm with me up front)

## Project Components Detail

### API Review

You should review the API for [petgraph][petgraph] write an analysis of this.
This should include

   * a description of the high-level features the API provides
   * an analysis of how safe using the API is (where could a user cause a panic?)
   * a description of strong and weak points of the API
   * and a high-level guess at how `petgraph` might represent graphs under the hood

However, you should not look at the implementation of `petgraph` during this
stage, or at any point during the project.

### API Specification

You should specify you API by providing an (unimplemented) skeleton of your
graph library. This should include all the types you intend to expose and
public methods/functions you intend to implement. These items should have
triple-slash comments with explanations of their purpose and examples. At this
stage the examples need not execute properly.

You need not flesh out the fields for your types, or how you'll implement the
methods, but you should be confident your API may be feasibly implementable.

Above I refer to triple-slash comments and examples. This is setting foot in
Rust's documentation system. Items, field, and enum variants can all be
proceeded by triple-slash comments that contain markdown which may contain rust
code blocks. The `rustdoc` command will turn these comments into `std`-style
documentation of your library. `cargo` will also eagerly compile and test any
rust code blocks when you run `cargo test`, to ensure your examples stay
up-to-date.

As an example of this is a project which contains the following code in
`src/lib.rs`:

```rust
/// This is MyBox!
/// 
/// ## Examples
///
/// ```rust
/// use omnom::MyBox;
/// let x = MyBox::new(5);
/// assert_eq!(*x, 5);
/// THERE SHOULD BE TRIPLE TICKS HERE BUT VIM HIGHLIGHTING IS BAD
pub struct MyBox {
    b: *mut i32,
}

impl Drop for MyBox {
    fn drop(&mut self) {
        println!("MyBox dropped");
    }
}

impl std::ops::Deref for MyBox {
    type Target = i32;
    fn deref(&self) -> &Self::Target {
        unsafe { &*self.b }
    }
}

impl MyBox {
    /// Creates a new `MyBox`.
    pub fn new(i: i32) -> Self {
        MyBox { b: Box::into_raw(Box::new(i)) }
    }
}
```

Running `cargo doc` generates the following documentation in `target/doc`:

![Rustdoc Output][doc]

### Algorithm Description

You should describe the two algorithms you will implement. Explain their inputs,
outputs, and what they compute. Explain _how_ they compute this. You may choose
to use pseudocode to do this, but you do not need to.

One of these algorithms should be chose from Djikstra's, Ford-Fulkerson, Floyd
Warshall, Prim's, or Kruskal's. The other can be whatever you want (it could be
another one from this list, but need not be).

You should submit these descriptions as a markdown file in your project.

### Graph & Algorithm Implementation

You should implement the API you promised your graph would have. You should then
use this to implement the graph algorithms you previously outlined.

### Demonstration

You should write a test suite to convince a reader (and more importantly,
yourself) that your graph implementation and algorithm implementation are
correct. You may also implement a GUI for you algorithms for bonus points.


[graph]: https://en.wikipedia.org/wiki/Graph_(discrete_mathematics)
[petgraph]: https://docs.rs/petgraph/0.4.1/petgraph/
[doc]: /images/2016-11-26-rustdoc.png
