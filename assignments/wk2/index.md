---

layout: assignment
title: "Week 2: Expressions"

---

This week it's time to venture out into the wild. You'll be using a Rust
library to implement an arithmetic expression engine. Likely, the majority of
your time will be spent on the parser, built using `non`, a parser combinator
library.

The starter code can be found [here][wk2-github]. It includes the 'parser' from
[nom's documentation][nom] which 'parses' an arithmetic expression into a
value (quite cleverly).

Your job this week it to expand the 'parser' into an actual parser - one that
produces an `Expr`. You'll also implement the following functions for `Expr`:

```rust
enum Expr {
    BinOp(Box<Expr>, BinOp, Box<Expr>),
    Literal(isize),
}

enum BinOp {
    Plus,
    Minus,
    Times,
    Over,
}

impl Expr {
    fn evaluate(&self) -> isize;
    fn operation_count(&self) -> usize;
    fn depth(&self) -> usize;
}
```

## Bonus

The bonus for this week should be completed after doing and submitting the
normal assignment. It is to change `Expr` so that it stores slices of the
original `&[u8]` instead of actual numeric literals. Doing this will be
excellent preparation for next week as you'll be dealing with explicit
lifetimes. If you find yourself going down this path, don't hesitate to come
talk to me!

[wk2-github]: https://github.com/hmc-memsafe-2016f/wk2-starter
[nom]: http://rust.unhandledexpression.com/nom/
