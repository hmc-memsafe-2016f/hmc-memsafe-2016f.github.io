---
layout: page
title: Schedule, Slides, & Assignments
permalink: /schedule/
---

# Logistics

The class is embedded within CS 189: Programming Practicum, but will only occur
duing the second half of the Fall semester. Interested students should attend
the first CS 189 session for a brief class pitch, but will otherwise have no
obligations until after Fall Break.

One class begins we will meet during Programming Practicum's typical timeslot:
4:15 - 5:30, Tuesdays, in Shanahan 3460. There will also be frequent office/lab
hours:

   * 7:00 - 8:00 pm Monday, Shanahan B470
   * 3:00 - 4:00 pm Sunday, by the Cafe
   * 11:00 - 12:00 am Friday, Shanahan B454

# Schedule

Week |  Date  | Topic                          | Slides       | Assignment             | Due Date |
-----|--------|--------------------------------|--------------|------------------------|----------|
     | 09/07  | Memory Management and Rust     | [slides][sz] | --                     | --       |
0    | 10/17  | Getting Acquianted With Rust   | [slides][s0] | [Warm Up][hw0]         | 10/25    |
1    | 10/25  | The Ownership System           | [slides][s1] | [A Linked List][hw1]   | 11/1     |
2    | 11/1   | Ownership: Ramifications       | [slides][s2] | [Expressions][hw2]     | 11/8     |
3    | 11/8   | Lifetimes: Disambiguated       | [slides][s3] | [A Baby Database][hw3] | 11/15    |
4    | 11/15  | Unsafe:                        | [slides][s4] | [The Wild West][hw4]   | 11/22    |
5    | 11/22  | Send, Sync, Threads, & NLL     | [slides][s5] | [Git-rs][hw5](Optional)| 11/29    |
6    | 11/29  | Project Kickoff                | ??           | Graphs!                | ??       |
7    | 12/6   |                                | ??           | Graphs!                | ??       |

# Presentation Schedule

It can be found [here][prez]

# Potential Presentation Topics

   1. Debugging Memory Errors in Rust (Assignment 1, Bonus C). --Adam
   2. `replace_with` -- this turned into part of the "unsafe" assignmnent.
   3. Doubly-linked lists and the Rust ownership model.
   4. What exactly are `Fn`, `FnOnce`, `FnMut`? Function types in Rust.
   5. Macros --Eric
   6. Anonymous lifetimes in Rust: Elision versus anonymity.
   7. Writing (and explaining) a safe Rust program which leaks memory --Max and
      Daniel
   8. Type System Abuse: Compile Time Arithmetic in Rust
   9. `Sized`: Sized and unsized types in Rust. (Also super cool)
   10. Dereference coercions & partial moves
      * Why sometimes `p.x` and `p.y` can be independently moved and other times
        they can't. --Ross and Luis
   11. Allocators. How can one go about allocating memory in (unsafe) Rust?
   12. [`crossbeam`][crossbeam]: aturon's scoped thread library.
      * You know how threads require `'static` inputs? What if they didn't?
   13. Single Entry Multiple Exit/Static Single Assignment/Dominators take on Non-Lexical Lifetimes.
   14. Associated Type Constructors.
   15. Garbage collection traits.

[sz]: http://slides.com/alexozdemir/memory-safety-and-rust
[s0]: /slides/00/
[s1]: /slides/01/
[s2]: /slides/02/
[s3]: /slides/03/
[s4]: /slides/04/
[s5]: /slides/05/

[hw0]: /assignments/wk0/
[hw1]: /assignments/wk1/
[hw2]: /assignments/wk2/
[hw3]: /assignments/wk3/
[hw4]: /assignments/wk4/
[hw5]: /assignments/wk5/
[hw6]: https://www.youtube.com/watch?v=dQw4w9WgXcQ

[crossbeam]: http://aturon.github.io/crossbeam-doc/crossbeam/

[prez]: https://docs.google.com/spreadsheets/d/1EM6xf0YVGYcrNmU5CVSD_X-I0qlYnCBHO93KauNY_kI/edit?usp=sharing

[troll]: https://www.youtube.com/watch?v=dQw4w9WgXcQ

