# Java Memory Model Pragmatics (transcript)

Table of Contents

*   [Preface](https://shipilev.net/blog/2014/jmm-pragmatics/#_preface)
*   [Intro](https://shipilev.net/blog/2014/jmm-pragmatics/#_intro)
*   [Part I. Access Atomicity](https://shipilev.net/blog/2014/jmm-pragmatics/#_part_i_access_atomicity)
    *   [What Do We Want](https://shipilev.net/blog/2014/jmm-pragmatics/#_what_do_we_want)
    *   [What Do We Have](https://shipilev.net/blog/2014/jmm-pragmatics/#_what_do_we_have)
    *   [Test Your Understanding](https://shipilev.net/blog/2014/jmm-pragmatics/#_test_your_understanding)
    *   [Value Types and C/C++](https://shipilev.net/blog/2014/jmm-pragmatics/#_value_types_and_c_c)
    *   [JMM Updates](https://shipilev.net/blog/2014/jmm-pragmatics/#_jmm_updates)
*   [Part II. Word Tearing](https://shipilev.net/blog/2014/jmm-pragmatics/#_part_ii_word_tearing)
    *   [What Do We Want](https://shipilev.net/blog/2014/jmm-pragmatics/#_what_do_we_want_2)
    *   [What Do We Have](https://shipilev.net/blog/2014/jmm-pragmatics/#_what_do_we_have_2)
    *   [Test Your Understanding](https://shipilev.net/blog/2014/jmm-pragmatics/#_test_your_understanding_2)
    *   [Layout Control and C/C++](https://shipilev.net/blog/2014/jmm-pragmatics/#_layout_control_and_c_c)
    *   [JMM Updates](https://shipilev.net/blog/2014/jmm-pragmatics/#_jmm_updates_2)
*   [Part III: SC-DRF](https://shipilev.net/blog/2014/jmm-pragmatics/#_part_iii_sc_drf)
    *   [What Do We Want](https://shipilev.net/blog/2014/jmm-pragmatics/#_what_do_we_want_3)
    *   [What Do We Have](https://shipilev.net/blog/2014/jmm-pragmatics/#_what_do_we_have_3)
    *   [Java Memory Model](https://shipilev.net/blog/2014/jmm-pragmatics/#_java_memory_model)
        *   [Program Order](https://shipilev.net/blog/2014/jmm-pragmatics/#_program_order)
        *   [Synchronization Order](https://shipilev.net/blog/2014/jmm-pragmatics/#_synchronization_order)
        *   [Happens-Before](https://shipilev.net/blog/2014/jmm-pragmatics/#_happens_before)
        *   [Happens-Before: Publication](https://shipilev.net/blog/2014/jmm-pragmatics/#_happens_before_publication)
    *   [Happens-Before: Test Your Understanding](https://shipilev.net/blog/2014/jmm-pragmatics/#_happens_before_test_your_understanding)
    *   [JMM Interpretation: Roach Motel](https://shipilev.net/blog/2014/jmm-pragmatics/#_jmm_interpretation_roach_motel)
    *   [Test Your Understanding](https://shipilev.net/blog/2014/jmm-pragmatics/#_test_your_understanding_3)
    *   [Benchmarks](https://shipilev.net/blog/2014/jmm-pragmatics/#_benchmarks)
    *   [JMM Updates](https://shipilev.net/blog/2014/jmm-pragmatics/#_jmm_updates_3)
*   [Part IV: Out of Thin Air](https://shipilev.net/blog/2014/jmm-pragmatics/#_part_iv_out_of_thin_air)
    *   [What Do We Want](https://shipilev.net/blog/2014/jmm-pragmatics/#_what_do_we_want_4)
    *   [What Do We Have](https://shipilev.net/blog/2014/jmm-pragmatics/#_what_do_we_have_4)
    *   [OoTA and C/C++](https://shipilev.net/blog/2014/jmm-pragmatics/#_oota_and_c_c)
    *   [JMM Updates](https://shipilev.net/blog/2014/jmm-pragmatics/#_jmm_updates_4)
*   [Part V: Finals](https://shipilev.net/blog/2014/jmm-pragmatics/#_part_v_finals)
    *   [Test Your Basic Understanding](https://shipilev.net/blog/2014/jmm-pragmatics/#_test_your_basic_understanding)
    *   [What Do We Want](https://shipilev.net/blog/2014/jmm-pragmatics/#_what_do_we_want_5)
    *   [What Do We Have](https://shipilev.net/blog/2014/jmm-pragmatics/#_what_do_we_have_5)
    *   [Constructive Example](https://shipilev.net/blog/2014/jmm-pragmatics/#_constructive_example)
    *   [Premature Publication](https://shipilev.net/blog/2014/jmm-pragmatics/#_premature_publication)
    *   [Test Your Understanding (tricky)](https://shipilev.net/blog/2014/jmm-pragmatics/#_test_your_understanding_tricky)
    *   [JMM Updates](https://shipilev.net/blog/2014/jmm-pragmatics/#_jmm_updates_5)
    *   [Benchmarks](https://shipilev.net/blog/2014/jmm-pragmatics/#_benchmarks_2)
*   [Conclusion](https://shipilev.net/blog/2014/jmm-pragmatics/#_conclusion)

Aleksey Shipilёv, [@shipilev](http://twitter.com/shipilev), [aleksey@shipilev.net](mailto:aleksey@shipilev.net)

| 

CAUTION

 | This is a very long transcript for a very long talk. The talk’s running time is close to 120-150 minutes, and transcript may take even longer if digested thoughtfully. Please plan your reading accordingly. There are five large sections which are mostly independent of each other, that should help to split the reading. |

| 

WARNING

 | The talk is correct to the best of my knowledge, but parts of it may still be incorrect, misguided, or sometimes wrong. The only true source of truth is [The Java Language Specification](http://docs.oracle.com/javase/specs/) itself. My understanding of JMM was proven incorrect more than once over the previous five years. If you see something wrong in the post, don’t hesitate to drop me a note. |

| 

NOTE

 | This post is also available in [ePUB](https://shipilev.net/blog/2014/jmm-pragmatics/article.epub) and [mobi](https://shipilev.net/blog/2014/jmm-pragmatics/article.mobi). |

## Preface

The Java Memory Model is the most complicated part of Java spec that must be understood by at least library and runtime developers. Unfortunately, it is worded in such a way that it takes a few senior guys to decipher it for each other. Most developers, of course, are not using JMM rules as stated, and instead make a few constructions out of its rules, or worse, blindly copy the constructions from senior developers without understanding the limits of their applicability. If you are an ordinary guy who is not into hardcore concurrency, you can pass this post, and read high-level books, like ["Java Concurrency in Practice"](http://www.amazon.com/Java-Concurrency-Practice-Brian-Goetz/dp/0321349601). If you are one of the senior folks interested in how all this works, read on!

This post is a transcript of the ["Java Memory Model Pragmatics"](http://shipilev.net/#jmm) talk I gave during this year at different conferences, mostly in Russian. There seems to be a limited supply of conferences in the world which can accommodate such a long talk, and being in need for exposing some background reading for my JMM Workshop at JVMLS this year, I decided to transcribe it.

We will reuse a lot of the slides, and I’ll try to build the narrative based on them. Sometimes I’ll just skip over without narrative when the slides are self-explanatory. The slides are available in [Russian](http://shipilev.net/talks/narnia-2555-jmm-pragmatics-ru.pdf) and [English](http://shipilev.net/talks/narnia-2555-jmm-pragmatics-en.pdf). The slides below are rasterized, but have a nice native resolution. Zoom-in if they are unreadable. Sane browsers smartly resize the images, and more details are visible when zoomed in. (I would make the illustrations in SVG, but my iPad crashes when trying to render 150+ of them on one page!)

I would like to thank [Brian Goetz](https://twitter.com/BrianGoetz), Doug Lea, David Holmes, [Sergey Kuksenko](https://twitter.com/kuksenk0), Dmitry Chyuko, [Mark Cooper](https://twitter.com/AstragaliUSA), [C. Scott Andreas](https://twitter.com/cscotta), [Joe Kearney](https://twitter.com/joejkearney) and many others for helpful comments and corrections. The example section on final fields contains the info untangled by [Vladimir Sitnikov](https://twitter.com/VladimirSitnikv) and Valentin Kovalenko, and is the excerpt from their larger talk on [Final Fields Semantics](http://www.slideshare.net/VladimirSitnikv/final-field-semantics).

## Intro

![page 004](https://shipilev.net/blog/2014/jmm-pragmatics/page-004.png)

First, a simple detector slide. Hey there, [@gakesson](https://twitter.com/gakesson), **waves**!

* * *

![page 005](https://shipilev.net/blog/2014/jmm-pragmatics/page-005.png)

If you read just about any language spec, you will notice it can be logically divided into two related, but distinct parts. First, a very easy part, is the language _syntax_, which describes how to write programs in the language. Second, the largest part, is the language _semantics_, which describes exactly what a particular syntax construct _means_. Language specs usually describe the semantics via the behavior of an abstract machine which executes the program, so the language spec in this manner is just an abstract machine spec.

* * *

![page 007](https://shipilev.net/blog/2014/jmm-pragmatics/page-007.png)

When your language has storage (in the form of variables, heap memory, etc.), the abstract machine also has storage, and you have to define a set of rules concerning how that storage behaves. That’s what we call a _memory model_. If your language does not have explicit storage (e.g. you pass the data around in call contexts), then your memory model is darn simple. In storage-savvy languages, the memory models appear to answer a simple question: "What values can a particular read observe?"

* * *

![page 008](https://shipilev.net/blog/2014/jmm-pragmatics/page-008.png)

In sequential programs, that seems a vacuous question to ask: since you have the sequential program, the stores into memory are coming in some given order, and it is obvious that the reads should observe the latest writes in that order. That is why people usually meet with memory models only for multi-threaded programs, where this question becomes complicated. However, memory models matter even in the sequential cases (although they are often cleverly disguised in the notion of _evaluation order_).

* * *

![page 009](https://shipilev.net/blog/2014/jmm-pragmatics/page-009.png)

For example, the infamous example of undefined behavior in a C program that packs a few increments between the sequence points. This program can satisfy the given assert, but can also fail it, or otherwise summon [nasal demons](http://www.catb.org/jargon/html/N/nasal-demons.html). One could argue that the result of this program can be different because the evaluation order of increments is different, but it would not explain, e.g. the result of 12, when neither increment saw the written value from the other. This is the memory model concern: what value should each increment see (and by extension, what it would store).

* * *

![page 010](https://shipilev.net/blog/2014/jmm-pragmatics/page-010.png)

Either way, when presented with a challenge of implementing the particular language, we can go one of two ways: interpretation, or compilation of abstract machine to the target hardware. Both interpretation and compilation are connected via [Futamura Projections](http://en.wikipedia.org/wiki/Partial_evaluation#Futamura_projections) anyway.

The practical takeaway is that both interpreter and compiler are tasked with emulating the abstract machine. Compilers are usually blamed for screwing up the memory models and multi-threaded programs, but interpreters are not immune, either. Failing to run an interpreter to the abstract machine spec may result in memory model violations. The simplest example: cache the field values over volatile reads in an interpreter, and you are done for. This takes us to an interesting trade-off.

* * *

![page 011](https://shipilev.net/blog/2014/jmm-pragmatics/page-011.png)

The very reason why programming languages still require smart developers is the absence of hypersmart compilers. "Hyper" is not a overstatement: some of the problems in compiler engineering are undecidable, that is, non-solvable even **in theory**, let alone in practice. Other interesting problems may be theoretically feasible, but not practical. Therefore, to make practical (optimizing) compilers possible, we need to cause some inconvenience in the language. The same goes for hardware, since (at least for Turing machines) it is just the algorithms _in silica_.

* * *

![page 012](https://shipilev.net/blog/2014/jmm-pragmatics/page-012.png)

To elaborate on this thought, the rest of the talk is structured as follows.

* * *

## Part I. Access Atomicity

### What Do We Want

![page 014](https://shipilev.net/blog/2014/jmm-pragmatics/page-014.png)

The simplest thing to understand in JMM is the access atomicity guarantee. To specify this more or less rigorously, we need to introduce a bit of notation. In the example on this slide, you can see the table with two columns. This notation reads as follows. Everything in the header happened already: all variables are defined, all initializing stores committed, etc. The columns are different threads. In this example, Thread 1 stores some value `V2` into global variable `t`. Thread 2 reads the variable, and asserts the read value. Here, we want to make sure the reading thread only observes the known value, not some value in between.

* * *

### What Do We Have

![page 015](https://shipilev.net/blog/2014/jmm-pragmatics/page-015.png)

This seems to be a very obvious requirement for a sane programming language: how you can possibly violate this, and why? Here is why.

To maintain atomicity under concurrent accesses, you have to at least have the machine instructions operating with the operands of given width, otherwise the atomicity is broken on instruction level: if you need to split the accesses into several sub-accesses, they can interleave. But even if you have the desired-width instructions, they still can be non-atomic: for example, the atomicity guarantees for 2- and 4-byte reads are unknown for PowerPC (they are implied to be atomic).

* * *

![page 016](https://shipilev.net/blog/2014/jmm-pragmatics/page-016.png)

Most platforms, however, do guarantee atomicity for up to 32-bit accesses. This is why we have a **compromise** in JMM which relaxes the atomicity guarantees for 64-bit values. Of course, there are still ways to enforce atomicity for 64-bit values, e.g. by pessimistically acquiring the lock on update and read, but that will come at a cost, and so we provide an escape hatch: users put _volatile_ where they need atomicity, and VM and hardware work together to preserve it, no matter what the costs are.

* * *

![page 017](https://shipilev.net/blog/2014/jmm-pragmatics/page-017.png)

On most hardware it is not enough, however, to have the desired-width operations to maintain atomicity. For example, if data access causes multiple transactions to memory, the atomicity is off, even though we executed a single access instruction. In x86, for example, the atomicity is not guaranteed if the read/write spans two cache lines, since it requires two memory transactions. This is why generally only the aligned reads/writes are atomic, which forces VMs to align the data.

In this example, which is printed by [JOL](http://openjdk.java.net/projects/code-tools/jol/), we can see the _long_ field being allocated at offset 16 from the object start. Coupled with object alignment of 8 bytes, we have the perfectly aligned _long_. Now, it would not violate the memory model to put _long_ at offset 12, if we know it is not _volatile_, but that will only work on x86 (other platforms may violently disagree on performing misaligned accesses), and possibly with performance disadvantages.

* * *

### Test Your Understanding

![page 019](https://shipilev.net/blog/2014/jmm-pragmatics/page-019.png)

Let’s test our understanding with a simple quiz. Setting `-1L` is equivalent to setting all the bits to `1` in _long_.

Answer (select over to reveal): No magic is involved; a _volatile long_ field inside AtomicLong guarantees this. This is required by the language spec, and no special treatment for AtomicLong from the VM side is needed for this sample to work.

* * *

### Value Types and C/C++

![page 022](https://shipilev.net/blog/2014/jmm-pragmatics/page-022.png)

In Java, we are "lucky" to have the built-in types of small widths. In other languages which provide _value types_, the type width is arbitrary, which presents interesting challenges for the memory model.

In this example, C++ follows C compatibility by supporting structs. C++11 additionally supports _std::atomic_, which requires access atomicity for every Plain Old Data (POD) type T. So, if we do a trick like this in C++11, the implementations are forced to deal with atomically writing and reading the 104-byte memory blocks. There are no machine instructions which can guarantee atomicity at these widths, so implementation should resort to either CAS-ing, or locking, or something else.

(It gets even **more** interesting since C++ allows separate compilation: now the **linker** is tasked with the job of figuring out what locks/CAS-guards are used by this particular _std::atomic_. I am not completely sure what happens if threads execute the code generated by different compilers in the example above.)

* * *

### JMM Updates

This section covers the atomicity considerations for the updated Java Memory Model. See a more-thorough explanation [in a separate post](http://shipilev.net/blog/2014/all-accesses-are-atomic/).

![page 023](https://shipilev.net/blog/2014/jmm-pragmatics/page-023.png)

In 2014, do we want to reconsider the 64-bit exception? There are few use cases when racy updates to _long_ and _double_ make sense, e.g. in scalable probabilistic counters. Developers may reasonably hope the _long_/_double_ accesses are atomic on 64-bit platforms, but they nevertheless require _volatile_ to be portable if the code is accidentally run on 32-bit platforms. Marking fields _volatile_ will pay the cost of memory barriers.

In other words, since _volatile_ is overloaded with two meanings: a) access atomicity; and b) memory ordering — you cannot get one without getting the other as baggage. One can speculate on the costs of removing the 64-bit exception. Since VMs are handling access atomicity separately by emitting special instruction sequences, we can hack the VM into unconditionally emitting atomic instruction sequences when required.

* * *

![page 024](https://shipilev.net/blog/2014/jmm-pragmatics/page-024.png)

It takes some time to understand this chart. We can measure reads and writes of _longs_ — three times for each access mode (plain, volatile, and via Unsafe.putOrdered). If we are implementing the feature correctly, there should be no difference on 64-bit platforms, since the accesses are already atomic. Indeed there is no difference between the colored bars on 64-bit Ivy Bridge.

Notice how heavyweight a _volatile long_ write can be. If I only wanted atomicity, I pay this cost for memory ordering.

* * *

![page 025](https://shipilev.net/blog/2014/jmm-pragmatics/page-025.png)

It gets more complicated when dealing with 32-bit platforms. There, you will need to inject special instruction sequences to get the atomicity. In the case of x86, FPU load/stores are 64-bit wide even in 32-bit platforms. You pay the cost of "redundant" copies, but not that much.

* * *

![page 026](https://shipilev.net/blog/2014/jmm-pragmatics/page-026.png)

On non-x86 platforms, we also have to use alternative instruction sequences to regain atomicity, with predictable performance impact. Note that in this case, as well in the 32-bit x86 case, _volatile_ is a bit slower with enforced atomicity, but that’s a systematic error since we need to also dump the values into a long field to prevent some compiler optimizations.

* * *

## Part II. Word Tearing

### What Do We Want

![page 028](https://shipilev.net/blog/2014/jmm-pragmatics/page-028.png)

Word tearing is related to access atomicity.

If two variables are distinct, then the actions on them should also be distinct, and should not be affected by the actions on adjacent elements. How can this example break? Quite simple: if our hardware cannot access a distinct array element, it will be forced to read several elements, modify one element in the bunch, and then put the entire bunch back.

If two threads are doing the same dance on their separate elements, it might happen that another thread stores its own steps back to memory, overwriting an element updated by the first thread. This may and will cause lots of headaches for unsuspecting users, because without the clear provisions in the language spec, runtimes are free to apply transformations that can lead to hard-to-diagnose bugs.

* * *

### What Do We Have

![page 029](https://shipilev.net/blog/2014/jmm-pragmatics/page-029.png)

If we want to prohibit word tearing, we need hardware support for accesses of a given width. In the most simple scenario of a _boolean[]_ array or a group of _boolean_ fields, you can’t readily access a single memory bit on most hardware, since the lower addressability bound is usually a single byte.

* * *

![page 030](https://shipilev.net/blog/2014/jmm-pragmatics/page-030.png)

Remarkably, you have to explain word tearing to programmers these days. Most of the systems programmers from days before are intimately familiar with it, and do understand the horror of chasing such a bug in a real system.

Therefore, Java, shooting to be a sane language, forbids word tearing. Period. Bill Pugh (FindBugs is his most attributed baby, but he was also the lead for JMM JSR 133) was [quite articulate](http://www.cs.umd.edu/~pugh/java/memoryModel/archive/0978.html) about that. I was chasing a word-tearing bug in a C++ program once — **NOT FUN**.

This requirement seems rather easy to fit with current hardware: the only data type you may care about is _boolean_, which you would probably want to take a full byte instead of single bits. Of course, you also need to tame any compiler optimizations which may buffer reads and writes along with the adjacent data.

* * *

![page 031](https://shipilev.net/blog/2014/jmm-pragmatics/page-031.png)

Most people look to docs for an allowed range of primitive values and infer the machine-representation widths from there. You can only imply the minimum machine width to represent, say 2^64 cases for _long_. It does not oblige a runtime to actually allocate 8 bytes per _long_; it could, in principle, use 128-byte _longs_, as long as it’s practical for some weird reason.

However, most runtimes I know of are practical, and the machine representation widths closely fit the value domains, wasting no space. _boolean_, as I said before, is the only exception from this rule. JOL tries to figure out the actual machine widths, and you can see the scales on this slide. The numbers are the bytes taken by _reference_, _boolean_, _byte_, _short_, _char_, _int_, _float_, _long_, and _double_, respectively — exactly what we would expect them to be. Other platforms may be perceived…​ [strange](http://parleys.com/play/5298f999e4b039ad2298c9e3/chapter27/agenda#).

* * *

### Test Your Understanding

![page 032](https://shipilev.net/blog/2014/jmm-pragmatics/page-032.png)

Answer (select over to reveal): Any of _(true, true)_, _(false, true)_, _(true, false)_, because BitSet stores the bits densely in long[] arrays and uses the bit magic to access a particular bit. Winning greatly in memory footprint, it breaks away from word-tearing guarantees of the language. (BitSet Javadocs say multi-threaded usages should be synchronized, so this is arguably an artificial example)

* * *

### Layout Control and C/C++

![page 036](https://shipilev.net/blog/2014/jmm-pragmatics/page-036.png)

Quite a few people want to control the memory layout for the particular class for a better footprint in marginal cases, and/or better performance. But in a language that allows an arbitrary layout for its variables, you cannot consistently forbid word tearing, because you would have to pay the price, as in this example.

There are no machine instructions that can write 7 bits at once, or read only 3 bits at once, so implementations would need to get creative if they are tasked with avoiding word tearing. C/C++11 allow you to use this sharp tool, but tell you that once you start, you are on your own.

* * *

### JMM Updates

![page 037](https://shipilev.net/blog/2014/jmm-pragmatics/page-037.png)

Nobody disputes that word tearing should remain forbidden.

* * *

## Part III: SC-DRF

### What Do We Want

![page 039](https://shipilev.net/blog/2014/jmm-pragmatics/page-039.png)

Now we are getting to the most interesting part of a memory model: reasoning about program reads at large. It would be natural to think that programs are executing their statements in some global order, sometimes switching between the threads. This is a really simple model, and Lamport has defined it for us already: sequential consistency.

* * *

![page 041](https://shipilev.net/blog/2014/jmm-pragmatics/page-041.png)

Notice the highlight. Sequential consistency does not mean the operations were executed in a particular total order! (The stronger [strict consistency](http://en.wikipedia.org/wiki/Linearizability) provides that). It is only important that the result is **indistinguishable** from some execution which has the total order of operations. We call executions like these _sequentially consistent executions_, and their results are called _sequentially consistent results_.

* * *

![page 042](https://shipilev.net/blog/2014/jmm-pragmatics/page-042.png)

SC apparently gives us the opportunity to optimize the code. Since we are not bounded by _actual_ total execution order, but only have to _pretend_ to have one, we can have funny optimizations. For example, this program transformation does not break SC: there is obviously a SC execution of the original program which yields the same result (assuming nobody cares about the values of `a` and `b` anymore).

Notice that SC allows us to _shrink_ the set of possible executions. At the extreme, we are free to choose a single order and stick to it.

* * *

### What Do We Have

![page 043](https://shipilev.net/blog/2014/jmm-pragmatics/page-043.png)

However, the optimizability under SC is overrated. Notice that current optimizing compilers, not to mention hardware, only care about the current instruction stream. So, if we have two reads in the instruction stream, can we reorder them as in this example and maintain SC?

* * *

![page 044](https://shipilev.net/blog/2014/jmm-pragmatics/page-044.png)

Turns out, you can’t. If another part of a program stores the values into `a` and `b`, then read reordering breaks SC. Indeed, the original program executing under SC can only have the results matching `(*, 2)` or `(0, *)`, but the modified program, even executed in total order manner, yields `(1, 0)`, baffling developers expecting SC from their code.

* * *

![page 045](https://shipilev.net/blog/2014/jmm-pragmatics/page-045.png)

You see, then, to figure out whether even a very simple transformation is plausible, you need sophisticated analysis, which does not readily scale to realistic programs. In theory, we can have a smart global optimizer (GMO) that can perform this analysis. I think the existence of a GMO is closely tied to the existence of Laplace’s Demon :)

But since we don’t have a GMO, all optimizations are conservatively forbidden for fear of inadvertently violating SC, and that costs performance. So what? We can go without the transformation, right? Unlikely: even the very basic transformations would be forbidden. Think about it: can you put a variable on register if that effectively eliminates the reads elsewhere in the program, i.e. does reordering?

* * *

![page 047](https://shipilev.net/blog/2014/jmm-pragmatics/page-047.png)

…​and while we can forbid some of the optimizations in compilers to stop wreaking havoc in otherwise SC programs, hardware cannot be easily negotiated with. Hardware already reorders lots of stuff, and provides the costly escape hatches to intimidate reorderings (_"memory barriers"_). Therefore, a model which does not control what transformations are possible and what optimizations are encouraged would not realistically run with decent performance. For example, if we are to require sequential consistency in the language, we will probably have to pessimistically emit memory barriers around almost every single memory access, in order to slay hardware attempts at "optimizing".

* * *

![page 048](https://shipilev.net/blog/2014/jmm-pragmatics/page-048.png)

Moreover, if your program contains **races**, current hardware does not guarantee any particular outcome from those conflicting operations. Hans Boehm and Sarita Adve stand firm on [this](http://soft.vub.ac.be/races/wp-content/uploads/2012/09/races2012_submission_3.pdf).

* * *

![page 049](https://shipilev.net/blog/2014/jmm-pragmatics/page-049.png)

Therefore, to accommodate the reality into the model with plausible performance, we need to weaken it.

* * *

### Java Memory Model

![page 050](https://shipilev.net/blog/2014/jmm-pragmatics/page-050.png)

This is where things get significantly more complicated. Since the language spec should cover **all** possible programs expressible in the language, we can’t really provide a finite number of constructions which are guaranteed to work: their union will leave white spots in semantics, and white spots are bad.

Therefore, the JMM tries to cover all possible programs at once. It does so by describing the _actions_ which an abstract program can perform, and those actions describe what outcomes can be produced when executing a program. Actions are bound together in _executions_, which combine actions with additional orders describing the action relationships. This feels very ivory-tower-esque, so let’s get to the example right away.

* * *

#### Program Order

![page 051](https://shipilev.net/blog/2014/jmm-pragmatics/page-051.png)

The very first order is _Program Order (PO)_. It orders the actions in a single thread. Notice the original program, and one of the possible executions of this program. There, the program can read `1` from `x`, fall through to the _else_ branch, store `1` to `z`, and then go on to read something from `y`.

* * *

![page 052](https://shipilev.net/blog/2014/jmm-pragmatics/page-052.png)

Program order is total (within one thread), i.e. each pair of actions is related by this order. It is important to understand a few things.

The actions linked together in program order do not preclude being "reordered". In fact, it is a bit confusing to talk about reordering of actions, because one probably intends to speak of statement reordering in a program, which generates _new_ executions. It will then be an open question whether the _executions_ generated by this new program violate the provisions for JMM.

The program order does not, and I repeat, does not provide the ordering guarantees. The only reason it exists is to provide the link between possible executions and the original program.

* * *

![page 053](https://shipilev.net/blog/2014/jmm-pragmatics/page-053.png)

This is what we mean. Given the simple schematics of actions and executions, you can construct an infinite number of executions. These executions are detached from any reality; they are just the "primordial soup", containing everything possible by construction. Somewhere in this soup float the executions which can explain a particular outcome of the given program, and the set of all such plausible executions cover all plausible outcomes of the program.

* * *

![page 054](https://shipilev.net/blog/2014/jmm-pragmatics/page-054.png)

Here is where Program Order (PO) jumps in. To filter out the executions we can take to reason about the particular program, we have **intra-thread consistency** rules, which eliminate all unrelated executions. For instance, in the example above, while the illustrated execution is abstractly possible, it does not relate to the original program: after reading `2` from `x`, we should have written `1` to `y`, not to `z`.

* * *

![page 055](https://shipilev.net/blog/2014/jmm-pragmatics/page-055.png)

Here is how we can illustrate this filtering. Intra-thread consistency is the very first execution filter, which most people do implicitly in their heads when dealing with JMM. You may notice at this point that JMM is a non-constructive model: we don’t build up the solution inductively, but rather take the entire realm of executions, and filter out those interesting to us.

* * *

#### Synchronization Order

![page 056](https://shipilev.net/blog/2014/jmm-pragmatics/page-056.png)

Now we begin to build the part of the model which really orders stuff. In weak memory models, we don’t order all the actions, we only impose a hard order on a few limited primitives. In the JMM, those primitives are wrapped in their respective _Synchronization Actions_.

* * *

![page 057](https://shipilev.net/blog/2014/jmm-pragmatics/page-057.png)

_Synchronization Order (SO)_ is a total order which spans all synchronization actions. But this is not the most interesting part about this order. The JMM provides two additional constraints: SO-PO consistency, and SO consistency. Let’s unpack these constraints using a trivial example.

* * *

![page 058](https://shipilev.net/blog/2014/jmm-pragmatics/page-058.png)

This is a rather simple example derived from the [Dekker Lock](http://en.wikipedia.org/wiki/Dekker%27s_algorithm). Try to think what outcomes are allowed and why. After that, we’ll move on to analyzing it with the JMM.

* * *

The slides below are self-explanatory, and we’ll just skip them over:

![page 059](https://shipilev.net/blog/2014/jmm-pragmatics/page-059.png)

![page 060](https://shipilev.net/blog/2014/jmm-pragmatics/page-060.png)

![page 061](https://shipilev.net/blog/2014/jmm-pragmatics/page-061.png)

![page 062](https://shipilev.net/blog/2014/jmm-pragmatics/page-062.png)

![page 063](https://shipilev.net/blog/2014/jmm-pragmatics/page-063.png)

![page 064](https://shipilev.net/blog/2014/jmm-pragmatics/page-064.png)

* * *

Now if we look at these rules more closely, we’ll notice an interesting property. SO-PO consistency tells us that the effects in SO are visible as if the actions are done in program order. SO consistency tells us to observe all the actions preceding in the SO, even those that happened in a different thread. It is as if SO-PO consistency tells us to follow the program, and SO consistency allows us to "switch between threads" with all effects trailing us. Mixed with the totality of SO, we arrive at an interesting rule:

* * *

![page 065](https://shipilev.net/blog/2014/jmm-pragmatics/page-065.png)

**Synchronization Actions are sequentially consistent.** In a program consisting of volatiles, we can reason about the outcomes without deep thinking. Since SAs are SC, we can construct all the action interleavings, and figure out the outcomes from there. Notice there is no "happens-before" yet; SO is enough to reason.

* * *

![page 067](https://shipilev.net/blog/2014/jmm-pragmatics/page-067.png)

IRIW is another good example of SO properties. Again, all operations yield synchronization actions. The outcomes may be generated by enumerating all the interleavings of program statements. Only a single quad is forbidden by that construction, as if we observed the writes of `x` and `y` in different orders in different threads.

The real takeaway was best summed up by Hans Boehm. If you take an arbitrary program, no matter how many races it contains, and sprinkle enough _volatile_-s around that program, it will eventually become sequentially consistent, i.e. all the outcomes of the program would be explained by some SC execution. This is because you will eventually hit a critical moment when all the important program actions turn into synchronization actions, and become totally ordered.

* * *

![page 068](https://shipilev.net/blog/2014/jmm-pragmatics/page-068.png)

To conclude with our Venn diagram, SO consistencies filter out the executions with broken synchronization "skeletons". The outcomes of all the remaining executions can be explained by program-order-consistent interleavings of synchronization actions.

* * *

#### Happens-Before

![page 069](https://shipilev.net/blog/2014/jmm-pragmatics/page-069.png)

While providing a good basis to reason about programs, SO is not enough to construct a practical weak model. Here is why.

* * *

![page 070](https://shipilev.net/blog/2014/jmm-pragmatics/page-070.png)

Let us analyze a simple case. Given all we learned so far about SO, do we know if `(1, 0)` outcome is allowed?

* * *

![page 072](https://shipilev.net/blog/2014/jmm-pragmatics/page-072.png)

Let’s see. Since SO only orders the actions over `g`, nothing prevents us from reading either 0 or 1 from `x`. Bad…​

* * *

![page 073](https://shipilev.net/blog/2014/jmm-pragmatics/page-073.png)

We need something to connect the thread states, something which will drag the non-SA values along. SO is not usable for that, because it is not clear when and how it drags the state along. So, we need a clear-cut suborder of SO which describes the data flow. We call this suborder _synchronizes-with order (SW)_.

* * *

![page 074](https://shipilev.net/blog/2014/jmm-pragmatics/page-074.png)

It is rather easy to construct SW. SW is a partial order, and it does not span all the pairs of synchronization actions. For example, even though two operations on `g` on this slide are in SO, they are not in SW.

* * *

![page 075](https://shipilev.net/blog/2014/jmm-pragmatics/page-075.png)

SW only pairs the specific actions which "see" each other. More formally, the volatile write to `g` synchronizes-with all subsequent reads from `g`. "Subsequent" is defined in terms of SO, and therefore because of SO consistency, the write of `1` only synchronizes-with with reads of `1`. In this example, we see the SW between two actions. This suborder gives us the "bridge" between the threads, but applying to synchronization actions. Let’s extend this to other actions.

* * *

![page 076](https://shipilev.net/blog/2014/jmm-pragmatics/page-076.png)

Intra-thread semantics are described by Program Order. Here it is.

* * *

![page 077](https://shipilev.net/blog/2014/jmm-pragmatics/page-077.png)

Now, if we construct the union of PO and SW orders, and then [transitively close](http://en.wikipedia.org/wiki/Transitive_closure) that union, we get the derived order: _Happens-Before (HB)_. HB in this sense acquires both inter-thread and intra-thread semantics. PO leaks the information about sequential actions within each thread into HB, and SW leaks when the state "synchronizes". HB is partial order, and allows for construction of equivalent executions with reordered actions.

* * *

![page 078](https://shipilev.net/blog/2014/jmm-pragmatics/page-078.png)

Happen-before comes with yet another consistency rule. Remember the SO consistency rule, which stated that synchronization actions should see the latest relevant write in SO. **Happens-before consistency** is similar in application to HB Order: it dictates what writes can be observed by a particular read.

* * *

![page 079](https://shipilev.net/blog/2014/jmm-pragmatics/page-079.png)

HB consistency is interesting in allowing _races_. When no races are present, we can only see the latest preceding write in HB. But if we have a write unordered in HB with respect to a given read, then we also can see that (racy) write. Let’s define it more rigorously.

* * *

![page 080](https://shipilev.net/blog/2014/jmm-pragmatics/page-080.png)

The first part is rather relaxing: we are allowed to observe the writes happened before us, or any other unordered write (that’s a _race_). This is a very important property of the model: we specifically **allow** races, because races happen in the real world. If we forbid races in the model, runtimes would have a hard time optimizing code because they would need to enforce order everywhere.

Notice how that disallows seeing writes ordered after the read in HB order.

* * *

![page 081](https://shipilev.net/blog/2014/jmm-pragmatics/page-081.png)

The second part puts additional constraint on seeing the preceding writes: we can only see the **latest** write in happens-before order. Any other write before that is invisible to us. Therefore, in the absence of races, we can **only** see the latest write in HB.

* * *

![page 082](https://shipilev.net/blog/2014/jmm-pragmatics/page-082.png)

The consequence of HB consistency is to filter yet another subset of executions which observe something we allow them to observe. HB extends over non-synchronized actions, and therefore lets the model embrace all actions in the executions.

* * *

![page 083](https://shipilev.net/blog/2014/jmm-pragmatics/page-083.png)

This is what SC-DRF is all about: if we have no races in the program — that is, all reads and writes are ordered either by SO or HB — then the outcome of this program can be explained by some sequentially consistent execution. There is a formal proof for SC-DRF properties, but we will use intuitive understanding as to why this should be true.

* * *

#### Happens-Before: Publication

![page 084](https://shipilev.net/blog/2014/jmm-pragmatics/page-084.png)

The examples above were rather highbrow, but that is how language spec is defined. Let’s look at the example to understand this more intuitively. Take the same code example, and analyze it with HB consistency rules.

* * *

![page 085](https://shipilev.net/blog/2014/jmm-pragmatics/page-085.png)

This execution is happens-before consistent: _read(x)_ observes the latest write in HB. The outcome `(1, 1)` is therefore plausible.

* * *

![page 086](https://shipilev.net/blog/2014/jmm-pragmatics/page-086.png)

This execution is happens-before consistent, as we read the default value of `x`. We had omitted the HB edge coming default initialization that synchronizes-with first action in the thread on this chart.

* * *

![page 087](https://shipilev.net/blog/2014/jmm-pragmatics/page-087.png)

Somewhat surprisingly, the execution with outcome `(0, 1)` is **also** happens-before consistent, even though there is no transitive HB between read and write of `x`. We just read the value via the race — remember the first part in HB consistency definition.

* * *

![page 088](https://shipilev.net/blog/2014/jmm-pragmatics/page-088.png)

And this execution fails to adhere to HB consistency, and therefore cannot be used to reason about the program outcomes. Therefore this outcome is impossible. Notice that we eliminated `(1, 0)` from the four of possible outcomes, and that effectively means that we are forced to observe `x` as `1`, if we observed `g` as `1`.

* * *

![page 091](https://shipilev.net/blog/2014/jmm-pragmatics/page-091.png)

It will hurt our brains to figure out HB orders for real programs, so instead we can derive some simple rules. The source of a synchronizes-with edge is called "release", and the destination is called "acquire". HB contains SW, and therefore, HB spanning different threads also starts at "release", and ends at "acquire".

_Release_ can be thought of as the operation which releases all the preceding state updates into the wild, and _acquire_ is the paired operation which receives those updates. So, after a successful _acquire_, we can see all the updates preceding the paired _release_.

Because of the constructions we laid out before, it only works if acquire/release happened on the same variable, and we actually saw the written value. The quiz below further explores this.

* * *

### Happens-Before: Test Your Understanding

![page 092](https://shipilev.net/blog/2014/jmm-pragmatics/page-092.png)

Let’s play a bit more realistically here. Suppose you have the wrapper class which stores ((mail)boxes) some value of type T. Obviously, you have the setter which takes the value, and the getter which returns it. In most programs, reads vastly outnumber writes (otherwise why are you storing the value?), so synchronized getters may become scalability bottlenecks.

* * *

![page 093](https://shipilev.net/blog/2014/jmm-pragmatics/page-093.png)

People come with their profilers, look at the code and argue: well, it’s just a simple value T, we store it under synchronization, _caches are flushed by synchronization_, and so we can skip synchronization on read.

Is that true? Answer (select over to reveal): There is certainly a release action on monitor unlock in the setter, but the acquire action is missing in the getter. Therefore, the memory model does not mandate that the values stored before the store to _val_ be visible after we read _val_ in another thread — very bad news if those were the stores into _val_ fields.

* * *

![page 094](https://shipilev.net/blog/2014/jmm-pragmatics/page-094.png)

Acquire barrier is missing, you say? OK, let us add one, since we "know" the compiler emits one for a volatile read.

Is this broken? If so, why? Answer (select over to reveal): In current practice, it works for a given conservative VM implementation, but JMM-wise, since we don’t do acquire on the same variable as we did the release on, this is not guaranteed. In short, a smarter VM can see you do not use the sinked value, and therefore can pretend we did not see updates to _BARRIER_, if any, and eliminate it altogether.

* * *

![page 095](https://shipilev.net/blog/2014/jmm-pragmatics/page-095.png)

This is a correct way to do this. Marking the field _volatile_ provides the release action in the setters, and the paired acquire in the getters. This allows us to relax _synchronized_ in the getters, and leave only the lightweight _volatile_.

Is _synchronized_ in the setter still required? Answer (select over to reveal): Yes, because the setter requires mutual exclusion: it should set the _val_ only once.

* * *

### JMM Interpretation: Roach Motel

![page 097](https://shipilev.net/blog/2014/jmm-pragmatics/page-097.png)

It may be hard for an optimizing compiler to figure out if a particular optimization breaks the provisions of JMM. Some advanced compilers may construct the memory flows directly. But, basic compiler guys need a set of simple rules to figure out if something is allowed or not. JSR 133 Expert Group created [The JSR 133 Cookbook For Compiler Writers](http://gee.cs.oswego.edu/dl/jmm/cookbook.html) to cover this.

It is important to note that the Cookbook is the set of conservative interpretations, not the JMM itself. We will talk briefly about how those interpretations may be derived.

* * *

Consider a program, which can be represented by this template execution. The first two types of reorderings are simple:

![page 098](https://shipilev.net/blog/2014/jmm-pragmatics/page-098.png)

![page 099](https://shipilev.net/blog/2014/jmm-pragmatics/page-099.png)

* * *

![page 100](https://shipilev.net/blog/2014/jmm-pragmatics/page-100.png)

These rules effectively allow pushing the code into the acquire/release blocks, e.g. pushing the code into the locked regions, which enables lock coarsening without violating JMM.

* * *

Another two types of reorderings are **conservatively** forbidden. Note that they are not forbidden by JMM itself, but we have to forbid it if local analysis is not able to determine correctness (in some cases, e.g. field stores in constructors, it can):

![page 101](https://shipilev.net/blog/2014/jmm-pragmatics/page-101.png)

![page 102](https://shipilev.net/blog/2014/jmm-pragmatics/page-102.png)

* * *

### Test Your Understanding

![page 103](https://shipilev.net/blog/2014/jmm-pragmatics/page-103.png)

Let us try some real examples again.

What can this code print? Answer (select over to reveal): There is a synchronized-with edge between storing of _ready_, and reading _ready==true_ Therefore, we can see the latest write in the HB order, and that is 42\. However, we can also see the out-of-HB (racy) write, and that also brings us 43.

* * *

![page 105](https://shipilev.net/blog/2014/jmm-pragmatics/page-105.png)

Now we drop _volatile_.

What can this code print? Answer (select over to reveal): Any value is possible, because we can observe any value via the race, and also we can see nothing at all if _while_ loop is reduced to _while(true)_.

* * *

### Benchmarks

![page 108](https://shipilev.net/blog/2014/jmm-pragmatics/page-108.png)

Of course, what’s the use of me posting anything without benchmarks? We want to quantify at least some of the costs. It does not strike me as a good idea to measure the absolute numbers, and therefore we would only show a few important high-level points. The benchmarks are driven by [JMH](http://openjdk.java.net/projects/code-tools/jmh/), and we assume you are familiar with it.

* * *

Let us start with a "hoisting" benchmark. We would like to run a synthetic test which naively computes `v` times `v`. The difference lies within the sharing of underlying _Storage_, and `v` volatility. Not surprisingly, when we are **reading** stuff, it seems like sharing is not important.

The _volatile_ test cases are significantly slower. However, it is not the cost of volatiles themselves, but rather the evidence of too-conservative implementation breaking the hoisting of `s.v` out of the loop, which will move the read before the acquire (see the "Roach Motel" above). Pre-reading `s.v` into a local variable and measuring again is left as an exercise for the reader.

![page 109](https://shipilev.net/blog/2014/jmm-pragmatics/page-109.png)

![page 110](https://shipilev.net/blog/2014/jmm-pragmatics/page-110.png)

* * *

For a **writing** test, we can start incrementing the same variable. We do a bit of backoff to stop bashing the system with writes, and here we can observe the difference both between shared/unshared cases and between volatile/non-volatile cases. One would expect _volatile_ tests to lose across the board, however we can see the _shared_ tests are losing. This reinforces the idea that data sharing is what you should avoid in the first place, not volatiles.

![page 111](https://shipilev.net/blog/2014/jmm-pragmatics/page-111.png)

![page 112](https://shipilev.net/blog/2014/jmm-pragmatics/page-112.png)

* * *

### JMM Updates

![page 113](https://shipilev.net/blog/2014/jmm-pragmatics/page-113.png)

Current mainstream languages seem to adopt SC-DRF across the board. However, there is evidence that strictly supporting SC-DRF might not be profitable for all scenarios. For example, Linux RCUs relax some of the constraints with very good performance improvements on weakly-ordered platforms, and arguably do that without breaking usability much.

So the question for the next JMM update is: can/should we relax SC-DRF to get more performance?

* * *

## Part IV: Out of Thin Air

### What Do We Want

![page 115](https://shipilev.net/blog/2014/jmm-pragmatics/page-115.png)

It would seem that once SC-DRF is established, we are good to go. Any transformation is valid in between synchronized actions, because if there was an unannotated race in the code, the behavior **was** non-SC to begin with. Teach runtimes the SC-DRF rules and you are good, right?

* * *

### What Do We Have

![page 116](https://shipilev.net/blog/2014/jmm-pragmatics/page-116.png)

However, this is only part of the truth. Some transformations still break SC. Consider this program: somewhat surprisingly, it is correctly synchronized, since all SC executions have no races, and the only possible result is `(0, 0)`

* * *

![page 119](https://shipilev.net/blog/2014/jmm-pragmatics/page-119.png)

Now, an optimizing runtime/hardware comes in and tries to speculate over the branch. It might be a good idea at times to speculatively execute the branch and back off if speculation failed.

Suppose we **did** this speculation in the code.

* * *

![page 120](https://shipilev.net/blog/2014/jmm-pragmatics/page-120.png)

Let’s now execute the modified program. The second thread runs unmodified. If we run this program in the order depicted on the slide, we will have `(42, 42)`, because the speculation had turned itself into the self-justifying prophecy! It seems as if `42` came _out of thin air_.

This example _seems_ artificial, until you realize the variable `a` could easily be, say, `System.securityManager`, and we had just undermined platform security guarantees! Scary.

* * *

![page 121](https://shipilev.net/blog/2014/jmm-pragmatics/page-121.png)

To cover for that, JMM forbids out-of-thin-air (OoTA) values. To constructively forbid OoTA, you need some notion of causality (i.e. "what caused what"), which is tricky to introduce in a model that tries to escape the deadly embrace of global time.

* * *

![page 123](https://shipilev.net/blog/2014/jmm-pragmatics/page-123.png)

The entire section in [JLS 17.4.8](http://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.4.8) tries to rigorously specify the "commit semantics", which additionally validates the executions for causality violations. We won’t dive into details here, so enjoy this nice Bill-looking guy trying to explain it.

* * *

![page 124](https://shipilev.net/blog/2014/jmm-pragmatics/page-124.png)

Commit semantics give the last filter in the executions soup. Executions violating the causality requirements cannot be used to reason about the program.

* * *

![page 125](https://shipilev.net/blog/2014/jmm-pragmatics/page-125.png)

This takes us to the final picture. In order to test if an execution is plausible under JMM, you need to see if it passes all the requirements. Note, however, that you can quickly branch-and-bound the set of considered executions based on the failure of a particular test. In most cases we don’t even get to commit semantics, because all the executions that passed the other filters yield only the desired outcomes, and we don’t care to distinguish between them anymore.

* * *

### OoTA and C/C++

![page 126](https://shipilev.net/blog/2014/jmm-pragmatics/page-126.png)

Remarkably, Java so far seems to be the only platform which tried to specify what OoTA really means. Not that Java is very successful with that, given the very complicated and sometimes counter-intuitive causality model.

* * *

### JMM Updates

![page 127](https://shipilev.net/blog/2014/jmm-pragmatics/page-127.png)

Therefore, in the next JMM update the largest question of all is being asked: can we reformulate/fix this to be more bullet-proof, concise, and understandable?

* * *

## Part V: Finals

### Test Your Basic Understanding

![page 129](https://shipilev.net/blog/2014/jmm-pragmatics/page-129.png)

Since we learned a lot about JMM already, let’s start from this simple quiz. What does this program print? Answer (select over to reveal): Nothing, 0, 42, or throws NPE (!). Racy reads abound, and we can really read any value written: either the default one, or that written in constructor. We can even observe first _(a != null)_, and print, only to realize the racy read the second time returned _(a == null)_, setting up an NPE.

* * *

### What Do We Want

![page 132](https://shipilev.net/blog/2014/jmm-pragmatics/page-132.png)

We obviously want to modify the object declaration in such a way that we only get `42` (or nothing). You can guess what hides under five question marks there, right?

* * *

![page 133](https://shipilev.net/blog/2014/jmm-pragmatics/page-133.png)

We need this to protect ourselves from races. We cannot afford to catch fire and break object invariants if an object receiver acts maliciously — otherwise we can’t write secure code.

Nowadays, some brave folks dance around the races trying to optimize performance. Go read [Hans again](http://soft.vub.ac.be/races/wp-content/uploads/2012/09/races2012_submission_3.pdf), please.

* * *

### What Do We Have

![page 134](https://shipilev.net/blog/2014/jmm-pragmatics/page-134.png)

Final fields are remarkably simple to implement, compared to how we need to spec them. On most architectures, it is enough to put a memory barrier at the end of the constructor, and tie the load of object fields via the dependency to the load of original reference. Done.

* * *

![page 135](https://shipilev.net/blog/2014/jmm-pragmatics/page-135.png)

The specification, however, gets rather complicated. We have to reference the _constructors_ in an otherwise syntax-oblivious spec, and introduce special _freeze_ actions at the end of constructor. Intuitively, this freeze action "sticks" the field with the values initialized in constructors. It does not, however, limit the fields from being modifiable (you can still circumvent finality through Reflection), _freeze_ is only about the initializing stores.

* * *

![page 136](https://shipilev.net/blog/2014/jmm-pragmatics/page-136.png)

And here is how it is formally specified. Notice that _w_ and _r2_ may or may not be the write and the read of the _final_ field, they might as well be the write and read of the ordinary, non-final field. What really matters is that the subchain containing freeze action _F_, some action _a_, and _r1_ which reads the final field — all together make _r2_ observe _w_.

Notice two new orders, _dereference order_, and _memory order_ are needed to qualify how frozen values propagate.

* * *

![page 137](https://shipilev.net/blog/2014/jmm-pragmatics/page-137.png)

We "only" need to make sure that all paths from a target read lead up to a write of frozen value via _F_ and the final field read.

* * *

### Constructive Example

This example was greatly untangled by Vladimir Sitnikov and Vladimir Kovalenko, kudos to them! Here is a visualization based on their analysis:

![page 138](https://shipilev.net/blog/2014/jmm-pragmatics/page-138.png)

![page 139](https://shipilev.net/blog/2014/jmm-pragmatics/page-139.png)

![page 140](https://shipilev.net/blog/2014/jmm-pragmatics/page-140.png)

![page 141](https://shipilev.net/blog/2014/jmm-pragmatics/page-141.png)

![page 142](https://shipilev.net/blog/2014/jmm-pragmatics/page-142.png)

![page 143](https://shipilev.net/blog/2014/jmm-pragmatics/page-143.png)

![page 144](https://shipilev.net/blog/2014/jmm-pragmatics/page-144.png)

![page 145](https://shipilev.net/blog/2014/jmm-pragmatics/page-145.png)

![page 146](https://shipilev.net/blog/2014/jmm-pragmatics/page-146.png)

* * *

### Premature Publication

![page 147](https://shipilev.net/blog/2014/jmm-pragmatics/page-147.png)

This is a great example from Jeremy Manson’s PoPL paper. There, the first thread initializes the object and stores `42` to final field `f`, then "leaks" the object reference through `p`, and only then properly publishes via `q`.

Conventional wisdom suggests that final field guarantees evaporate with premature publication, but really, the third thread only observes the fully-constructed object, and we can find only the proper final path. (See the example above for analogy.)

The second thread, however, breaks out of the final path when reading through `p`, and therefore may observe non-frozen value. It is somewhat surprising that the read through `q` can _also_ observe the non-frozen value. This is formally allowed by the properties of _dr_ and _mc_ orders, and has a pragmatic reason:

* * *

![page 148](https://shipilev.net/blog/2014/jmm-pragmatics/page-148.png)

The pragmatic reason is that runtimes may cache final fields once they discover one! Which means that if the compiler discovered `p` and `q` are aliasing the same object, then we can say `r3 = r2`, and be done with it. So, if we observed the under-constructed object, our thread became _tainted_, and all hell broke loose.

* * *

### Test Your Understanding (tricky)

![page 149](https://shipilev.net/blog/2014/jmm-pragmatics/page-149.png)

Notice that the spec talks about initializations in _constructors_, and here we have something else. Answer (select over to reveal): Of course, we will see either 42, or nothing. Field initializers and instance initializers fire in the course of instance initialization, and arguably are part of constructor.

* * *

### JMM Updates

![page 151](https://shipilev.net/blog/2014/jmm-pragmatics/page-151.png)

There are quite a few problems with _final_-s, mostly with its orthogonality with regard to other JMM elements. It is particularly interesting how to achieve visibility for initializing stores in a constructor if the field is already _volatile_. (Hint: volatile [is not enough](http://cs.oswego.edu/pipermail/concurrency-interest/2013-November/011954.html) as it stands in spec, at this point).

It is also interesting, from a pedagogical standpoint, whether we should ostracize users who forget to declare their write-once-in-constructor fields _final_, and get their code to blow up on non-x86 platforms.

Therefore, the next JMM update needs to decide whether we should extend the final field guarantees to all fields initialized in all constructors.

* * *

### Benchmarks

This section covers the final field considerations for the updated Java Memory Model. See the more-thorough explanation [in a separate post](http://shipilev.net/blog/2014/all-fields-are-final/).

![page 153](https://shipilev.net/blog/2014/jmm-pragmatics/page-153.png)

Of course, we would like to rigorously quantify what would it cost to mark all fields final. Since final field stores require memory barriers in the constructors for weakly ordered platforms, we also take the ARM host as a testing platform.

* * *

Here are the benchmarks: _chained_ call N constructors up the hierarchy, initializing a single field per class, and _merged_ initialize all N fields at once. Fields can be plain or final. We test with different N-s to see if performance changes in a sane manner.

![page 154](https://shipilev.net/blog/2014/jmm-pragmatics/page-154.png)

![page 155](https://shipilev.net/blog/2014/jmm-pragmatics/page-155.png)

* * *

![page 156](https://shipilev.net/blog/2014/jmm-pragmatics/page-156.png)

x86, being a Total Store Order machine, does not have memory barriers, and so the difference between all four variants is within the measurement error, regardless of N.

* * *

![page 157](https://shipilev.net/blog/2014/jmm-pragmatics/page-157.png)

On weakly-ordered machines, _final_ involves a real memory barrier, and the barrier cost is clearly visible in the addition in execution time on the green bar. Moreover, we have barriers in each superclass, which explain why it takes linearly more time for the red bar. We can teach the VM to merge the barriers though, after which the cost of enforcing the final semantics would drown in allocation costs.

* * *

## Conclusion

![page 160](https://shipilev.net/blog/2014/jmm-pragmatics/page-160.png)

When we were dealing with a nasty JMM bug, Doug dropped a gem of wisdom that should sum up this talk nicely. It would also be good if people using concurrency constructs were able to figure out why and when those constructs work. Hopefully this talk improved JMM understanding.

* * *

![page 161](https://shipilev.net/blog/2014/jmm-pragmatics/page-161.png)

There are some known problems we would like to address in JMM…​

* * *

![page 162](https://shipilev.net/blog/2014/jmm-pragmatics/page-162.png)

…​which gave rise to the ["Java Memory Model update"](http://openjdk.java.net/jeps/188) effort.

* * *

![page 163](https://shipilev.net/blog/2014/jmm-pragmatics/page-163.png)

At the end of the day, a few useful links for readers:

*   [Goetz et al: "Java Concurrency in Practice"](http://www.amazon.com/Java-Concurrency-Practice-Brian-Goetz/dp/0321349601)

*   [Herilhy, Shavit, "The Art of Multiprocessor Programming"](http://www.amazon.com/Art-Multiprocessor-Programming-Revised-Reprint/dp/0123973376)

*   [Adve, "Shared Memory Consistency Models: A Tutorial"](http://www.hpl.hp.com/techreports/Compaq-DEC/WRL-95-7.pdf)

*   [McKenney, "Is Parallel Programming Hard, And, If So, What Can You Do About It?"](ftp://kernel.org/pub/linux/kernel/people/paulmck/perfbook/perfbook.html)

*   [Manson, "Special PoPL Issue: The Java Memory Model"](http://unladen-swallow.googlecode.com/files/journal.pdf)

*   [Huisman, Petri, "JMM: A Formal Explanation"](http://www-sop.inria.fr/everest/personnel/Gustavo.Petri/publis/jmm-vamp07.pdf)