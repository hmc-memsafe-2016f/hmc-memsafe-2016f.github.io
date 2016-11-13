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
   4. complete the implementations of `IntoIterator<...>` for the above types

The big constraint imposed on you for this assignment is that **you may not
elide any lifetimes**[^elision]. This is sort of a pain, but it's a good way of making
sure that you know what's going on. Regardless, most of the functions you'll be
implementing wouldn't allow elision anyway, or the elision mechanism would get
the lifetimes wrong.

[^elision]:
    technically, the restriction is stronger that. You must not only forsake
    *elided lifetimes*, but also *anonymous lifetimes* of any form. One possible
    presentation topic for this week details the history of anonymous lifetimes
    in Rust, and where elision fits into that.

The nature of this assignment means that most of your time will be spent just
getting the tests to compile, because they won't compile until the lifetime
parameters you're using have the correct semantics.

This week -- like the linked list week -- does not involve writing much code,
but does involve thinking carefully about some fairly tricky concepts
(lifetimes). You should anticipate little typing, and lots of pondering
lifetime bounds and compiler error messages.

[wk3-github]: https://github.com/hmc-memsafe-2016f/wk3-starter

