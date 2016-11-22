layout: true
class: center middle

---

## Non-Lexical Lifetimes

22 November 2016

Credits: [Niko Matsakis](http://smallcultfollowing.com/babysteps/blog/2016/04/27/non-lexical-lifetimes-introduction/)

---

layout: false
class: left

## Program 1

```rust
fn foo() {
    let mut data = vec!['a', 'b', 'c'];
    let r = &mut data[..];
    r.push('d');
    data.push('e');
    data.push('f');
}
```

---

## Program 2

```rust
use std::collections::HashMap;
fn process_or_default<K,V>(map: &mut HashMap<K,V>,
                           key: K)
      where K: std::cmp::Eq + std::hash::Hash,
            V: Default
{
    match map.get_mut(&key) {
        Some(value) => process(value),
        None => {
            map.insert(key, V::default());
        }
    }
}

fn process<T>(t: T) { }
```

---

## Program 3

```rust
fn get_default<'m,K,V>(map: &'m mut HashMap<K,V>, key: K)
                      -> &'m mut V
      where K: std::cmp::Eq + std::hash::Hash,
            V: Default
{
    match map.get_mut(&key) {
        Some(value) => value,
        None => {
            map.insert(key, V::default());
            map.get_mut(&key).unwrap()
        }
    }
}
```

---

class: center middle

## Memory Safety

### Unsafe & The Standard Library

22 November 2016

---

layout: false
class: left

## Outline of today

   1. Assignment Recap
      * Pointer: Dereference vs. Read
   1. Threads
   1. Send and Sync
   1. Type Tour
      * Rc, Arc
      * Cell, RefCell
      * RwLock
      * Mutex

---

class: center middle

## Assignment Recap

---

## The Difference Between ptr::read and dereference

Compile and run this code to find out!

```rust
struct A { x: i32 }

impl Drop for A {
    fn drop(&mut self) { println!("Dropped"); }
}

fn main() {
    let x = A { x: 5 };
    let p = &x as *const _;
    let r = unsafe {
        &*p
        //&std::ptr::read(p)
    }
}
```

---

class: center middle

## Threads

---

## Thread API

Let's take a look at the threading API.

```rust
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
   where F: FnOnce() -> T,
         F: Send + 'static,
         T: Send + 'static
```

---

## Thread API

Let's take a look at the threading API.

```rust
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
   where F: FnOnce() -> T,
         F: Send + 'static,
         T: Send + 'static
```

Do you expect the following to work?

```rust
use std::thread;

thread::spawn(|| {
    println!("{}", 5);
}).join().unwrap();
```

---

## Thread API

Let's take a look at the threading API.

```rust
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
   where F: FnOnce() -> T,
         F: Send + 'static,
         T: Send + 'static
```

What about the following?

```rust
use std::thread;

let x = Box::new(5);

thread::spawn(|| {
    let y = x;
    println!("{}", y);
}).join().unwrap();
```

---

## Thread API

Let's take a look at the threading API.

```rust
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
   where F: FnOnce() -> T,
         F: Send + 'static,
         T: Send + 'static
```

What about the following?

```rust
use std::thread;

let x: &Box<_> = &Box::new(5);

thread::spawn(move || {
    let y = x;
    println!("{}", y);
}).join().unwrap();
```

---

## Thread API

Let's take a look at the threading API.

```rust
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
   where F: FnOnce() -> T,
         F: Send + 'static,
         T: Send + 'static
```

What about the following?

```rust
use std::thread;

let x: &str = "Hi there!";

thread::spawn(move || {
    let y = x;
    println!("{}", y);
}).join().unwrap();
```

---

## Thread API

Let's take a look at the threading API.

```rust
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
   where F: FnOnce() -> T,
         F: Send + 'static,
         T: Send + 'static
```

What about the following?

```rust
use std::thread;

let x: &str = "Hi there!";

thread::spawn(move || {
    let y = x;
    println!("{}", y);
}).join().unwrap();
```

---

## Thread API

Let's take a look at the threading API.

```rust
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
   where F: FnOnce() -> T,
         F: Send + 'static,
         T: Send + 'static
```

What about the following?

```rust
use std::thread;
use std::rc::Rc;

let rc = Rc::new(5);
let rc2 = rc.clone();

thread::spawn(move || {
    let y = rc2;
    println!("{}", y);
}).join().unwrap();
```

---

## Send and Sync

* `Send` means "this type can be sent to another thread without causing memory
  errors"
   * `Rc` is not `Send` (see it's documentation)

--

```rust
use std::thread;

struct B{ i: Box<i32> };

let b = B{ i: Box::new(5) };

thread::spawn(move || {
    let y = b;
    println!("{}", y.i);
}).join().unwrap();
```

--

We can see that `B` has "auto-derived" `Send`

---

## Auto-Deriving Send

A type whose components are all `Send` is also `Send`.
Which of the following are `Send`?

```rust
use std::rc::Rc;
struct A {
    i: i32,
    b: Box<i32>,
}
enum B {
    Just(Rc<i32>),
    Nothing
}
struct C {
    f: &i32,
}
struct D<T> {
    f: &T,
}
```

--

A is, B is not, C is, and D depends.

---

## The Importance of Sync

`Sync` is a trait which is intricately connected to `Send`.

`T` is `Sync` *if and only* if `&T` is `Send`. See the
[documentation](https://doc.rust-lang.org/std/marker/trait.Send.html).

Question: Are there any types which are not `Sync`?

---

## Implementing Sync and Send

You can also force a type to **be** or **not be** `Send` or `Sync`.

```rust
use std::thread;
use std::rc::Rc;

struct B{ i: Box<i32> };

impl !Send for B { };

let b = B{ i: Box::new(5) };

thread::spawn(move || {
    let y = b;
    println!("{}", y.i);
}).join().unwrap();
```

--

Well, almost.

---

## Send and Sync in Action

To explore `Send` and `Sync` a bit more, we'll take a look at the following:

   * `Arc`
   * `Cell/RefCell`
   * `Mutex`
   * `RwLock`
   * `mpsc::channel`

Get a sense for the API each type provides. Why would you use it, how would you
use it? Put an example on the board.

Pay special attention to whether the types (and related types) are `Send` and/or
`Sync`, and why!

---

## What the Future Holds

### The Assignment

The assignment this week is strictly optional (worth 2 bonus points).

It is a simplified version of git, which is easiest to implement using types
from `std`.

### After Thanksgiving

Presentations pick up in earnest (talk to me!)

We begin the final project!
