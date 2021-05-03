<!-- Please prefix the notes with the date as in [22/12/2020] -->

[30/04/2021]

The imperative of **performance and access to low-level programming** encourage use of C or C++ when dealing with memory manager. Rust promises *a systems programming language that runs blazingly fast, prevents segfaults, and guarantees thread safety*.

Rust presents several important aspects such as. **Ownership**, where variable binding grants a variable unique ownership of the value it is bound to. Unbound variables are not allowed and rebinding consists of transferring variables ownership. When a variable goes out of scope, the associated ownership expires and resources are reclaimed. **References** are borrowed in Rust and it allows one or more co-existing *immutable* references to an object and exactly one *mutable* reference with no immutable references. The ownership cannot be moved when the reference is borrowed. This eliminates data races altogether. **Data Guarantees (Wrapper Types)** are provided with different guarantees and tradeoffs. Plain references guarantee read-write lock for single-threaded code. `Box<T>` represents a pointer which uniquely owns a piece of heap-allocated data, `Arc<T>` provides an atomically-counted reference-counted shared pointer to data and guarantees it stays available until `Arc<T>` goes out of scope. **Unsafe** operations (raw pointers, sharing data across threads) are allowed in Rust but need to be declared within a marked block.

Building a GC in Rust brought up the following challenges:

- **Encapsulating address type:** a difference needs to be made between an *address* (arbitrary location in the memory space) and an *object reference* (language-level object that points to a memory piece with added meta-data). Object reference to address is always safe while the other way around is not. A single-field `tuple struct` is used for abstracting both other the word-width integer `usize`. The `Address`s can be created from raw pointers or derived from an existing `Address` but cannot be created from an arbitrary number (except 0 to avoid a `None` initialization overhead).
- **Ownership of memory blocks:** Thread-local allocation is an essential element of high performance memory management for multi-threaded languages. The usual approach is to maintain a global pool of raw memory regions from which thread-local allocators take memory as they need and to which thread-local collectors push memory as they recover it. `Blocks` objects are created to be in a coherent state among *usable*, *used* or *being allocated into by a unique thread*. The global memory owns the block and arranges them into a list of usable and used `Blocks` . The Rust's ownership model ensures that allocation will not happen unless the allocator *owns* the `Block`. Therefore every `Block` is either: owned by the global space as usable, owned by a single allocator and being allocated into or owned by the global space are used.
- **Globally accessible per-thread state:** A thread-local allocator avoids costly synchronization on the allocation fast path. However allocators might be told to yield by a collector and need to provide some access to their state. Every `Allocator` is broken up in two pieces, a *local* and a *global* one.

- **Library-supported parallelism:** The efficiency of the collector depends critically on the implementation of fast, correct, parallel work queues. Those are coming from `std::sync::mpsc` (multiple-producers single-consumer FIFO) and `crossbeam::sync::chase_lev` (lock-free Chase-Lev work stealing deque). 

Some **abuses** of Rust had to be performed to improve performance. To represent collections state, memory managers often use bit maps (or byte maps). Examples include **card tables** (remember modified memory regions) and **mark tables** (remember marked objects). Concurrent writing is needed in an array but forbidden by Rust. 
