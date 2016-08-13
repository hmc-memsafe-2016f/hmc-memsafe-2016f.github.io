---

layout: assignment
title: "Week 2: Expressions"

---

This week it's time to venture out into the wild. You'll be using a Rust
library to implement an arithmetic expression engine. Likely, a lot of your
time will be spent on the parser, built using `nom`, a parser combinator
library. This will also be the first assignment involving non-trivial project
structure.

The starter code can be found [here][wk2-github]. It includes the 'parser' from
[nom's documentation][nom] which 'parses' an arithmetic expression into a
value (quite cleverly).

The project is organized as follows:

```
src/
  repl.rs        - A binary which runs a REPL
  expr/          - The expr module
    mod.rs       - The entry to the expr module: re-exports Expr and its parser
    expr.rs      - The Expr definition
    parser.rs    - The Expr parser
  reduced_expr/  - See the bonus
    rexpr.rs
```

Your job this week it to:

   1. Expand the 'parser' into an actual parser - one that produces an `Expr`
      rather that just its value.
   1. Implement `evaluate` for `Expr`, and fix the REPL to use it. This is
      integer arithmetic.
   1. Implement a few statistic-gathering functions for `Expr`: `depth` and
      `operation_count`.
   1. Write a new binary, named `stat`, which prints the value, depth, and
      operation count of an expression (separated by spaces).

## Bonus A: Optimizing Arithmetic Expressions.

This bonus has a few memory issues upfront, plus an open-ended optimization
problem!

The idea is to minimize the _depth_ of our arithmetic expressions. This is a
fun problem and also has some practical use: when building circuits the depth
of an expression roughly corresponds to the time required to compute it. The
starting parser (and likely the one you build) is strictly left-associative,
like division and subtraction are. However, addition and multiplication are
associative operations, and we'd like to take advantage of that.

This bonus proceeds in the following steps:

   1. Implement a conversion from `Expr` to `RExpr` (reduced expression), in
      which substraction is replaced with addition+negation. By doing so, you
      expose more associativity to your optimizations.
   2. Implement evaluation and the statistic functions for `RExpr` (see
      `src/reduced_expr/rexpr.rs`).
   3. Implement tree rotations for `RExpr`. These rotations should perform the
      rotation if it is structurally possible and would not change the value of
      the expression, and return whether or not the rotation was done. In
      essence, these rotations express the structural freedom given by
      associativity.
   4. Use rotations (or some other tools) to write `minimize_depth` for
      `RExpr`.  You may also take advantage of other properties of
      addition/multiplication (I.E. commutativity) if you want, but be sure to
      document this.
      * This part is entirely open-ended - have fun!

Use your optimization to write a binary named `reduce` which parses input
arithmetic expressions as `Expr`'s, prints their depth, converts them to
`RExpr`, prints the new depth, minimizes the depth, and prints the final depth.
All these depths should be space separated.

## Bonus B: Whitespace Insensitivity.

Change your parser so that it is no longer whitespace sensitive! There are a
few ways to go about this, so I'll leave it up to you, just include a
discussion of the methods you considered and their properties.

## Bonus C: Lifetime Preview.

This bonus is weakly motivated, but deals more directly with memory management
issues.

The goal for this bonus is to implment yet a _third_ version of `Expr`, lets
call it `SliceExpr`, which instead of storing the integer value of literals in
the expression will store the slices into the input - the slices where those
literals are found. You should also implement `evaluate` for this new data
structure as well.

While this approach is useful in some cases (`nom` uses techniques like this
under the hood), it's pretty silly here because integer literals are actually
smaller than slices :laughing:.

[wk2-github]: https://github.com/hmc-memsafe-2016f/wk2-starter
[nom]: http://rust.unhandledexpression.com/nom/
