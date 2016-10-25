layout: true
class: center middle

---

## Memory Safety

### The Ownership System

25 October 2016

---

layout: false
class: left

## Administrivia

Course Components:

* Assignments
   * 6 in total
   * Group work and pair programming are both okay.
* Presentations
   * Presenting a problem/solution, an intriguing feature of Rust
   * Connected [perhaps loosely] to the current assignment
   * Done once, with a partner (Signups Soon!)
* Final Projects
   * Graph APIs, Implementations, and Algorithms?
   * I'm open to other ideas too

---

## Administrivia

Grading:

   * Full marks for:
       * Completing all assignments, your presentation, your project, and doing
         a bonus or two OR
       * Putting in the 8 - 1.5 hours expected by Programming Practicum

Collaboration:

   * Pair programming
      * Okay -- include your partner's name in the PR.
   * Working with each other
      * Encouraged!
      * Requirement: you need to write your own code.

Communication:

   * Piazza
      * Tell me if you're not on the site!

---

class: center middle

# Ownership

---

## The Problem

   * Memory Management

## Solutions

   1. Garbage Collection
      * Reference Counting
         * Swift, Python, ObjC, ...
      * Tracing
         * JVM, Ruby, Javascript, ...
   2. Manual Memory Management
      * C, C++, Pascal, FORTRAN

Ownership is heavily inspired by manual memory management.

---

### Manual Memory Management Woes

Use after free:

```cpp
int silly_sum(vector<int>& v) {
   int * sum = new 0;
   for (int i : v)
      *sum += i
   return i;

   // Oops! `*sum` is leaked!
}
```

---

### Manual Memory Management Woes

Use after free:

```cpp
void main() {
   int * x = new 5;
   cout << *x << endl;  // "5"
   delete x;
   cout << *x << endl;  // Undefined Behavior (UB)
}
```

---

### Manual Memory Management Woes

Dangling pointer:

```cpp
void main() {
   int * x = new 5;
   int& r = *x;
   cout << r << endl;  // "5"
   delete x;
   cout << r << endl;  // Undefined Behavior (UB)
}
```

--

_The reference outlives the referent_

---

### Manual Memory Management Woes

Iterator invalidation:

```cpp
void main() {
   vector<int> xs;
   xs.push_back(5);
   auto it = xs.begin();
   cout << *it << endl;  // "5"
   xs.push_back(6);
   cout << *it << endl;  // Undefined Behavior (UB)
}
```

--

_Simultaneous aliasing and mutation_

---

## Summary

Some patterns of problematic memory management.

   1. Memory may be leaked (unused but un-freed).
   2. References may outlive the referrent
      * "Dangling pointer"
      * "Use after free"
   3. Simultaneous aliasing and mutation
      * "Iterator invalidation"

Ownership is designed to prohibit these patterns

---

## 1: Memory Leaks

Rust enforces that all values have an owner.

```rust
fn main() {
    let x = Box::new(x);
    // `x` owns a boxed integer

    println!("{}", *x);
}
```

---

## 1: Memory Leaks

Ownership is transferable. In particular, assignment (`=`) transfers ownership.

```rust
fn main() {
    let x = Box::new(5);
    let y = x;
    // `y` owns a boxed integer
    // `x` does not

    println!("{}", *y);
}
```

--

.center[
_move semantics_
]

---

## 1: Memory Leaks

Once ownership has been transferred, the value is no longer accessible.

```rust
fn main() {
    let x = Box::new(5);
    let y = x;
    // `y` owns a boxed integer
    // `x` does not

    println!("{}", *x); // ERROR: "use of moved value: `x`"
}
```

---

## Shortcomings of Ownership

Just having ownership is a pain:

```rust
#[derive(Debug)]
struct Vector(i32, i32);

impl Vector {
    fn scale(mut self, scalar: i32) {
        self.0 *= scalar;
        self.1 *= scalar;
    }
}

fn main() {
    let mut v = Vector(5, 2);
    v.scale(3);
    println!("{:?}", v);  // ERROR: "use of moved value: `v`"
}
```

--

We need a concept of _references_ or _borrowing data_.

---

## 2: Reference Lifetimes

We can create references to expressions using `&`.


```rust
fn main() {
    let x = Box::new(5);
    let y = &x;
    // `y` is a reference to the box
    // `x` owns the box

    println!("{}", *y);
    println!("{}", *x);
}
```

---

## 2: Reference Lifetimes

References are often used with methods:

```rust
#[derive(Debug)]
struct Vector(i32, i32);

impl Vector {
    fn scale(&mut self, scalar: i32) {
        self.0 *= scalar;
        self.1 *= scalar;
    }
}

fn main() {
    let mut v = Vector(5, 2);
    v.scale(3);
    println!("{:?}", v);      // Totally Valid
}
```

---

## 2: Reference Lifetimes

References are not permitted to outlive their referent!

```rust
fn main() {
    let rx;
    {
        let x = 5;
        rx = &x;
    }
    println!("{}", *rx);
}
```

--

Error: "`x` does not live long enough"

---

## 3: Aliasing and Mutation

You can create mutable alias (aliases that allow you to mutate the underlying
data):

```rust
fn main() {
    let mut x = 5;
    let rx = &mut x;
    *rx += 1;
}
```

---

## 3: Aliasing and Mutation

However, mutable references must be _unique_: they must be the only reference to
that data.

```rust
fn main() {
    let mut x = 5;
    let rx = &mut x;

    let rrx = &x;// Error:"cannot borrow `x` as immutable because
                 //        it is also borrowed as mutable"
}
```

---

## 3: Aliasing and Mutation

You also cannot mutate data while it is borrowed:

```rust
fn main() {
    let mut x = 5;
    let rx = &x;
    let x += 1;         // Error: "cannot assign to `x`
                        //         because it is borrowed"
}
```

---

### Ownership and Borrowing Rules: Recap

In a paragraph, the reference rules:

> First, any borrow must last for a scope no greater than that of the owner.
> Second, you may have one or the other of these two kinds of borrows, but not
> both at the same time:
>
>    * one or more references (&T) to a resource,
>    * exactly one mutable reference (&mut T).

-- _The Rust Book_

---

### Ownership and Borrowing Rules: Recap

We wanted to address the following problematic behavior:

   1. Memory being leaked
      * -> All data has an owner
   2. References outliving referents
      * -> Referents must outlive referents
   3. Simultaneous aliasing and mutation
      * -> Outstanding references block mutation

---

## References: Expressions, Types and Borrows

We've seen how to take an expression and borrow it:

```rust
let my_ref = &(my_expr);
```

Borrowing a `T` produces a reference type, `&T`, pronounced "and-T":
```rust
let r: &i32 = &5;
```

---

## References: Expressions, Types and Borrows

You can also borrow data using patterns:

```rust
let ref r = 5;
// r has type `&i32`.
```

An example is below. Notice that in pattern `&` is part of the type, and `ref`
is how you make references.

```rust
fn as_ref<T>(o: &Option<T>) -> Option<&T> {
    match o {
        &Some(ref inner) => Option(inner),
        &None => None,
    }
}
```

This is sometimes _really_ useful.

---

class: center, middle

## Traits

---

## Using Traits

Rust feature _Traits_, which are a lot like interfaces in OO languages. A common
trait is `Ord` -- the trait for ordered data.

If you're writing generic code then you can use traits to bound that
generic-ness:

```rust
fn max<T: Ord>(x: T, y: T) -> T {
    if (x >= y) {
        x
    } else {
        y
    }
}
```

What would happen if we removed the `: Ord`?

There are many other ways that traits are useful, but we won't explore them much
during this course.

---

## Implementing Traits

You can implement traits!

```rust
struct Pair(i32, i32);

impl Ord for Pair {
    fn cmp(&self, other: &Self) -> Ordering {
        match self.0.cmp(other.0) {
            Ordering::Less => Ordering::Less,
            Ordering::Less => self.1.cmp(other.1),
            Ordering::Less => Ordering::Greater,
        }
    }
}
```

This example doesn't _quite_ work. Ord requires PartialOrd already be
implemented.

---

## Deriving Traits

Many traits have a natural default implementation, including Debug, Ord, Eq,
Clone, and Hash.

You can use this implementation using the `derive` attribute:

```rust
#[derive(Ord, PartialOrd, PartialEq, Debug)]
struct Pair(i32, i32);
```

---

## The Copy Trait

One particularly important trait is Copy. Types that implement Copy will be
cloned rather than moved.

```rust
let x: i32 = 5;
let y = x;
println!("{}", x); // Totally fine
```

---

## Management tools from STD

There are a few particularly useful functions in `std`:

   * `mem::swap`
   * `mem::replace`
   * `Option::take`

---

## The Assignment

This week you'll be building a singly linked list.

Important things to think about:
   * How would one implement a singly linked list in C++? In particular, what
     members would your struct's have?
      * Node likely have pointer to nodes elsewhere on the heap. How does one
        represent a heap-pointer in Rust?
   * What are the ownership semantics of a singly linked list? The list must
     (transitively) own the whole list. Node may "own" other nodes...

---

## Outline

   * Identifying problematic behaviors
      * Ownership
      * Simultaneous aliasing and mutation
   * The rules
      * Ownership
      * Getting data:
         * Transferring ownership
         * Borrowing
         * Mutably borrowing
         * This has been ownership in expressions
      * In types
         * What type is a borrow?
         * Lifetimes.
      * In patterns
         * Types + Bindings
   * `Option::take`
   * Traits
      * Copy
   * `Box`

[memory-cpu]: http://www.hitequest.com/Kiss/comp_arch_general.gif
[map]: https://doc.rust-lang.org/std/result/enum.Result.html#method.map
[and-then-option]: https://doc.rust-lang.org/std/option/enum.Option.html#method.and_then
