---

layout: assignment
title: "Week 0: Warm-Up"

---

## Overview

This week you'll be implementing a simple command line application in Rust, as
a way of getting used to the language. Our application will be Tower's of Hanoi
game.

During this assignment you should gain some familiarity with:

   * Rust syntax for
      * functions and methods
      * product types (`struct`s)
      * sum types (`enum`s)
   * The Rust type system
   * Common parts of Rust's standard library, `std`:
      * `Vec`: `new`, `push`, `pop`, `last`
      * `Option`: `map`, `ok`, `and_then`, `unwrap`, `unwrap_or`, etc.
      * `Result`: `map`, `ok`, `and_then`, `unwrap`, `unwrap_or`, etc.
      * `try!`
   * The style of returning errors rather than raising exceptions

In order to make sure the above topics come up during the assignment, the
design of the application is set by the [starter code][starter]. The application has a
main loop which

   1. Prints the board
   1. Reads user input
   1. Parses that input into an `Action`
   1. Processes that action into a `NextStep`
      * eg. by executing it if the action is a `Move`
   1. Resolves the `NextStep`.

This flow is made more complex by that fact that many of the steps may produce
an error instead of succeeding - causing a `HanoiError` to be returned.

Your job is to implement the parsing (although the input language is simple
enough that this can just be done with a `match`) and `Action` processing.

Places where there is implementation to be done are denoted with the
`unimplemented!` macro, which allows the type system to check out, but panics
during execution. As you work you can build the project with `cargo build` and
run the application using `cargo run [number of disks]`.

## Bonus

The bonus for this assignment is fairly open-ended: just extend the application
in some way! Possibilities include:

   * Add an AI
   * Improve the way a board is displayed
   * Modify the original Towers of Hanoi game

## Example Behavior

This assignment will not be auto-graded, so there is no need to satisfy any
specific interface. However, an example of one game is below:

```
  Left: 3 2 1
Center: 
 Right: 
$ lc 
  Left: 3 2
Center: 1
 Right: 
$ lc
Error: A disk of size 2 can't go on peg Center: there's already a smaller disk
  Left: 3 2
Center: 1
 Right: 
$ lr
  Left: 3
Center: 1
 Right: 2
$ cr
  Left: 3
Center: 
 Right: 2 1
$ l
Error: Unknown Command
  Left: 3
Center: 
 Right: 2 1
$ lc
  Left: 
Center: 3
 Right: 2 1
$ rl
  Left: 1
Center: 3
 Right: 2
$ rc
  Left: 1
Center: 3 2
 Right: 
$ lc
  Left: 
Center: 3 2 1
 Right: 
You Won!
```

[starter]: https://github.com/hmc-memsafe-2016f/wk0-starter
