layout: true
class: center middle

---

## Project Kickoff: Graphs

Also know as

### MemSafe meets DSLs

29 November 2016

---

layout: false
class: left

## Project Overview

You'll be designing a graph API, implementing it, and using it to implement a
pair of graph algorithms.

--

This is intentionally open-ended.

It could be a project on implementing a highly performant graph using `unsafe`.

It could be a project about implementing crazy graph algorithms.

It could be a project about making good use of the standard library.

It could be a project about designing a safe & ergonomic API.

---

## Timeline

By next Tuesday (4:15 pm) you must have

   * reviewed the `petgraph` API
   * written an (unimplemented) API for your own graph
   * described the algorithms you will implement

By Thursday, December 15th (10:00 pm) you must have

   * implemented your graph
   * implemented the algorithms
   * written tests that convince me (and you) that both are correct

---

## A Note on Memory Leaks

I've put instructions on Piazza that detail how to use valgrind to detect leaks
in Rust programs.

Make sure your graph doesn't leak memory.

## A Note on Group Work

It's allowed and even encouraged here.

