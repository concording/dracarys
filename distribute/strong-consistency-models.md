# [Strong consistency models](https://aphyr.com/posts/313-strong-consistency-models)


_Update, 2018-08-24: For a more complete, formal discussion of consistency models, see [jepsen.io](https://jepsen.io/consistency/)._

_Network partitions [are going to happen](http://aphyr.com/posts/288-the-network-is-reliable)._ Switches, NICs, host hardware, operating systems, disks, virtualization layers, and language runtimes, not to mention program semantics themselves, all conspire to delay, drop, duplicate, or reorder our messages. In an uncertain world, we want our software to maintain some sense of _intuitive correctness_.

Well, obviously we want intuitive correctness. Do The Right Thing™! But what exactly _is_ the right thing? How might we describe it? In this essay, we’ll take a tour of some “strong” consistency models, and see how they fit together.

## Correctness

There are many ways to express an algorithm’s abstract behavior–but just for now, let’s say that a _system_ is comprised of a _state_, and some _operations_ that transform that state. As the system runs, it moves from state to state through some history of operations.

![uniprocessor-history.jpg](https://aphyr.com/data/posts/313/uniprocessor-history.jpg "uniprocessor-history.jpg")

For instance, our state might be a variable, and the _operations_ on the state could be the writes to, and reads from, that variable. In this simple Ruby program, we write and read a variable several times, printing it to the screen to illustrate the reads.

`x = "a"; puts x; puts x x = "b"; puts x x = "c" x = "d"; puts x`

We already have an _intuitive_ model of this program’s correctness: it should print “aabd”. Why? Because each of the statements _happen in order_. First we `write the value a`, then `read the value a`, then `read the value a`, then `write the value b`, and so on.

Once we set a variable to some value, like `a`, reading it should return `a`, until we change the value again. Reading a variable returns the most recently written value. We call this kind of system–a variable with a single value–a _register_.

We’ve had this model drilled into our heads from the first day we started writing programs, so it feels like second nature–but this is _not_ the only way variables could work. A variable could return _any_ value for a read: `a`, `d`, or `the moon`. If that happened, we’d say the system was _incorrect_, because those operations don’t align with our _model_ of how variables are supposed to work.

This hints at a definition of _correctness_ for a system: given some _rules_ which relate the operations and state, the history of operations in the system should always _follow those rules_. We call those rules a _consistency model_.

We phrased our rules for registers as simple English statements, but they could be arbitrarily complicated mathematical structures. “A read returns the value from two writes ago, plus three, except when the value is four, in which case the read may return either cat or dog” is a consistency model. As is “Every read always returns zero”. We could even say “There are no rules at all; every operation is permitted”. That’s the _easiest_ consistency model to satisfy; every system obeys it trivially.

More formally, we say that a consistency model is the _set of all allowed histories of operations_. If we run a program and it goes through a sequence of operations in the allowed set, that particular execution _is consistent_. If the program screws up occasionally and goes through a history _not_ in the consistency model, we say the history was _inconsistent_. If _every_ possible execution falls into the allowed set, the system _satisfies_ the model. We want real systems to satisfy “intuitively correct” consistency models, so that we can write predictable programs.

## Concurrent histories

Now imagine a concurrent program, like one written in Node.js or Erlang. There are multiple logical threads of control, which we term “processes”. If we run a concurrent program with two processes, each of which works with the same register, our earlier register invariant could be _violated_.

![multiprocessor-history.jpg](https://aphyr.com/data/posts/313/multiprocessor-history.jpg "multiprocessor-history.jpg")

There are two processes at work here: call them “top” and “bottom”. The top process tries to `write a`, `read`, `read`. The bottom process, meanwhile, tries to `read`, `write b`, `read`. Because the program is _concurrent_, the operations from these two processes could interleave in _more than one order_–so long as the operations for a single process happen in the order that process specifies. In this particular case, top writes `a`, bottom reads `a`, top reads `a`, bottom writes `b`, top reads `b`, and bottom reads `b`.

In this light, the concept of concurrency takes on a different shape. We might imagine every program as _concurrent by default_–when executed, operations could happen in any order. A _thread_, a _process_–in the logical sense, anyway–is a _constraint_ over the history: operations belonging to the same thread _must_ take place in order. Logical threads impose a partial order over the allowed operations.

Even with that order, our register invariant–from the point of view of an individual process–no longer holds. The process on top wrote `a`, read `a`, then read `b`–which is _not_ the value it wrote. We must _relax_ our consistency model to usefully describe concurrency. Now, a process is allowed to read the most recently written value from _any_ process, not just itself. The register becomes a _place of coordination_ between two processes; they _share state_.

## Light cones

![lightcone-history.jpg](https://aphyr.com/data/posts/313/lightcone-history.jpg "lightcone-history.jpg")

Howerver, this is not the full story: in almost every real-world system, processes are _distant_ from each other. An uncached value in memory, for instance, is likely on a DIMM _thirty centimeters away_ from the CPU. It takes light over a full nanosecond to travel that distance–and real memory accesses are much slower. A value on a computer in a different datacenter could be thousands of kilometers–hundreds of milliseconds–away. We just can’t send information there any faster; physics, thus far, forbids it.

This means our operations are _no longer instantaneous_. Some of them might be so fast as to be negligible, but in full generality, operations _take time_. We _invoke_ a write of a variable; the write travels to memory, or another computer, or the moon; the memory changes state; a confirmation travels back; and then we _know_ the operation took place.

![concurrent-read.jpg](https://aphyr.com/data/posts/313/concurrent-read.jpg "concurrent-read.jpg")

The delay in sending messages from one place to another implies _ambiguity_ in the history of operations. If messages travel faster or slower, they could take place in unexpected orders. Here, the bottom process invokes a read when the value is `a`. While the read is in flight, the top process writes `b`–and by happenstance, its write arrives _before_ the read. The bottom process finally completes its read and finds `b`, not `a`.

This history violates our concurrent register consistency model. The bottom process did _not_ read the current value at the time it invoked the read. We might try to use the completion time, rather than the invocation time, as the “true time” of the operation, but this fails by symmetry as well; if the read arrives _before_ the write, the process would receive `a` when the current value is `b`.

In a distributed system–one in which it takes time for an operation to take place–we must _relax_ our consistency model again; allowing these ambiguous orders to happen.

How far must we go? Must we allow _all_ orderings? Or can we still impose some sanity on the world?

## Linearizability

![finite-concurrency-bounds.jpg](https://aphyr.com/data/posts/313/finite-concurrency-bounds.jpg "finite-concurrency-bounds.jpg")

On careful examination, there are some bounds on the order of events. We can’t send a message back in time, so the _earliest_ a message could reach the source of truth is, well, _instantly_. An operation cannot take effect _before_ its invocation.

Likewise, the message informing the process that its operation completed cannot travel back in time, which means that no operation may take effect _after_ its completion.

If we assume that there _is_ a single global state that each process talks to; if we assume that operations on that state take place _atomically_, without stepping on each other’s toes; then we can rule out a great many histories indeed. We know that **each operation appears to take effect atomically at some point between its invocation and completion**.

We call this consistency model _linearizability_; because although operations are concurrent, and take time, there is some place–or the appearance of a place–where every operation happens in a nice linear order.

![linearizability-complete-visibility.jpg](https://aphyr.com/data/posts/313/linearizability-complete-visibility.jpg "linearizability-complete-visibility.jpg")

The “single global state” doesn’t have to be a single node; nor do operations actually have to be atomic. The state could be split across many machines, or take place in multiple steps–so long as the external history, from the point of view of the processes, appears _equivalent_ to an atomic, single point of state. Often, a linearizable system is made up of smaller coordinating processes, each of which is itself linearizable; and _those_ processes are made up of carefully coordinated smaller processes, and so on, down to [linearizable operations provided by the hardware](http://en.wikipedia.org/wiki/Compare-and-swap).

Linearizability has powerful consequences. Once an operation is complete, _everyone_ must see it–or some later state. We know this to be true because each operation _must_ take place before its completion time, and any operation invoked subsequently _must_ take place after the invocation–and by extension, after the original operation itself. Once we successfully write `b`, every subsequently invoked read _must_ see `b`–or some later value, if more writes occur.

We can use the atomic constraint of linearizability to _mutate state safely_. We can define an operation like `compare-and-set`, in which we set the value of a register to a new value if, and only if, the register currently has some other value. We can use `compare-and-set` as the basis for mutexes, semaphores, channels, counters, lists, sets, maps, trees–all kinds of shared data structures become available. Linearizability guarantees us the safe interleaving of changes.

Moreover, linearizability’s time bounds guarantee that those changes will be visible to other participants after the operation completes. Hence, linearizability prohibits stale reads. Each read will see _some_ current state between invocation and completion; but not a state prior to the read. It also prohibits _non-monotonic reads_–in which one reads a new value, then an old one.

Because of these strong constraints, linearizable systems are easier to reason about–which is why they’re chosen as the basis for many concurrent programming constructs. All variables in Javascript are (independently) linearizable; as are volatile variables in Java, atoms in Clojure, or individual processes in Erlang. Most languages have mutexes and semaphores; these are linearizable too. Strong assumptions yield strong guarantees.

But what happens if we can’t satisfy those assumptions?

## Sequential consistency

![sequential-history.jpg](https://aphyr.com/data/posts/313/sequential-history.jpg "sequential-history.jpg")

If we allow processes to skew in time, such that their operations can take effect _before_ invocation, or _after_ completion–but retain the constraint that operations from any given process must take place in that process’ order–we get a weaker flavor of consistency: _sequential consistency_.

Sequential consistency allows more histories than linearizability–but it’s still a useful model: one that we use every day. When a user uploads a video to YouTube, for instance, YouTube puts that video into a _queue_ for processing, then returns a web page for the video right away. We can’t actually _watch_ the video at that point; the video upload _takes effect_ a few minutes later, when it’s been fully processed. Queues remove _synchronous_ behavior while (depending on the queue) preserving order.

Many caches also behave like sequentially consistent systems. If I write a tweet on Twitter, or post to Facebook, it takes time to percolate through layers of caching systems. Different users will see my message at different times–but each user will see my operations _in order_. Once seen, a post shouldn’t disappear. If I write multiple comments, they’ll become visible sequentially, not out of order.

## Causal consistency

We don’t have to enforce the order of _every_ operation from a process. Perhaps, only _causally related_ operations must occur in order. We might say, for instance, that all comments on a blog post must appear in the same order for everyone, and insist that any _reply_ be visible to a process _only after the post it replies to_ is visible. If we encode those causal relationships like “I depend on operation X” as an explicit part of each operation, the database can delay making operations visible until it has all the operation’s dependencies.

This is weaker than ordering every operation from the same process–operations from the same process with independent causal chains could execute in any relative order–but prevents many unintuitive behaviors.

## Serializable consistency

![serializable-history.jpg](https://aphyr.com/data/posts/313/serializable-history.jpg "serializable-history.jpg")

If we say that the history of operations is equivalent to one that took place in some single atomic order–but say nothing about the invocation and completion times–we obtain a consistency model known as _serializability_. This model is both much stronger and much weaker than you’d expect.

Serializability is _weak_, in the sense that it permits many types of histories, because it places no bounds on time or order. In the diagram to the right, it’s as if messages could be sent arbitrarily far into the past or future, that causal lines are allowed to cross. In a serializable database, a transaction like `read x` is always allowed to execute at time 0, when `x` had not yet been initialized. Or it might be delayed infinitely far into the future! The transaction `write 2 to x` could execute right now, _or_ it could be delayed until the end of time, never appearing to occur.

For instance, in a serializable system, the program

`x = 1 x = x + 1 puts x`

is allowed to print `nil`, `1`, _or_ `2`; because the operations can take place in any order. This is a surprisingly weak constraint! Here, we assume that each line represents a single operation and that all operations succeed.

On the other hand, serializability is _strong_, in the sense that it prohibits large classes of histories, because it demands a linear order. The program

`print x if x = 3 x = 1 if x = nil x = 2 if x = 1 x = 3 if x = 2`

can only be ordered in one way. It doesn’t happen in the same order we _wrote_, but it will reliably change `x` from `nil` -> `1` -> `2` -> `3`, and finally print `3`.

Because serializability allows arbitrary reordering of operations (so long as the order appears atomic), it is not particularly useful in real applications. Most databases which claim to provide serializability actually provide _strong serializability_, which has the same time bounds as linearizability. To complicate matters further, what most SQL databases term the SERIALIZABLE consistency level [actually means something weaker](http://www.bailis.org/papers/hat-hotos2013.pdf), like repeatable read, cursor stability, or snapshot isolation.

## Consistency comes with costs

We’ve said that “weak” consistency models _allow more histories_ than “strong” consistency models. Linearizability, for example, guarantees that operations take place between the invocation and completion times. However, _imposing order requires coordination_. Speaking loosely, the more histories we exclude, the more careful and communicative the participants in a system must be.

You may have heard of the [CAP theorem](http://lpd.epfl.ch/sgilbert/pubs/BrewersConjecture-SigAct.pdf), which states that given consistency, availability, and partition tolerance, any given system may guarantee _at most_ two of those properties. While Eric Brewer’s CAP conjecture was phrased in these informal terms, the CAP _theorem_ has very precise definitions:

1.  Consistency means linearizability, and in particular, a linearizable register. Registers are equivalent to other systems, including sets, lists, maps, relational databases, and so on, so the theorem can be extended to cover _all kinds_ of linearizable systems.

2.  Availability means that every request to a non-failing node must complete successfully. Since network partitions are allowed to last _arbitrarily long_, this means that nodes cannot simply defer responding until after the partition heals.

3.  Partition tolerance means that partitions _can happen_. Providing consistency and availability when the network is reliable is _easy_. Providing both when the network is not reliable is _provably impossible_. If your network is _not_ perfectly reliable–and it isn’t–you cannot choose CA. This means that all practical distributed systems on commodity hardware can guarantee, at maximum, either AP or CP.

![family-tree.jpg](https://aphyr.com/data/posts/313/family-tree.jpg "family-tree.jpg")

“Hang on!” you might exclaim. “Linearizability is not the end-all-be-all of consistency models! I could work around the CAP theorem by providing _sequential_ consistency, or serializability, or snapshot isolation!”

This is true; the CAP theorem only says that we cannot build totally available linearizable systems. The problem is that we have _other_ proofs which tell us that you cannot build totally available systems with sequential, serializable, repeatable read, snapshot isolation, or cursor stability–or any models stronger than those. In this map from Peter Bailis’ [Highly Available Transactions paper](http://www.vldb.org/pvldb/vol7/p181-bailis.pdf), models shaded in red _cannot be fully available_.

If we _relax_ our notion of availability, such that client nodes must always talk to the same server, _some_ types of consistency become achievable. We can provide causal consistency, PRAM, and read-your-writes consistency.

If we demand _total_ availability, then we can provide monotonic reads, monotonic writes, read committed, monotonic atomic view, and so on. These are the consistency models provided by distributed stores like Riak and Cassandra, or ANSI SQL databases on the lower isolation settings. These consistency models don’t have linear orders like the diagrams we’ve drawn before; instead, they provide _partial_ orders which come together in a patchwork or web. The orders are _partial_ because they admit a broader class of histories.

## A hybrid approach

![weak-not-unsafe.jpg](https://aphyr.com/data/posts/313/weak-not-unsafe.jpg "weak-not-unsafe.jpg")

Some algorithms depend on linearizability for safety. If we want to build a distributed lock service, for instance, linearizability is _required_; without hard time boundaries, we could hold a lock from the future or from the past. On the other hand, many algorithms _don’t_ need linearizability. Eventually consistent sets, lists, trees, and maps, for instance, can be safely expressed as [CRDTs](http://hal.upmc.fr/docs/00/55/55/88/PDF/techreport.pdf) even in “weak” consistency models.

Stronger consistency models also tend to require more coordination–more messages back and forth–to ensure their operations occur in the correct order. Not only are they less available, but they can also _impose higher latency constraints_. This is why modern CPU memory models are _not_ linearizable by default–unless you explicitly say so, modern CPUs will reorder memory operations relative to other cores, or worse. While more difficult to reason about, the performance benefits are phenomenal. Geographically distributed systems, with hundreds of milliseconds of latency between datacenters, often make similar tradeoffs.

So in practice, we use _hybrid_ data storage, mixing databases with varying consistency models to achieve our redundancy, availability, performance, and safety objectives. “Weaker” consistency models wherever possible, for availability and performance. “Stronger” consistency models where necessary, because the algorithm being expressed demands a stricter ordering of operations. You can write huge volumes of data to S3, Riak or Cassandra, for instance, then write a _pointer_ to that data, linearizably, to Postgres, Zookeeper or Etcd. Some databases admit multiple consistency models, like tunable isolation levels in relational databases, or Cassandra and Riak’s linearizable transactions, which can help cut down on the number of systems in play. Bottom line, though: anyone who says their consistency model is the only right choice is likely trying to sell something. You can’t have your cake and eat it too.

Armed with a more nuanced understanding of consistency models, I’d like to talk about how we go about _verifying_ the correctness of a linearizable system. In the next Jepsen post, we’ll discuss the linearizability checker I’ve built for testing distributed systems: [Knossos](http://aphyr.com/posts/314-computational-techniques-in-knossos).

For a more formal definition of these models, try Dziuma, Fatourou, and Kanellou’s [Survey on consistency conditions](http://www.ics.forth.gr/tech-reports/2013/2013.TR439_Survey_on_Consistency_Conditions.pdf)

## [译文](https://zhuanlan.zhihu.com/p/48782892)
