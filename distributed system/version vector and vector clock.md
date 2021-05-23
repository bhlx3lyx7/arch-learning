# Version Vector vs. Vector Clock

## Version Vector
A version vector is a mechanism for tracking changes to data in a distributed system, where multiple agents might update the data at different times.

检测多主系统的写冲突

- Initially all vector counters are zero.
- Each time a replica experiences a local update event, it increments its own counter in the vector by one.
- Each time two replicas a and b synchronize, they both set the elements in their copy of the vector to the maximum of the element across both counters: `Va(x) = Vb(x) = max(Va(x), Vb(x))`. After synchronization, the two replicas have identical version vectors.

Pairs of replicas, a, b, can be compared by inspecting their version vectors and determined to be either: identical (`a=b`), concurrent (`a||b`), or ordered (`a<b` or `b<a`). The ordered relation is defined as: Vector `a<b` if and only if every element of Va is less than or equal to its corresponding element in Vb, and at least one of the elements is strictly less than. If neither `a<b` or `b<a`, but the vectors are not identical, then the two vectors must be concurrent.

## Vector clock
A vector clock is a data structure used for determining the partial ordering of events in a distributed system and detecting causality violations.

决定分布式系统中的偏序关系, 因果关系检测

- `VC(x)` denotes the vector clock of event x, and `VC(x)z` denotes the component of that clock for process z.
- `VC(x) < VC(y)`, if and only if `VC(x)z` is less than or equal to `VC(y)z` for all process indices z, and at least one of those relationships is strictly smaller (that is, `VC(x)z' < VC(y)z'`).
- `x -> y` denotes that event x happened before event y. It is defined as: if `x -> y`, then `VC(x) < VC(y)`.

**Properties**
- Antisymmetry: if `VC(a)<VC(b)`, then `!(VC(b)<VC(a))`
- Transitivity: if `VC(a)<VC(b)` and `VC(b)<VC(c)`, then `VC(a)<VC(c)`; or, if `a -> b`; and `b -> c`, then `a -> c`

**Relation with other orders**
- Let `RT(x)` be the real time when event x occurs. If `VC(a)<VC(b)`, then `RT(a)<RT(b)`
- Let `C(x)` be the Lamport timestamp of event x. If `VC(a)<VC(b)`, then `C(a)<C(b)`


## References
- version vector: https://en.wikipedia.org/wiki/Version_vector
- vector clock: https://en.wikipedia.org/wiki/Vector_clock