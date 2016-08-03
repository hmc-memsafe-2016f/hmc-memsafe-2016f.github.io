layout: true
name: c
class: center middle

---

layout: true
name: l
class: left top

---

template: c

## Rust: Down the Rabbit Hole

.white[-]

#### Speed, Safety, and the Loophole that Makes Both Possible

Alex Ozdemir

---

template: l

## Low-Level Languages

.left-column.blue[
### Benefits
* Low overhead
   * (Good Performance)
   * Direct control over computer

]
.right-column.red[
### Problems
* Not _memory safe_
   * Potential for "memory errors"
      * Buffer overflow, use-after-free, etc.
   * Prominant source of bugs
      * i.e. Heartbleed

.center.img-tiny[
![Heartbleed][heartbleed]
]
]

???

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

---

## Rust

.center[
![Trade Off][trade-off]
]

---

## Rust

.center[
![No Trade Off][no-trade-off]
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
but safe programs.
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
]

---

## Phase 1: Syntactic Analysis

.center[
.img-large[
![Unsafe in Rust][contexts]
]

.footnote[A careful discussion of results and methods can be found [here][blog].]

]

---

## Phase 2: Data-Flow Analysis

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
    let p = &x as *const i32;


    unsafe {

        *p
    };
}
```
]

---

count: false

## Phase 2: Data-Flow Analysis

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
    let p = &x as *const i32;


    unsafe {
        // "p must be a valid pointer"
        *p
    };
}
```
]

---

count: false

## Phase 2: Data-Flow Analysis

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
    let p = &x as *const i32;
    //      ^^ This operation ensures p
    //         is a valid pointer!
    unsafe {
        // "p must be a valid pointer"
        *p
    };
}
```
]

---

## Summary

Rust provides powerful guarantees, but includes a loophole, `unsafe`.

.left-column[
### Syntactic Analysis
* How common is unsafe code?
* What syntactic patterns does it fall into?
]
.right-column[
### Data-Flow Analysis
* Can we detect uses of the `unsafe` feature that are _definitely safe_ or
    _definitely unsafe_?
   * _Super_ cool!
   * Work in Progress
]

---

template: c

![Bye!][thats-all]

[thats-all]: http://img09.deviantart.net/0248/i/2013/295/d/8/that_s_all_folks__by_surrimugge-d6rfav1.png
[heartbleed]: https://upload.wikimedia.org/wikipedia/commons/d/dc/Heartbleed.svg
[overruns]: /images/2016-07-26-buffer-overruns.png
[contexts]: /images/2016-08-02-unsafe-prevelence.png
[trade-off]: https://docs.google.com/drawings/d/1hDhy3MN6q4P2f755t4BkllHUXlAUXdc54XCgoBEiNdU/pub?w=713&h=431
[no-trade-off]: https://docs.google.com/drawings/d/1nsZ0IU8gCc_J0sgWwjVvo_ZdzxrtlAj6vXFATeJjKMs/pub?w=713&h=431
[graph]: https://cdn-images-1.medium.com/max/400/1*Q9n58avTamrRmY66Ne0Hug.png
[blog]: https://alex-ozdemir.github.io/rust/unsafe/unsafe-in-rust-syntactic-patterns/
[syntax]: https://docs.google.com/drawings/d/1LIeBdnuG-N0ilCzbl_Dl38Auku6xJb0CqNZQyzQ_NGc/pub?w=1210&h=367
[looking-glass]: http://nerdreactor.com/wp-content/uploads/2016/02/alice-through-the-looking-glass-poster-1.jpg
