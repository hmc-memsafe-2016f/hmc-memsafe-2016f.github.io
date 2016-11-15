layout: true
class: center middle

---

## Memory Safety

### Unsafe

#### Back to the Wild West

15 November 2016

---

layout: false
class: left

## Outline of today

   1. Motivation
   1. Meet Unsafe Rust
      * `unsafe` contexts
      * `unsafe` operations
   1. Examples of `unsafe`
      * `get`, using `get_unchecked`
      * `get`, using a raw pointer
      * `OurVec`
   1. Tour of `unsafe` functions
   1. Using `unsafe` properly
      * The Abstraction Safety Contract
   1. Assignment breifing

---

class: middle center

## Motivation for Unsafe

---

## Cyclic Data Strutures

![A Double Linked List][double-list]

---

## Interior Mutability

```rust
impl RefCell<T> {
    /// Mutably borrows the wrapped value.
    ///
    /// The borrow lasts until the returned RefMut exits scope.
    /// The value cannot be borrowed while this borrow is active.
    ///
    /// Panics if the value is currently borrowed.
    fn borrow_mut(&self) -> RefMut<T>;
}
```

---

## Uninitialized Memory

![A vector][vector]

---

## Data Structures

Basically all of `std::collections`.

---

## Shared Mutable Data

```rust
static mut B_DROPS: u32 = 0;

struct B<T>{ b: Box<T> }

impl<T> Drop for B<T> {
    fn drop(&mut self) {
        B_DROPS += 1;         // ERROR
    }
}
```

---

## Things Safe Rust doesn't allow

   * Cyclic data structures
   * Interior mutability
   * Unintialized values
   * Unchecked array access
   * Shared mutable data

--

Unsafe will allow us to express these ideas.

---

class: middle center

## How Unsafe Works

### The Rules

---

## How Unsafe Works

The language feature `unsafe` operates as following:

   * One can declare contexts as `unsafe`:
      * Blocks can be unsafe
      * Functions can be unsafe
--
   * Within those contexts one can perform unsafe operations:
      1. Dereference raw pointers
--
      1. Use mutable static variables
--
      1. Call unsafe functions
--
         * including compiler intrinsics
--
   * One can also implement unsafe traits
      * Traits that must be implemented correctly to maintain memory safety, but
        the compiler cannot check.

--

We're going to focus on the first two: the interaction between unsafe contexts
and unsafe operations.

???

Notice that `unsafe` does not turn off the borrow checker or anything like that.
It just gives you access to things that will allow you to circumvent various
parts of the Rust type system (including the borrow checker).

---

## Unsafe Operations: Dereferencing Raw Pointers

```rust
fn main() {
    let x = 5;
    let r = &x;
    let p = r as *const _;
    unsafe {
        println!("{}",*p);
    }

    let p2 = 0 as *mut _;
    println!("{}", unsafe { *p });
}
```

---

## Unsafe Operations: Calling Unsafe Functions

```rust
use std::mem;
fn main() {
    let y: i32 = -1;
    let x: u32 = unsafe {
        mem::transmute(y)
    };
    println!("{} {:b}", x, x);
}
```

---

## Unsafe Operations: Mutable Statics

```rust
static mut B_DROPS: u32 = 0;

struct B<T>{ b: Box<T> }

impl<T> Drop for B<T> {
    fn drop(&mut self) {
        unsafe {
            B_DROPS += 1;
        }
    }
}
```

---

## Unsafe Contexts: Functions

We've seen plenty of unsafe blocks. You can also write unsafe functions. They
look like this:

```rust
pub unsafe fn read<T>(ptr: *const T) -> &T {
   &*ptr
}
```

What's wrong with example? How do you fix it?

--

The output has an anomynous lifetime which cannot be determined using any of the
elision rules. We should fix it by adding a lifetime parameter.

---

class: center middle

## Examples of How Unsafe is Used

---

## Get (Safe)

In Rust if you index a slice (`[T]`) out of bounds then the program panics.
Suppose that you wanted to implement a version of index which returned an
`Option<T>` (`slice::get`).

You could write:

```rust
fn get<'a, T>(slice: &'a [T], index: usize) -> Option<&'a T> {
    if index < slice.len() {
        Some(&slice[index])
    } else {
        None
    }
}
```

but if you do this then the bounds check actually happens twice. Once
explicitly, and the other time in `slice[index]`.

--

We can eliminate this second check using `slice::get_unchecked`.

???

Fair warning: this optimization is likely unnecessary -- it seems like something
that LLVM could handle.

---

## Get (with get_unchecked)

```rust
fn get<'a, T>(slice: &'a [T], index: usize) -> Option<&'a T> {
    if index < slice.len() {
        unsafe { Some(slice.get_unchecked(index)) }
    } else {
        None
    }
}
```

--

This works, but there's also another way -- we could get a point to the slice,
advance it, and derefence.

---

## Get (with a pointer)

```rust
fn get<'a, T>(slice: &'a [T], index: usize) -> Option<&'a T> {
    if index < slice.len() {
        let ptr = slice.as_ptr();
        unsafe { Some(&*ptr.offset(index as isize)) }
    } else {
        None
    }
}
```

---

## OurBox<T>

What if we wanted to implement our own box?

What are the important operations we'd need?

--
   * `new`
   * `*`
   * `drop`

Let's start with the layout. What should `OurBox<T>` contain?

--

```rust
struct OurBox<T> {
    ptr: *mut T,
}
```

---

## OurBox<T>: Constructor

What should our constructor look like?

It needs to do only one thing: allocate memory.

--

```rust
impl<T> OurBox<T> {
    fn new(t: T) -> Self {
        OurBox {
            ptr: Box::into_raw(Box::new(t)),
        }
    }
}
```

The way we'll allocate memory in this class is what is shown here: creating a
box and then turning the box into a pointer.

`Box::into_raw` not only gives us a pointer to the underlying data, it also
destroys the box without running `T`'s destructor or releasing the memory.

--

On the `*mut` from earlier. We eschew `*const` for `*mut` because that's what
`Box::into_raw` gives us. Why does it give us `*mut`?

???

`Box::into_raw` gives a `*mut` because if you owned the original box, you should
probably be able to mutate the referent of the pointer you received. Disclaimer:
`*mut` and `*const` can be casted to one another, so this distinction is pretty
meaningless.

---

## OurBox<T>: Destructor

What does the destructor need to do?

--

It only needs to release the memory and run `T`'s destructor.

```rust
impl<T> Drop for OurBox<T> {
    fn drop(&mut self) {
        unsafe {
            Box::from_raw(self.ptr);
        }
    }
}
```

This is how we'll release memory in this class: by turning a pointer to that
memory into a box. Then, when the box goes out of scope the memory is free'd
(and `T`'s destructor is run).

---

## OurBox<T>: Dereference

To make our box useful, we should also make sure we can dereference it to get at
the underlying data. We do that by implementing `ops::Deref`, which described
how to immutably deref.

--

```rust
use std::ops;

impl<T> ops::Deref for OurBox<T> {
    type Target = T;
    fn deref(&self) -> &T {
        unsafe { &*self.ptr }
    }
}

```

We would similarly implement `ops::DerefMut`.

---

## Recap

Some particularly important points for the assignment

   * You can allocate using `Box::into_raw(Box::new(...))`
   * You can release (and destruct) using `Box::from_raw(...);`

---

class: middle center

## Tour of Some Unsafe Functions

---

## Common Unsafe Functions

The standard library contains a number of primitive unsafe functions, useful for
writing unsafe code.

   1. `mem::uninitialized`
   1. `mem::forget` (actually safe)
   1. `mem::transmute`
   1. `mem::zeroed`
   1. `ptr::read`
   1. `ptr::write`
   1. `ptr::drop_in_place`
   1. `ptr::copy`
   1. `pointer::offset`
   1. `pointer::as_ref`

With a friend, choose two, go figure out what they do.

---

   1. `mem::uninitialized` : produces an uninitialized value.
   1. `mem::forget` : consume without running destructor
   1. `mem::transmute` : change one type to another type of the same size
   1. `mem::zeroed` : produce a value with all 0 bits
   1. `ptr::read` : copy the referent to produce a new instance of the type
   1. `ptr::write` : move into a pointer's referent without destructing the
      original referent
   1. `ptr::drop_in_place` : run the destructor of the referent
   1. `ptr::copy` : like `memcopy`
   1. `pointer::offset` : advance a pointer some integral number of TYPESIZEs
   1. `pointer::as_ref` : turn a pointer into a reference of **any** lifetime

---

class: center middle

## Using Unsafe _Safely_

### The Abstraction Safety Contract

---

## Why Use Unsafe Rust?

![Space of Programs][pgm-space]

---

## Why Use Unsafe Rust?

![Space of Programs & Examples][pgm-space-ex]

---

## Why Use Unsafe Rust?

![Safe Rust][safe]

---

## Why Use Unsafe Rust?

![Unsafe Full][unsafe-bad]

---

## Why Use Unsafe Rust?

![Unsafe Well USed][unsafe-good]

---

## The Abstraction Safety Contract

To facilitate using `unsafe` to write safe programs, we adhere to the following
contract.

   1. Unsafe operations & functions require that certain _safety conditions_
      must be met for the operation/function to be memory safe.
   1. Every `unsafe` block must satisfy the safety conditions of its contents.
   1. Every `unsafe` function must satisfy the safety conditions of its
      contents, so long as its own (documented) safety conditions are met.

As an example: dereferencing a pointer has a safety condition: the pointer has a
valid referent.

---

## Example

Does this function adhere to the Abstraction Safety Contract?

```rust
fn as_ref<'a, T>(t: *mut T) -> &'a T {
    unsafe { &*t }
}
```

--

No! It doesn't know that the referent of `t` is valid!

---

## Example II

What about this one?

```rust
/// Takes a valid pointer and returns a reference valid until that pointers
/// referent is invalidated.
unsafe fn as_ref<'a, T>(t: *mut T) -> &'a T {
    &*t
}
```

--

Yep!

---

## Example III

What about this one?

```rust
fn get<'a, T>(slice: &'a [T], index: usize) -> Option<&'a T> {
    if index < slice.len() {
        unsafe { Some(slice.get_unchecked(index)) }
    } else {
        None
    }
}
```

--

Yep!

---

class: middle center

## Assignment Briefing

---

## Assignment Briefing

For this assignment you'll be implementing reference-counted pointers.

Your type will be modelled after the standard library's `std::rc::Rc`. It must
satisfy the following interface:

   * `Rc::new` should store its input on the heap, and use reference counting to
     free that data only when the last `Rc` to it is gone.
   * `Rc::clone` should produce a new `Rc` which refers to the same data
   * `Rc::consume` should consume the input `Rc` if it is the only `Rc` to the
     underlying data and produce the underlying data. Otherwise it should
     produce the same `Rc`.
   * `Rc` should be `Deref`, and dereferencing it should produce the underlying
     data.

You're type should be called `MyRc`. The test suite it must pass is provided.

For this assignment I ask you not too look at how `std::rc::Rc` is implemented.
You may however, look at it's reference documentation and example code to
understand how reference counting works. Note: we're implementing a subset of
its functionality.

[double-list]: https://docs.google.com/drawings/d/1Ff_SUC1zhjpqf6dZrmm2unoIIQd8BXOGXe3FADH_OQM/pub?w=972&h=402
[vector]: https://docs.google.com/drawings/d/160gZqBeYDf1JO6Iorb7QaGZMSz9RzdkBRjAPni6LAWU/pub?w=924&h=321
[pgm-space]: https://docs.google.com/drawings/d/1aEPjCLLNNiJvZqGoXa0mBwjEj4K9AuIPEiKLNd-bjVI/pub?w=698&h=668
[pgm-space-ex]: https://docs.google.com/drawings/d/1BsOQV1XiDIe8SDieUwRGpJ8RPN4NqukR-rZiHytmbls/pub?w=698&h=668
[safe]: https://docs.google.com/drawings/d/12Hu0PKoZ9XD5NJAPZoQr4_gmC_dirTOtA3RKbPEl-EY/pub?w=698&h=668
[unsafe-bad]: https://docs.google.com/drawings/d/1-F1_gvhLct9zksxPiD2YGphm7bWMTWhb86mfxoxJ_JA/pub?w=698&h=668
[unsafe-good]: https://docs.google.com/drawings/d/13ZqhaohM91AUcIvdmJv9v-pCVL3za6tm62V35UdHqvg/pub?w=698&h=668
