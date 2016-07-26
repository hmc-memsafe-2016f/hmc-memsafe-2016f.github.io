layout: true
name: c
class: center middle

---

layout: true
name: l
class: left top

---

template: c

## Rust: Speed & Safety

.center[
.column-3.center[
![Bus][bus]
Safe
]
.column-3.center[
![Car][car]
Fast
]
.column-3.center[
![Plane][plane]
Both
]
]

.white[-]

Alex Ozdemir

---

template: l

## Systems Programming

.left-column.blue[
### Characteristics
* Low overhead
   * Cycles
   * Memory
* Direct control over computer
* Semantics mirror the hardware
]
.right-column.red[
### Uses
* Performance critical programs
   * Web infrastructure
   * Graphics & Games
* Hardware Interaction
   * Operating Systems
   * Drivers
* Extremely robust code
   * Airplanes
   * Space
]

---

template: l

## Systems Programming

.left-column.orange[
## Problems
* buffer overrun
* uninitialized memory
* use after free
* ...

.img-tiny[
![Heartbleed][heartbleed]
]
]
.right-column[
![Buffer Overruns][overruns]
.center[recorded buffer overrun vulnerabilities]
]

---

## Rust

![Trade Off][trade-off]

---

## Rust

.center[
.img-opaque[
![Trade Off][trade-off]
]

.img-large[
![No Trade Off][no-trade-off]
]
]

---

## Rust

_Rust features a powerful static type system which guarantees memory safety_

.left-column.blue.left[
### Advantage

Prohibits a large and important class of bugs.
]
.right-column.red.right[
### Disadvantage

Prohibits some complex,
but sound programs.
]

---

## Unsafe Rust

* Relaxes Rust's type system
* Allows programmers to write libraries with
    * internally complex memory semantics
    * externally safe interfaces

.center.img-large[
![Graph][graph]
]

---

class: center, middle

.big-idea[
Unsafe is all about encapsulating .orange[dangerous operations] within
.blue[safe interfaces]
]

---

template: c

## Questions

.big-ideas[

How do programmers write unsafe code?

Why do programmers write unsafe code?

How should programmers write unsafe code?

]

---

## Phase 1: Syntactic Analysis
.center[
![Flowchart for Syntax][syntax]

.footnote[A careful discussion of results and methods can be found [here][blog].]
]

---

## Phase 2: Dataflow

.left-column[
* We model unsafe code as requiring _assumptions_ in order to safely execute.

* These assumptions are statements about values that flow through the program.

* We attempt to verify these assumptions, validating the unsafe code that
requires them.
]
.right-column.small-code[
```rust
fn main() {
    let x = 5;
    let p = &5 as *const i32;


    unsafe {

        *p
    }
}
```
]

---

## Phase 2: Dataflow

.left-column[
* We model unsafe code as requiring _assumptions_ in order to safely execute.

* These assumptions are statements about values that flow through the program.

* We attempt to verify these assumptions, validating the unsafe code that
requires them.
]
.right-column.small-code[
```rust
fn main() {
    let x = 5;
    let p = &5 as *const i32;


    unsafe {
        // "p must be a valid pointer"
        *p
    }
}
```
]

---

## Phase 2: Dataflow

.left-column[
* We model unsafe code as requiring _assumptions_ in order to safely execute.

* These assumptions are statements about values that flow through the program.

* We attempt to verify these assumptions, validating the unsafe code that
requires them.
]
.right-column.small-code[
```rust
fn main() {
    let x = 5;
    let p = &5 as *const i32;
    //      ^^ This operation ensures p
    //         is a valid pointer!
    unsafe {
        // "p must be a valid pointer"
        *p
    }
}
```
]

[plane]: http://orig13.deviantart.net/d271/f/2012/203/6/f/airplane_by_paullus23-d58794k.jpg
[car]: http://resources.carsguide.com.au/styles/cg_hero_large/s3/Mitsubishi-Mirage-sedan-(4).jpg
[bus]: http://farm5.staticflickr.com/4002/4551273440_b8253f2383_z.jpg
[heartbleed]: https://upload.wikimedia.org/wikipedia/commons/d/dc/Heartbleed.svg
[overruns]: /images/2016-07-26-buffer-overruns.png
[trade-off]: https://docs.google.com/drawings/d/1DUug8LxqRalnfckaHwUTd8tyxWMxDn2Aubqr6v8-DyY/pub?w=894&h=134
[no-trade-off]: https://i.imgur.com/nNSYO84.png
[graph]: https://cdn-images-1.medium.com/max/400/1*Q9n58avTamrRmY66Ne0Hug.png
[blog]: https://alex-ozdemir.github.io/rust/unsafe/unsafe-in-rust-syntactic-patterns/
[syntax]: https://docs.google.com/drawings/d/1LIeBdnuG-N0ilCzbl_Dl38Auku6xJb0CqNZQyzQ_NGc/pub?w=1210&h=367
