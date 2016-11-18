---

layout: assignment
title: "Week 4: Unsafe"

---

This week you'll be writing unsafe code for the first time! As you'll no doubt
find, with great power comes great responsibility. For the first time, the
compiler will not be able to protect you from many mistakes you'll encounter.

You'll find tests (but no starter code) [here][starter]. **After completing the
assignment, fill out [the exit questionnaire][survey].**

## Part A: Replace With

Often times when writing code in the functional style we find ourself wanting
to write:

```rust
self: &mut T
*self = expression!(*self)
```

But we can't, because we can't move a borrowed value, even temporarily. So we
end up writing:

```rust
self: &mut T
tmp_self = mem::replace(self, TMP)
*self = expression!(tmp_self)
```

But this may be less efficient (or require LLVM to do more work), isn't
expressing our intention clearly, and may be impossible when there is no
convenient TMP.

For this part you'll implement:

```rust
/// Replaces `*t` with `f` applied to the original `*t`,
pub fn replace_with<T, F: FnOnce(T) -> T>(t: &mut T, f: F);
```

which allows one to use the above programming pattern without providing a
temporary values.

Since you're providing a library, you should make sure the library's entry point
is `src/lib.rs` within the `replace_with` crate. You can test your solution by running

```
cargo test --test required
```

## Part B: Rc

For the second part of this assignment you'll implement `MyRc`, a clone of
`std::rc::Rc`. `MyRc` will allow for multiple "owning" pointers to share the
same underlying data, such that the underlying data is free'd only when all the
`Rc`'s go out of scope.

Refer to the class slides for a more complete description of the idea behind
`Rc`.

Your `MyRc<T>` type must have the following public functions:

   * `fn new(t: T) -> MyRc<T>;` which wraps a T within reference-counted memory.
   * `fn consume(t: T) -> Result<T,MyRc<T>>;` which turns a `MyRc<T>` back into
     a `T` if the `MyRc` was the only remaining `MyRc` to the underlying data.

It must also implement the following traits:

```rust
impl<T> Deref for MyRc<T> { // refer to the underlying data
   type Target = T;
   // TODO
}
impl<T> Clone for MyRc<T> { // Make a new `MyRc` to the same underlying data
   // TODO
}
```

and for all of this to work you'll need to correctly implement `Drop` for
`MyRc`, so that the underlying data is freed/not-freed at the correct times.

You can test your implementation using

```
cargo test --test required
```

For this assignment I ask that you do not look at the standard library's
implementation of `Rc`. You may however, look at the API for `Rc`, as well as
any code that uses it, in order to better understand the API. Note that we're
implementing a subset of its functionality.

## Part C: Breaking Rc

For the final part of the assignment, you must do two things:

   1. Describe a way to break `MyRc` by changing something inside an `unsafe`
      block or function.
   2. Describe a way to break `MyRc` by changing something _outside_ an `unsafe`
      block of function.

where a change breaks the type if it causes the type to violate the Abstraction
Safety Contract. As a review, such a violation would allow a user of the type to
get a memory error without using any unsafe code of their own.

For each breaking change, verify that the change would actually compile, explain
the issue, and outline a usage pattern (perhaps an example) which would produce
the memory error. Give these explanations in a markdown file.

## Bonuses

### Replace_With Unsoundness

So it turns out that it is **very hard** (impossible, according to anyone I've
talked to) to safely implement `replace_with`. Ultimately this is because Rust
insists that programs must be memory safe even after a panic[^panic].

For 2 bonus points, think of a safe Rust program which uses `replace_with` to
produce a memory error. Demonstrate that this program does indeed produce an
error by adding it to the test suite in the form of a functions. The relevant
test suite is in `src/tests/required.rs` in the `replace_with` crate.

For 1 bonus point, you may do this problem with the help of a hint -- look at an
example of such a program (located on the `unsound-hint` branch of the starter
code), and explain _why_ it produces a memory error.

### Weak<T>

One of the big problems with reference counting is that you can get cyclical
reference patterns which cause all the data involved to be retained until the
program ends, even if that data is no longer reachable.

This can be mitigated by programming with "weak reference counted pointer",
pointers which point to the data being reference counted, but which do not
prevent that data from being freed. Understandably, if weak pointers do not
cause the data to be retained, they can not guarantee access to it either.

They can be created by:

```rust
MyRc<T>::downgrade(&self) -> MyWeak<T>;
```

They can be used to produce a `MyRc` on demand (which can then be used to access
the data) using:

```rust
MyWeak<T>::upgrade(&self) -> Option<MyRc<T>>;
```

Modify your reference counting system so that it incorporates a weak pointer
named `MyWeak<T>` and includes the above functions.

A note: planning the system before implementing it is _very_ important. Explore
edge cases during the planning phase, to make sure your system works. I cannot
emphasize this advice enough: the step from `Box` to `Rc` is smaller than the
step from `Rc` to `Weak`.

You can test your implementation using

```
cargo test --test weak
```

### Reference Count Overflow

Most reference counters use keep their counts in `usize`.

The memory safety of reference counted containers depends very critically on the
reference counts they store. Any time safety depends on integer values, one
should consider the possibility and impact of overflow. Most reference counters
(including the standard library) keep their counts in a `usize`; since `usize`s
can store the size of the address space it seems like there is no potential for
the reference count to overflow.

In a markdown file explain

   1. Why overflow would be problematic and outline how it could lead to a
      memory error.
   2. Why even keeping reference counts in a `usize` doesn't _entirely_
      eliminate the possibility of overflow, and what conditions/actions would
      have to exist/happen to lead to the overflow in a program that uses only
      safe Rust.

As a hint: the problematic behavior is tied to the size of the address space. I
probably couldn't produce this bug on my machine (64 bit), but it probably
wouldn't be too hard on a 16 bit machine.

Another hint: Rust doesn't promise to run destructors.

[starter]: https://github.com/hmc-memsafe-2016f/wk4-starter.git
[survey]: https://docs.google.com/forms/d/e/1FAIpQLScuTNb3CK1gUH4ejN6bTadDm5XVqOVbHyrL19Cl1y1-kwiK1g/viewform
