---

layout: assignment
title: "Week 5: Beyond Ownership"

---

## The Model

The idea for this week is to go beyond programs and data structures which can
be expressed using the Rust ownership system. To this end you'll be implementing
an in-memory revision tracking system, somewhat similar to git. The model is as
follows:

### Commits

The system tracks _commits_, where each commit is a single revision. Each commit
has exactly one parent, but multiple commits may have the same parent. In this
way, the commits form a tree.

### Branches

Commits can never be named directly, instead all commits are referred to using
_branches_. A branch is essentially just a label for some commit. When the
revision tracking is initialized with a first commit, the first branch,
_master_, is automatically created, and points to that commit.

### Making new branches

Further branches can be created by specifying an existing branch to "copy",
which will make the new branch point to the same commit as the old one. Branches
can also be created pointing to an ancestor of an existing commit (e.g. a
commit's 3rd parent).

In this way, commits which are ancestors of commits pointed to by branches may
be referenced. We say that any commit which is an ancestor of a commit pointed
to by a branch is _reachable_.

Branch names will always be alphanumeric strings.

### Making new commits

New commits are created by specifying which branch to commit to. This creates a
new commit which has that branch's old target as its parent, and updates the
branch to point to the new commit.

### Deleting branches (and commits)

Branches may also be deleted. When this happens, some commits may become
unreachable. These commits should be deleted - that is their memory should be
freed.

### Recap

Consider this image:

![Commits][commits]

   * Branch D could be created by creating a branch to the 2nd parent of C.
      * It could also be created by making a branch to the 2nd parent of A or B.
   * If a commit was added to branch B, then branch B would now point to a
     commit, _7_, whose parent would be _5_.
   * If branch A were deleted, then commits _3_ and _6_ would also be.

Note, while commits in this image have names (_1_, _2_, ...) for convenience,
commits in our system are anonymous.

## The Specification

Really, a specification by example. During its run, your program should provide
the user a prompt (`> `). When they enter commands it should provide output like
below.  The blank lines are required.

After every command the program should output some information about what
happened, as done in the example below.

```text
$ path/to/binary 'Starting Payload'
master -> 'Starting Payload'

> new branch A master               // master~0 is equivalent
A -> 'Starting Payload'

> new commit 'Payload 1' A
A -> 'Payload 1'

> new branch B A~1
B -> 'Starting Payload'

> new commit 'Payload 2' B
B -> 'Payload 2'

> new commit 'Payload 3' B
B -> 'Payload 3'

> delete branch B
B deleted
'Payload 3' deleted
'Payload 2' deleted

> delet branch B
Error

> delete branch B
Error
```

You may assume that all payloads contain only alphanumeric characters and
spaces.

You may assume that all branches are alphanumeric.

When commits must be deleted, they should be deleted starting with the commits
closest to the deleted branch.

If the user provides _any_ instructions that don't make sense, print Error.

After the user enters ctrl-D or ctrl-C, your program can do whatever it wants.


## Grading

This assignment won't be auto-graded, is optional, and is worth 2 bonus points.

You'll need to create your own cargo project, from scratch!

If you do it, fill out
[this](https://docs.google.com/forms/d/e/1FAIpQLSegU6Zb4mNUSpbhRE7fiFMojluoN9etYIZQrEgqi_WXv-1wrA/viewform) form.

[commits]: https://docs.google.com/drawings/d/1NdZZiarwfJQLxAPDmdggTZf_kKAj5GaUYKm5FOEDJPU/pub?w=556&h=456
