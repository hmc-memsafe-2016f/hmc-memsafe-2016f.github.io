layout: true
class: center middle

---

## Memory Safety and Rust

Alex Ozdemir

30 August 2016

---

class: center top

## Control vs. Safety

![The tradeoff][trade-off]

---

## In the Beginning

.left-column.left[
### Memory is just a row of bins:
* `fn read(Address) -> Value;`
* `fn write(Address,Value);`
]
.right-column[
![Memory and CPU relationship][memory-cpu]
]

---

## What are C and C++?

---
layout:false
## What are C and C++?

### Characteristics
* strong alignment of semantics and architecture
* direct control over memory
* zero-cost abstractions

### Benefits
* performance
   * speed
   * low memory usage

### Drawbacks
* easy to make mistakes
   * use after free
   * memory leaks
   * buffer overruns
---

## How do people write C/C++?

```c
int getaddrinfo(const char *node, const char *service,
                const struct addrinfo *hints,
                struct addrinfo **res);

void freeaddrinfo(struct addrinfo *res);
```

--

> The getaddrinfo() function allocates and initializes a linked list of
> addrinfo  structures,  one  for  each  network  address  that  matches node
> and service, subject to any restrictions imposed by hints, and returns a
> pointer to the start of the  list  in  res.  The items in the linked list are
> linked by the ai_next field.

> ...

> The freeaddrinfo() function frees the memory that  was  allocated  for  the
> dynamically allocated linked list res.

--

Summary: _carefully_

---

## Ownership Semantics

Programmers reason about who _owns_ data - who has access to it and who has
responsibility for freeing it.

These ideas eventually made their way into the C++'s Standard Template Library
(STL):

* `unique_ptr` - Unique pointer (unique owner, move semantics)
* `shared_ptr` - Shared access (reference counted)
* `weak_ptr` - Shared access (does not affect reference count)

---

## Ownership Semantics II

Modern C++ codebases make extensive use of ownership semantics.

```cpp
// From FB's ReactNative/ReactCommon/cxxreact/Instance.h:28
class Instance {
 public:
  ~Instance();
  void initializeBridge(
    std::unique_ptr<InstanceCallback> callback,
    std::shared_ptr<JSExecutorFactory> jsef,
    std::shared_ptr<MessageQueueThread> jsQueue,
    std::unique_ptr<MessageQueueThread> nativeQueue,
    std::shared_ptr<ModuleRegistry> moduleRegistry);
  ...
}
```

---

## What is Rust?

### Characteristics
* Ownership semantics




[trade-off]: https://docs.google.com/drawings/d/1DUug8LxqRalnfckaHwUTd8tyxWMxDn2Aubqr6v8-DyY/pub?w=894&h=134
[memory-cpu]: http://www.hitequest.com/Kiss/comp_arch_general.gif
