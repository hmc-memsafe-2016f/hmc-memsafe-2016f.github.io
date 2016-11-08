layout: true
class: center middle

---

## Memory Safety

### Ownership III: Lifetimes

8 November 2016

---

layout: false
class: left

## Administrivia

   1. Old Bonuses
      * In some cases you'll be able to take a crack at old bonuses (if I don't
        have or release a solution to them). I'll call this out the week after
        the original bonus.

---

layout: false
class: left

## Outline of today

   1. Labelled loops.
   1. Resource and Variable Lifetimes
      * Dynamic Lifetimes
      * Static Approximations Thereof
      * Connection to Ownership
   1. Reference Lifetimes in Blocks
      * Reference lifetimes
      * The Outlives Relation
      * Borrow lifetimes
   1. Lifetimes in Function Signatures
   1. Lifetimes in Structures

---

class: center middle

## Assignment Debreif

--

I heard it was a rough one.

---

## Labelled Loops

In Rust, you can label your loop to specify which loop to break out of:

```rust
'search:
for city in cities {
   for official in city.officials() {
      if official.name = target_name {
         println!("{} is an official in {}", target_name,
                                             city.name);
         break 'search;
      }
   }
}
```

Hmm. Interesting way of naming loops. Interesting that loops and lexical scopes
(curly braces) tend to coincide.

---

## The Ownership System

Recall that the Rust ownership system (enforced by `borrowck`):

   1. Requires all data have a unique owner.
   1. Requires no reference outlives its referent.
   1. Prohibits simultaneous aliasing and mutation.

--

Today we're going to explore the meaning of "simultaneous" and "outlives".

--

We'll do so by _extending_ the types `&` and `&mut` (sort of like how one could
extend an `i` type into {`i8`, `i16`, `i32`, `i64`, and `isize`}).

---

class: center middle

## Dynamic Lifetimes and Static Approximations

---

## Lifetimes

Most of today's discussion will revolve around the second rule of the Ownership
system:

> No reference may outlive its referent.

--

To make sense of this we'll define a few _dynamic_ concepts:

   * Resource (value) lifetime:
      * The time from when a resource is obtained to when it is released
         * Memory
         * File Handles
   * Variable (binding) lifetime:
      * Time time from a variable's first assignment to last use*.


By *dynamic* we mean that these concepts are defined with respect to program
traces: records of all actions taken by a program at runtime.

---

## Value and Binding lifetimes
.left-column[
### Source
```rust
fn main() {
   let x = Box::new(5);
   println!("{}", *x);
}
```
]
.right-column[
### Trace
```
0) a is an allocation with 5
1) x = &a
2) print *x
3) free a
```
]

.left[
What is the lifetime of the allocation?

What is the lifetime of the variable `x`?
]

--

The allocation is live from 0 to 3. `x` is live from 1 to 2. Notice that the
resource outlives it's binding.

---

## Value and Binding lifetimes
.left-column[
### Source
```rust
fn main() {
   let x = Box::new(5);
   println!("{}", *x);
}
```
]
.right-column[
### Trace
```
0) a is an allocation with 5
1) x = &a
2) free a
3) print *x
```
]

.left[
What is the lifetime of the allocation?

What is the lifetime of the variable `x`?
]

--

The allocation is live from 0 to 2. `x` is live from 1 to 3. This trace is
problematic because the resource _does not_ outlive its binding.

---

## Static Approximations

Reasoning about specific program executions doesn't already meet our needs. It's
great to precisely specify the lifetime of a variable in one execution, but that
doesn't tell us how to allocate registers when generating assembly.

Often we want _static approximations_ of variable and resource lifetime.

```rust
fn main() {              // 0
   let x = 5;            // 1
   let y = x + 1;        // 2
   println!("{}", y);    // 3
}
```

Here we could identify source lines 1 - 2 as the lifetime of `x`. However,
while the dynamic lifetime of a variable (it's lifetime in an execution trace)
can always be exactly determined a _conservative, static approximation_ is not
always exact.

Question: Can you think of a program where the dynamic lifetime of a variable
cannot be exactly statically approximated?

---

## Static Approximations

One program where the dynamic lifetime of a variable cannot be exactly
statically approximated:

```rust
fn main() {               // 0
   let x = 5;             // 1
   if bool_from_user() {  // 2
      println!("{}", x);  // 3
   }                      // 4
}
```

--

A particular program trace would give `x` a dynamic lifetime of 1 - 1 or 1 - 3.
A conservative static approximation would say (at least) 1 - 3.

---

## Lifetimes in Rust

Since Rust features a **static** type system, it reasons about static
approximations of resource and variable lifetimes.

From hereon out, "lifetime" means a static approximation of dynamic lifetimes.

---

## Lifetimes and Ownership

This idea of resource and variable lifetimes can be related to the concept of
ownership.

If all values are owned by a unique variable, the Rust dictates that the value's
lifetime will end immediately after the owning variable goes out of scope.

Critical Note: A variable's scope conservatively approximates its lifetime.

```rust
fn main() {
   let x = 5;     // -+ ----------------+
   let y = 6;     //  | x's lifetime    |
   let z = x + y; // -+                 | x's scope
   let a = z + y; //                    |
}                 // -------------------+
```


---

class: middle center

## Reference Lifetimes in Blocks

---

## The Outlives Relation

Recall the second rule of the ownership system:

> No reference may outlive its referent.

Can we express this using lifetimes? Indeed!
To improve our idea of the outlives relation we'll extend the `&T` type.

Specifically, we'll add more information to the type. We'll extend the type `&T`
into `&'lifetime T`. If we say `x` has type `&'a T`, we're saying that `x`'s
referent is valid for lifetime `'a`.

_NOT REAL SYNTAX: **`'b : { .. }` is NOT RUST SYNTAX**_

```rust
'b: {
    'a: {
        let x = 5;
        let r: &'a i32 = &x;
    }
}
```

Could we say that `r: &'b i32`?
--
No!

---

## The Outlives Relation

We'll use these extended reference types to take another crack at the outlives
relation, by imposing _three_ rules.

   1. If `x` has type `T` and lifetime (scope) `'a`, then `&x` has type `&'a T`.
   2. If `x: &'a T = y` and `y: &'b T`, then `'b` must contain `'a`.
   3. If `x` has type `&'b T`, then `'b` must contain the lifetime (scope) of
      `x`.

---

## Example 1

   1. If `x` has type `T` and lifetime (scope) `'a`, then `&x` has type `&'a T`.
   2. If `x: &'a T = y` and `y: &'b T`, then `'b` must contain `'a`.
   3. If `x` has type `&'b T`, then `'b` must contain the lifetime (scope) of
      `x`.

```rust
'a: {
   let x = 5;
   let rx: &i32 = &x;
}
```

Question: What is the full type of `&x`? Of `rx`? What is the lifetime (scope)
of `rx`?

--

Both `&x` and `rx` have type `&'a i32`. `rx` has lifetime `'a`.


---

## Example 2

   1. If `x` has type `T` and lifetime (scope) `'a`, then `&x` has type `&'a T`.
   2. If `x: &'a T = y` and `y: &'b T`, then `'b` must contain `'a`.
   3. If `x` has type `&'b T`, then `'b` must contain the lifetime (scope) of
      `x`.

```rust
fn main() 'a: {
   let r: &i32;
   'b: {
      let n = 5;
      let rtmp = &n;
      r = rtmp;
   }
}
```

Is there any assignment of type for `&n`, `rtmp`, and `r` which satisfy the
above rules?

--

No.

`n` must outlive `rtmp`'s type, which must outlive `r`'s type, which must
outlive `r`. But `n` does not outlive `r`.

---

## Recap

If we extend reference types `&T` to some `&'l T`, where `&'l T` means that the
underlying `T` is valid for at least `'a`, then we can precisely encode
lifetimes into the type system.

This allows us to prevent references from outliving referents.

--

### But ...

You may have noticed that you don't see _that_ many `'a`s floating around in
Rust code.

Within blocks `rustc` will infer the extended types of your references. In fact,
it is impossible to name your blocks and use those names to fully specify the
types of references.

```rust
'a: {
    let x: &'a i32 = &5;
}
```

---

## Reference Lifetime Inference

`rustc` is happy to infer reference lifetimes within blocks. However, it _does
not_ infer reference lifetimes across function boundaries.

Thus we find ourselves concerned with specifying the lifetimes of function
parameters and return types.

--

`rustc` does however use a set of _elision rules_ to allow you to elide
lifetimes in common situations. (Thank goodness).

---

class: middle center

## Reference Lifetimes in Function Signatures

---

## The High Level

Consider a map method that does a lookup. Such a method might take a reference
to a map and a key, and return a reference to the value. Something like:

```rust
impl Map<K, V> {
    fn get(&self, key: &K) -> &V { .. }
}
```

Since the reference to the value is a reference to data stored in a map, we need
to be able to express that this value reference is only valid for as long as the
map (self) reference is.

---

## Get

We can say that the map must "outlive" the value using the reference lifetime
notation and the outlives relation: `'a: 'c`.

```rust
impl Map<K, V> {
    fn get<'a, 'b, 'c>(&'a self, key: &'b K) -> &'c V
    where 'a: 'c
    { .. }
}
```

--

This ends up being equivalent to:

```rust
impl Map<K, V> {
    fn get<'a, 'b>(&'a self, key: &'b K) -> &'a V
    { .. }
}
```

--

(its equivalent because actual parameters are assigned to formal parameters, and
similar things involving the return value)

---

## Lifetime puzzles:

Write the following using explicit lifetime parameters:

```rust
fn max(x: &i32, y: &i32) -> &i32 {
   if x > y { x } else { y }
}
```

```rust
fn idx<T>(array: &[T], i: &usize) -> &T {
   &array[*i]
}
```

```rust
fn split(p: &(i32, i32)) -> (&i32, &i32) {
   (&p.0, &p.1)
}
```

---

## Lifetime puzzles:

With explicit lifetime parameters.

```rust
fn max<'a>(x: &'a i32, y: &'a i32) -> &'a i32 {
   if x > y { x } else { y }
}
```

```rust
fn idx<'a, 'b, T>(array: &'a [T], i: &'b usize) -> &'a T {
   &array[*i]
}
```

```rust
fn split<'a>(p: &'a (i32, i32)) -> (&'a i32, &'a i32) {
   (&p.0, &p.1)
}
```

---

## Lifetime Elision Rules

While Rust will not infer lifetimes at function boundaries, it does use
a simple set of rule which allows you to elide them:

   * Each elided lifetime in the inputs becomes a distinct lifetime parameter.
   * If there is exactly one input lifetime (elided or not), that lifetime is
     assigned to all elided output lifetimes.
   * If there are multiple input lifetime positions, but one of them is `&self`
     or `&mut` self, the lifetime of self is assigned to all elided output
     lifetimes.

This simplifies the prior:

```rust
    fn get<'a, 'b>(&'a self, key: &'b K) -> &'a V { .. }
```

to 

```rust
    fn get(&self, key: &K) -> &V { .. }
```

---

class: middle center

## Lifetimes in Structures

---

## Lifetimes in Structures

Recall our earlier example:

```rust
fn max(x: &i32, y: &i32) -> &i32 {
   if x > y { x } else { y }
}
```

What if we wanted to write `max` for a `struct` of our own?

```rust
struct Simple { v: &i32 }

fn max(x: Simple, y: Simple) -> Simple {
   if x.v > y.v { x } else { y }
}
```

How would we express the lifetime bounds?

---

## Lifetimes in Structures

We talk about the lifetime associated with structure by making those structures
generic over the lifetimes they're connected to:

```rust
struct Simple<'a> { v: &'a i32 }

fn max<'a>(x: Simple<'a>, y: Simple<'a>) -> Simple<'a> {
   if x.v > y.v { x } else { y }
}
```

--

Example from `std`: an iterator over a slice:

```rust
impl [T] {
    fn iter<'a>(&'a self) -> std::slice::Iter<'a, T> { .. }
}
```

---

## Lifetimes: Comprehensive Example

Lifetimes also 

```rust
impl<'a, T> IntoIterator for &'a Vec<T> {
    type Item = &'a T;
    type IntoIter = slice::Iter<'a, T>;

    fn into_iter(self) -> slice::Iter<'a, T> {
        self.iter()
    }
}
```

---

class: middle center

## That's all!

This week you'll be writing a small set of datastructure which involve various
lifetime relationships.

