layout: true
class: center middle

---

## Memory Safety

Alex Ozdemir

17 October 2016

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
   * I'm welcome to other ideas too

---

layout: center

## On our goals

---

## Installation

* We'll install Rust using Rustup!
   * We'll be using the `stable` build of the compiler for most of the course.

* It is found here: [https://rustup.rs/](https://rustup.rs/).

* If you use Arch you can use: `pacman -S rustup`.
   * A similar command *might* work for apt-get, Homebrew or other package
     managers.

---

class: middle, center
## Examples Time!

---

## Hello World

```rust
fn main() {
   println!("Why hello there!");
}
```

???

* Curly brace language
* Bare functions

---

## Variables

```rust
fn main() -> () {
   let name = "Prof. Lewis";
   println!("Why hello there, {}", name);
}
```

???

* Format macros
* Let bindings
   * Type inference
   * Q for next week: "What is the type on `name`?"

---

## Immutability by default

```rust
fn main() -> () {
   let name = "Prof. Lewis";
   name = "Prof. Ben";                    // ERROR
   println!("Why hello there, {}", name);
}
```

--

Fixed:

```rust
fn main() -> () {
*  let mut name = "Prof. Lewis";
   name = "Prof. Ben";                    // Okay
   println!("Why hello there, {}", name);
}
```

---

## Functions

### Explicit return

```rust
fn increment(x: i64) -> i64 {
   return x + 1;
}
```

???

* Type annotations in signatures

--

### Implicit return

```rust
fn increment(x: i64) -> i64 {
   x + 1
}
```

When possible, implicit return is the idiom. Explicit return is fine when it is
more convenient.

???

* Expression orientation `x + 1` is an expression, `x + 1;` and `return x + 1`
    are statements.
* `{ EXPR }` is also an expression
* Implicit return

---

## Branching

Okay:
```rust
fn max(a: isize, b: isize) -> isize {
   if a < b {
      return b;
   } else {
      return a;
   }
}
```

???

* Parenthesis are optional
* Curly braces are not

--

Better:

```rust
fn max(a: isize, b: isize) -> isize {
   if a < b {
      b
   } else {
      a
   }
}
```

???

* `if` statements are actually `if` expressions!

---

## For Loops

```rust
fn is_prime(n: u64) -> bool {
   for div in 2..n {
      if n % div == 0 {
         return false;
      }
   }
   true
}
```

???

* Integer range syntax
* Mod syntax (most syntax inherited from C)
* Early return is sometimes useful!

---

## While Loops

```rust
fn is_prime(n: i64) -> bool {
   let mut div = 2;
   while div < n {
      if n % div == 0 {
         return false;
      }
      div += 1;
   }
   true
}
```

???

* No `i++`

---

## 'Forever' Loops

```rust
fn is_prime(n: i64) -> bool {
   let mut div = 2;
   loop {
      if n % div == 0 {
         return false;
      }
      div += 1;
      if div == n {
         break;
      }
   }
   true
}
```

???

* No `i++`

---

## Type Aliases

Type aliases:

```rust
type Quotient = (i64, i64);

fn main() {
   let quo: Quotient = (6, 3);
}
```

???

   * Explicit type annotations.
   * Literally just an alias.

---

## Tuple structures

```rust
struct Quotient(i64, i64);

fn main() {
   let quo = Quotient(6, 3);
}
```

---

## Record structures

```rust
struct Quotient{
   dividend: i64,
   divisor:  i64,
}

fn main() {
   let quo = Quotient{ dividend: 6, divisor: 3 };
}
```

---

## `Struct` constructors

```rust
struct Quotient{
   dividend: i64,
   divisor:  i64,
}

impl Quotient {
   fn new(dividend: i64, divisor: i64) -> Self {
      Quotient{ dividend: dividend, divisor: divisor }
   }
}

fn main() {
   let quo = Quotient::new(6, 3);
}
```

???

   * `Self` is an alias for the type in the `impl`

---

## Method Syntax

```rust
struct Quotient{
   dividend: i64,
   divisor:  i64,
}

impl Quotient {
   fn new(dividend: i64, divisor: i64) -> Self {
      Quotient{ dividend: dividend, divisor: divisor }
   }
   fn eval(&self) -> i64 {
      self.dividend / self.divisor
   }
}

fn main() {
   let quo = Quotient::new(6, 3);
   quo.eval();
}
```

???

   * If the first argument is `self` then you can use dispatch syntax.
       * Fear not, this is static dispatch here.
   * the `&` in `&self` means "take by reference" or, "take by a pointer".
       * This happens automatically from the caller's perspective
       * More on this later

---

## Enums

You can express different options (classically called "disjoint unions"):

```rust
enum Option<T> {
   Some(T),
   None
}

fn main() {
   let o = Option::Some(5);
}
```

---

## Reading Enums: Pattern Matching

```rust
fn main() {
   let o = Some(5);
   match o {
      Some(i) => println!("The number is {}", i),
      None => println!("No number"),
   };
}
```

???

* `Option`, `Some`, and `None` are in the standard library.

---

# Match: Also an Expression

Of course, `match` constructs are also expressions

```rust
fn main() {
   let integer = Some(5);
   let half = match integer {
      Some(i) => Some(i / 2),
      None => None,
   };
}
```

--

However, the idiom is to use `map`. It's function signature is:

```rust
fn map<U, F>(self, op: F) -> Result<U, E> where F: FnOnce(T) -> U
```

```rust
fn main() {
   let integer = Some(5);
   let half = integer.map(|i| i / 2);
}
```

???

The `|i| i / 2` is an anonymous function which takes `i` as an input an returns
`i / 2`.

---

## Option<T>::and_then

Sometimes you want to do a "map", but the result of the "map" is actually an
`Option`. This is what `and_then` is for.

[Function signature][and-then-option]

```rust
fn main() {
   let integer = Some(5);
   let div = 0;
   let quotient = integer.and_then(|i| {
      if div == 0 {
         None
      } else {
         Some(i / div)
      }
   });
}
```

---

## Optional Exercise

Build functions `divide` and `multiply` for `Option<i64>` so that divide by zero
errors produce `None` values rather than runtime errors.

---

## Result<T>

While `Option<T>` can be used to express "`T` or an error", sometimes you want
more descriptive errors.

```rust
enum Result<T, E> {
   Ok(T),
   Err(E),
}
```

Let's take a look at the standard library's documentation for result!

---

## try!: Motivation

Sometimes you want to do a long series of computations, each of while might
fail:

```rust
fn process_file(file_name: String) -> Result<i64,MyError> {

   // Open file. If it fails, return the error

   // Read file. If it fails, return the error

   // Parse the contents as an integer. If it fails, return the error

   // Return the integer
}
```

This is what `try!` is for!

---

## try!: Definition

One can understand try like this:

Pseudo explansion rules:
```
     try!(Ok(stuff)) => stuff,
     try!(Err(err))  => return Err(err),
```

--

Macro definition:
```rust
macro_rules! try {
    ($e:expr) => (match $e {
        Ok(val) => val,
        Err(err) => return Err(err),
    });
}
```

[memory-cpu]: http://www.hitequest.com/Kiss/comp_arch_general.gif
[map]: https://doc.rust-lang.org/std/result/enum.Result.html#method.map
[and-then-option]: https://doc.rust-lang.org/std/option/enum.Option.html#method.and_then
