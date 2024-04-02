# "Programming Distributed Systems" by Mae Milano

link: https://www.youtube.com/watch?v=Mc3tTRkjCvE

## Problems in distributed system
- data race
	+ two threads access memory simultaneously, at least one of them is writing
	+ e.g. split increment
- coordination overhead
	+ transaction ensures atomicity, but runtime overhead is expensive
	+ consensus algorithm provides strong consistency, but hard to implement efficiently
- consistency error
	+ weak consistency lead to unexpected errors, hard to realize when programming
	+ maybe strong consistency required data depends on weak consistent mutation

## Example of a gaming system
1. table of win/loss number of users
	- weak consistency is tolerable
2. table of rank of users
	- strong consistency required
3. rank is populated by win/loss numbers
	- strong consistency depends on weak consistent mutation

**Rule 1: Strong outputs must stay strong**: Strong observation arer not influenced by weak operations

### Consistency level
Linearizability (strong) > causal consistency > eventual consistency

Improper influence: A less consistent read influences a more consistent write

### Potential solution
Upgrade the weak read to like quorum read, to make it stronger consistent

[MixT](https://github.com/mpmilano/MixT) can help prevent improper influence during compilation.

## When are weakly-consistent warranted

e.g. `Wins[p] >= 15`
- no concurrent mutation can violate this, because `Wins` number only increases
- then weak consistency is tolerable for this observation

**Rule 2: More restricted mutations allow more general observations**

e.g.
- constants: most observation, least mutation
- multi-writer: least observation, most mutation

### Monotonic objects
mutations are inflationary with respect to some order (CRDT), e.g.
- increment-only counters
- grow-only sets

monotonic objects can
- be correct under weak consistency
- avoid coordination

Many core system components are monotonic. 

[Derecho](https://derecho-project.github.io/) builds a state machine replication system, based on monotonicity.
- build strongly-consistent consensus atop the weak consistency
- monotonicity provides convergence and race freedom entirely for free

## Not everything is monotonic

e.g. split increment, a lock is needed to guarantee atomic operation

MixT introduces a flexible type system for fearless concurrency.

### Lock solution
- thread with lock can mutate the shared object
- mutual exclusive, one thread at a time
- challenge is to safely release the lock, need to track references to the locked objects

### Rust solution: allow only trees
- at most one reference to each locked object
- ownership exclusive, only one object can access to the shared object
- it is hard to implement doubly-linked list in Rust
- the workaround actually adds runtime overhead, and use unsafe internally
- root reason: distinct path might refer to the same object

### MixT solution: `iso` keyword
- introduce `iso` keyword in type system, means isolated data (seems like opposite to `volatile`)
- `iso` establishes statically-enforced boundaries to the object
- ownership exclusive, region is shared as a unit, no concurrency within a region

