---

layout: assignment
title: "Week 1: A Stack"

---

This week you'll be getting familiar with Rust's owenership system by
implementing a simple, singly-linked list.

Your list should implement the follow trait:

```rust
pub trait Stack<T> {
    fn new() -> Self;                                       // O(1)
    fn push_front(&mut self, item: T);                      // O(1)
    fn pop_front(&mut self) -> Option<T>;                   // O(1)
    fn peek_front(&self) -> Option<&T>;                     // O(1)
    fn len(&self) -> usize;                                 // O(1)
    fn remove_first(&mut self, _: &T) -> Option<T> { None } // O(n)
    fn reverse(&mut self) { }                               // O(n)
}
```
for all `T` which are `Eq`. The last two functions are _optional_.

The test set and starter code for the assignment may be found in the [wk1
repository][wk1-github]. You can clone the repository get started, use `cargo
build` to build, and `cargo test` to run the test set using your list.
Ultimately, you'll submit `src/stack.rs` to the course auto-grader, which will
run exactly the tests provided to you, and report the result.

This assignment does not require a lot of code, in fact, it requires much less
code than last week's. However, learning to adhere to the ownership system is
challenging at first - you will need to think very carefully about the owner of
the values your code manipulates. Are you trying to change that owner or not?

Furthermore, I encourage you to keep thinking once you get the compiler to
accept your code. Ask yourself if the code you've written could be expressed
more elegantly using Rust's standard library. With appropriate use of Rust
idioms the functionality required by this assignment can be written very
concisely - in fact the sample solution is less than 40 lines long, without the
bonus functions.

## Tips

Important things to think about:

   * How would one implement a singly linked list in C++? In particular, what
     members would your struct's have?
      * Nodes likely have pointers to nodes elsewhere on the heap. How does one
        represent a heap-pointer in Rust?
   * What are the ownership semantics of a singly linked list? The list must
     (transitively) own the whole list. Node may "own" other nodes...

## Bonuses

There are three bonuses. I'd recommend them strongly -- the first two are good
practice and the last is really neat.

   1. Implement `reverse`, which reverses the order of the elements in the
      stack.
   1. Implement `remove_first`, which removes the first occurance of some
      element from the stack.
   1. Run the tests using `cargo test -- --ignored`. Note that
      `tests::mystery::wat` fails. Figure out why and fix it! Useful tools
      include `cargo test -- --ignored --no-capture` and `rust-gdb`.

[wk1-github]: https://github.com/hmc-memsafe-2016f/wk1-starter
