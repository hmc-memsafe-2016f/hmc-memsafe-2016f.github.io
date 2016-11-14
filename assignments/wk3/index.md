---

layout: assignment
title: "Week 3: Lifetimes"

---

This week is when we will deal deeply with some intense areas of Rust's memory
model - you'll be dealing directly with
the idea of a _lifetime_, the concept which underlies Rust's borrow system. In
fact, you've actually been dealing with lifetimes all along but you've been
using them in situations which were simple enough that you were able to _elide_
them.

We'll be working on a mini-database: a list of records which allows you to
select a subset of them based on a predicate. We'll be using a very naive
strategy: go through each record and test it based on the predicate. Real
databases, of course, do fun things with indexing.

This week's task is actually fairly simple and could be done using iterators
from the standard library alone, but for the enhanced educational experience
we'll be doing things our own way (in a style that mirrors the way that
collections within the standard library are implemented).

In particular, our design uses the following types:

   * `DB`: A structure which owns a vector of records.
   * `DBView`: A structure which represents a set of records borrowed from
     somewhere else.
   * `DBViewMut`: A structure which represents a set of mutable records borrowed
     from somewhere else.

(This is similar to `std::vec::Vec`, `std::vec::Iter`, and `std::vec::IterMut`.)

You will be completing the following tasks:

   1. complete the definitions of the above types (by adding the necessary
      lifetime parameters)
   2. complete the definitions of various methods on the above types, and
      implement them
   3. complete the definitions of `filter_one` and `filter_two`, and implement
      them

Doing the above will likely require you use *named lifetime parameters*. Doing
this often requires careful thought, especially because the compiler doesn't
always provide the best error messages if you mess up your lifetime parameters.

The nature of this assignment means that most of your time will be spent just
getting the tests to compile, because they won't compile until the lifetime
parameters you're using have the correct semantics. To compile and run the
required tests, do

```
cargo test --test required
```

This week -- like the linked list week -- does not involve writing much code,
but does involve thinking carefully about some fairly tricky concepts
(lifetimes). You should anticipate little typing, and lots of pondering
lifetime bounds and compiler error messages. Revisiting the final example from
class may be useful.

The starter code is [here][wk3-github]. Once you're done, fill out
[this][exit-form] exit questionnaire.

# Bonuses

## Bonus A: Iterator Interfaces

Implement `IntoIterator<...>` for `DB`, `DBView`, and `DBMutView`.

   * A `DB` can produce an iterator over `T`, `&mut T`, or `&T`.
   * A `DBViewMut` can produce an iterator over `&mut T`.
   * A `DBView` can produce an iterator over `&T`.

You'll find incomplete implementations of `IntoIterator` for these type-target
pairs in the starter code.

You'll find a specification of the `IntoIterator` trait [here][trait-into-iter], and an example of
how `IntoIterator` can be implemented [here][vec-iter].

In completing these implementations, it is _strongly_ suggested to lean on the
standard library, rather that writing your own `Iterator` for each of
`IntoIterator` implementation. Even within the standard library, implementations
of `IntoIterator` use pre-existing iterators. For example, the `IntoIterator`
implementation which allows `Vec<T>` to produce an iterator over `&T` shares an
iterator with the implementation which allows `[T]` to produce an iterator over
`&T` (see the `IntoIterator` implementation for `Vec<T>` [here][vec-iter] and
that its `IntoIter` associated type is `slice::Iter`).

To run the tests for this bonus, do:

```
cargo test --test bonus_iterator
```

## Bonus B: Explaining Lifetimes

_Explain to your non-technical friend ..._

Just kidding -- you don't need to explain lifetimes to a non-technical friend.
However, this problem will be all about explaining why certain lifetime
bounds cause the compiler to reject code. Through this process you'll explore a
case where something called _higher-ranked lifetimes_ are useful.

Unfortunately, the compiler doesn't usually produce good error messages if you
give it problematic explicit lifetimes. Using the nightly compiler provides
_slightly_ better error messages, so you may want to use it for this problem.

Write the answers to the following questions in a markdown file in your repo.
Explain using your own words (not just the compilers!). It may be helpful to
include snippets of the code with scopes labelled.

   1. Take a look at `puzzle` in [this program][bonus-b-1], and how it's used.
      Compile it. What is the difference between how `a_outlives_b` and
      `b_outlives_a` use `puzzle`? (Hint: their names are suggestive).
   2. For the rest of this problem we will be modifying part of `puzzle`'s
      signature. Specifically we'll be trying to figure out exactly what
      lifetimes are involved with the trait bound `F: Fn(&i32) -> Option<&i32>`.
      Our [first take][bonus-b-a] is to say both unnamed lifetimes in the bound
      should be `'a`. Explain why this fails.
   3. Hmm... We could try making both those lifetimes `'b`, but by symmetry that
      shouldn't work either. Instead we introduce a new lifetime parameter `'c`,
      and use it ([code][bonus-b-c]). Explain why this also doesn't work.
   4. Observing that `Option<&'c i32>` couldn't be returned as an `Option<&'a
      i32>` (or a `Option<&'b i32>`), we conclude that it is because `'c` may
      not be as long as long as `'a` or `'b`. To fix this we add the extra
      condition that `'c` must outlive both `'a` and `'b` (written as `'c: 'a +
      'b`). Explain why the [result][bonus-b-c-bounded] also doesn't work.
   5. Frustrated, we add even more constraints! Now we add that `'a` and `'b`
      must each outlive `'c`. Explain why the [result][bonus-b-c-eq] also
      doesn't work.
   6. However, if we change our "test" functions (`a_outlives_b` and
      `b_outlives_a`), [it works][bonus-b-c-eq-bad-test]! Explain why this
      happens.
   7. Finally, we strike upon the correct idea. This code compiles and works
      just fine. What do you think the `for<'c>` means? Feel free to take a
      guess or read [this][hrtb] and then answer.

## Bonus C: Parsing Round II

This bonus is a repeat of Bonus C from last week:

The goal for this bonus is to implment yet a _third_ version of `Expr`, lets
call it `SliceExpr`, which instead of storing the integer value of literals in
the expression will store the slices into the input - the slices where those
literals are found. You should also implement `evaluate` for this new data
structure as well.

While this approach is useful in some cases (`nom` uses techniques like this
under the hood), it's pretty silly here because integer literals are actually
smaller than slices :laughing:.

It is, however, a great demonstration of your knowledge of lifetimes!


[wk3-github]: https://github.com/hmc-memsafe-2016f/wk3-starter
[vec-iter]: https://doc.rust-lang.org/src/collections/up/src/libcollections/vec.rs.html#1468
[trait-into-iter]: https://doc.rust-lang.org/std/iter/trait.IntoIterator.html
[bonus-b-1]: https://is.gd/IJx0Cr
[bonus-b-a]: https://is.gd/hl3qEl
[bonus-b-c]: https://is.gd/BxNjhS
[bonus-b-c-bounded]: https://is.gd/uNucer
[bonus-b-c-eq]: https://is.gd/eSzMj4
[bonus-b-c-eq-bad-test]: https://is.gd/8J1EPr
[bonus-b-done]: https://is.gd/xCLKc5
[hrtb]: https://doc.rust-lang.org/nomicon/hrtb.html
[exit-form]: https://docs.google.com/forms/d/e/1FAIpQLSfUWFMPPcQvf8MCA3pUG537nL0UGnzdme1bagT-yZUssdIXtQ/viewform
