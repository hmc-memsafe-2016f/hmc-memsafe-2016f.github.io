layout: true
class: center middle

---
## Rust: Through the Escape Hatch

Alex Ozdemir

---

## What is Rust?

> Rust is a systems programming language that runs blazingly fast, prevents
> segfaults, and guarantees thread safety.
>
> -- [www.rust-lang.org](//www.rust-lang.org)

---

## What is Rust?

> Rust is a **systems** programming language that runs blazingly fast, 
> **prevents segfaults**, and guarantees thread safety.
>
> -- [www.rust-lang.org](//www.rust-lang.org)

---

## What is Rust?

Rust is a **systems** programming langugage which guarantees **memory safety**.

---
## Systems Programming is Hard

buffer overflow

use after free

reading uninitialized memory

null pointer dereference

---

## The Trade-off

![Safety or Control?][trade-off]

---

## No Trade-off

![Both Safety and Control][no-trade-off]

---

layout: false
## The Rust Ownership System

All memory has a unique owner.

In practice:

```rust
fn main() {
   let x = vec![5];
   println!("The number is {}", x[0]);
   // The address x points to is free'd
}
```

---

## The Rust Ownership System

Assignment transfers ownership (move semantics).

```rust
fn main() {
   let x = vec![5];
   let y = x;
   println!("The number is {}", y[0]);
   println!("The number is {}", x[0]); // ERROR
}
```

---

## The Rust Ownership System

Assignment transfers ownership (move semantics).

```rust
fn main() {
   let x = vec![5];
   let y = x;
   println!("The number is {}", y[0]);
   // The address y points to is free'd
   // No memory is free'd because of x
}
```

---

## The Rust Ownership System

Values may be _borrowed_.

```rust
fn main() {
   let x = vec![5];
   let y = &x;
   println!("The number is {}", y[0]);
   // No memory is free'd because of y
   // The address x points to is free'd
}
```

---

## The Rust Ownership System

Borrows temporarily prohibit mutation.

```rust
fn main() {
   let mut x = vec![5];
   let y = &x;
   x.push(6); // ERROR
   println!("The number is {}", y[0]);
}
```

---

## The Rust Ownership System

Is this okay?

```rust
fn main() {
   let mut x = vec![5];
   {
       let y = &x;
       println!("The number is {}", y[0]);
   }
   x.push(6);
}
```

---

## The Rust Ownership System

### Recap

* All data has a unique owner (including data on the heap!).
* Data may be _borrowed_.
* Borrows temporarily prohibit mutation.

---

layout: true
class: center middle

---

## Lists

![A singly linked List][single-list]

---

## Lists

![A singly linked List][single-list]


![A doubly linked List][double-list]

---

## Enter "Unsafe", Stage Left

---


## Rust: Through the Escape Hatch

Alex Ozdemir

![Through the Looking Glass][looking-glass]

---

### Unsafe allows for

complex data structures

shared mutable state

foreign function interface

---

layout: false

## How Unsafe Works

1. Declare the **block** or **function** unsafe.
2. Do unsafe things
   * Dereference raw pointers
   * Call unsafe functions
   * ...

---

## Unsafe Example

A good use of `unsafe`:

```rust
fn index(v: &[i32], i: usize) -> Option<i32> {
    if i < v.len() {
        unsafe { Some(v.get_unchecked(i)) }
    } else {
        None
    }
}
```

--

A bad use of `unsafe`:

```rust
fn dereference(p: *const i32) -> i32 {
    unsafe { *p }
}
```

---

## Unsafe Example

A good use of `unsafe`:

```rust
fn index(v: &[i32], i: usize) -> Option<i32> {
    if i < v.len() {
        unsafe { Some(v.get_unchecked(i)) }
    } else {
        None
    }
}
```

A good use of `unsafe`:

```rust
unsafe fn dereference(p: *const i32) -> i32 {
    *p
}
```

---

## The Unsafe Dichotomy

The compiler understand `unsafe` in terms of _operations_.

Programmers should use `unsafe` to build safe _abstractions_.

---

## The Goals of My Project

1. Explore what it means to build safe abstractions around `unsafe`.
2. Describe how real-world Rust code uses `unsafe`.
3. Prescribe some ways `unsafe` should and should not be used
   * and built tools to detect mis-use

---

## A Two Pronged Approach

.left-column.red.left[
### Syntactic

How much `unsafe` is there?

How is it organized?

Why is it used?

]
.right-column.blue.right[
### Semantic

Is `unsafe` safely encapsulated?

How do assumptions flow through programs?

Are aliasing rules respected?

]

---
class:center, middle

.red[
## Syntactic Analysis
]

---

### How and Why is Unsafe Code Used?
* How much Rust code is unsafe?
   * In terms of libraries?
   * In terms of blocks?
   * In terms of functions?
* How frequent are different kind of unsafe operations?
   * Raw pointer dereferences?
   * Calling unsafe functions?
      * Foreign Function Interface (FFI)?
   * Using mutable statics?
   * Using inline assembly?
* How is unsafe organized?
   * Does it appear a few places in a code-base, or everwhere?
   * Does it occur inside closures?
   * Are unsafe blocks wrapped tightly or loosely around unsafe operations?

---

## The Unsafe AST

.left-column.small-code[
```rust
fn say_hi() {
    println!("Hello!");
}
```
]
.right-column.small-code[
```rust
fn index(v: &[i32], i: usize) -> Option<i32> {
    if i < v.len() {
        unsafe { Some(v.get_unchecked(i)) }
    } else {
        None
    }
}
```
]

---

## The Unsafe AST

.left-column.small-code[
```rust
fn say_hi() {
    println!("Hello!");
}
```
]
.right-column.small-code[
```rust
fn index(v: &[i32], i: usize) -> Option<i32> {
    if i < v.len() {
        unsafe { Some(v.get_unchecked(i)) }
    } else {
        None
    }
}
```
]
![The Unsafe AST layout][uast]

---

## The Pipeline

![Where do we analyze?][flow-chart]

---

## Syntactic Analysis Results

.center.middle[
![Number of Unsafe Contexts By Crate][unsafe-contexts-by-crate]
]

---

## Syntactic Analysis Results

.center.middle[
| Context   Type |  Total |    Safe | Unsafe | % Unsafe |
|:-------------- |  -----:|    ----:| ------:| --------:|
| Function       | 269,070 | 258,088  |  10,982 |      4.1 |
| Block          | 557,118 | 521,547  |  35,571 |      6.4 |
]

---

class: center middle

You can see more at
[alex-ozdemir.github.io/rust/unsafe/unsafe-in-rust-syntactic-patterns/](https://alex-ozdemir.github.io/rust/unsafe/unsafe-in-rust-syntactic-patterns/)

---

class: center middle blue

## Semantic Analysis

---

## Bad Uses of Unsafe

A bad use of `unsafe`:

```rust
fn dereference(p: *const i32) -> i32 {
    unsafe { *p }
}
```

An okay use of `unsafe`:
```rust
fn identity(i: i32) -> i32 {
    let p = &i as *const i32;
    unsafe { *p }
}
```

---

## Pointer Provenance

We're looking at a data-flow question

```rust
// Problem!
fn dereference(p: *const i32) -> i32 {
    // p must be deref-able
    let q = p;
    // q must be deref-able
    unsafe { *q }
}
```

```rust
fn identity(i: i32) -> i32 {
    let p = &i as *const i32;
    // ^ Nice! This guarantees p is deref-able!
    // p must be deref-able
    unsafe { *p }
}
```


---

### Data Structures

* Take in data
* Store that data in interesting ways
* Access that data in interesting ways

---

## Bad Uses of Unsafe

An okay use of `unsafe`:

```rust
pub struct MyBox {
    data: Box<i32>,
    p: *const i32,
}

impl MyBox {
    pub fn new(i: i32) -> MyBox {
        // Puts i in `data`'s Box
        // Points `p` at the contents of the box
    }
    pub fn get(&self) -> i32 {
        unsafe { *(self.p) }
    }
}
```

---

## Bad Uses of Unsafe

An **bad** use of `unsafe`:

```rust
pub struct MyBox {
    data: Box<i32>,
*   pub p: *const i32,
}

impl MyBox {
    pub fn new(i: i32) -> MyBox {
        // Puts i in `data`'s Box
        // Points `p` at the contents of the box
    }
    pub fn get(&self) -> i32 {
        unsafe { *(self.p) }
    }
}
```

---

## Provenance Analysis

* Interprocedural

* Access-Path aware (The deref-abilities of `x.ptr1` and `x.ptr2` are independent)

* Visibility-aware

* Alias-aware

---

class: center, middle

### Thanks!

---

## More Syntactic Analysis Results

.center.middle[
![Style of Unsafe][unsafe-style]
]

---

## More Syntactic Analysis Results

| Source |Deref ptr  | Call unsafe Rust function  | Call FFI  | Use `static mut`  | Use inline ASM  | All uses |
|---|---:|---:|---:|---:|---:|---:|
|  `derive` macro  |0 |12,058 |0 |0 |0 |12,058 |
|  External macro  |3,732 |9,843 |161 |8,841 |80 |22,657 |
|  Local macro  |801 |8,176 |2,087 |57 |0 |11,121 |
|  Not a macro  |4,496 |18,916 |13,061 |1,264 |0 |37,737 |
|  All sources  |9,029 |48,993 |15,309 |10,162 |80 |83,573 |

[trade-off]: https://docs.google.com/drawings/d/1DUug8LxqRalnfckaHwUTd8tyxWMxDn2Aubqr6v8-DyY/pub?w=894&h=134
[no-trade-off]: https://i.imgur.com/nNSYO84.png
[uast]: https://docs.google.com/drawings/d/1hSw97-1_yJuFz4lLRCKhHcZz5BsQa-JTG2NtZwHoIgw/pub?w=731&h=284
[flow-chart]: https://docs.google.com/drawings/d/1Utsbm6fwCy96jAXJjJZZntnEkYS7gi7IhePAxhqVZcs/pub?w=605&h=227
[single-list]: https://upload.wikimedia.org/wikipedia/commons/6/6d/Singly-linked-list.svg
[double-list]: https://upload.wikimedia.org/wikipedia/commons/5/5e/Doubly-linked-list.svg
[unsafe-contexts-by-crate]: https://alex-ozdemir.github.io/images/2016-07-07-unsafe-declarations-by-crate.png
[unsafe-style]: https://alex-ozdemir.github.io/images/2016-07-11-unsafe-blocks-rel-size-and-requirements.png
[looking-glass]: http://nerdreactor.com/wp-content/uploads/2016/02/alice-through-the-looking-glass-poster-1.jpg
