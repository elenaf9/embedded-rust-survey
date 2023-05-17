# Concurrency

## Rust Atomics and Locks - Low-Level Concurrency in Practice

by Mara Bos

<https://marabos.nl/atomics/>

### Basics

- Arc = atomic RC -> thread safe
- Interior mutability: mutate through immutable reference
- shared vs exclusive references
- Cell: only copying value or replacing it is allowed
- RefCell: like Cell, but allows (mut) borrowing content
    - panics if two threads mut panic same time
- RwLock: concurrent version of RefCell, block on concurrent access
- UnsafeCell: unsafe, gives raw pointer to inner value
- T is Send iff it T can be sent to another thread
- T is Sync iff &T is Send
- Rust Mutex vs C Mutex: In Rust contains data, in C(++) not
- Condvar: Conditional Variable: wait & notify

### Memory Ordering

- Problem: Within a thread one can assume related instructions are excecuted sequentially, but not between threads since re-ordering may happen. E.g.
```rust
static X: AtomicI32 = AtomicI32::new(0);
static Y: AtomicI32 = AtomicI32::new(0);

fn a() {
    X.store(10, Relaxed); (1)
    Y.store(20, Relaxed); (2)
}

fn b() {
    let y = Y.load(Relaxed); (3)
    let x = X.load(Relaxed); (4)
    println!("{x} {y}");
}
```
-> if a and b are executed by concurrent threads, a valid order is (2) (3) (4) (1), since 1 and 2 are not dependent and thus can be reordered within a.
- guarenteed "happens-before" orders:
    - related instructions
    - locking and unlocking a mutex
    - \<code> spawn(); join() \<code> 
- Ordering / fence()
    - Relaxed: no happens-before relationship, but "thread observe modification in same order"
    - Release-Acquire
        - Release: Applies to store operations
        - Acquire: Applies to load operations
        - Everything before Release-store happens before everything after Acquire-load
        - AcqRel: combination: acquire-ordering & release-ordering
    - SeqCst (strongest ordering): Release-Aquire ordering + total global consist order of SeqCst operations (=> within a thread operations with SeqCst will never be re-ordered)

### Interesting Patterns

-  Sequence locks: data + counter
    - writer only can write if counter is even
    - Writer increment counter before and after write
    - Reader read counter before and after reading data
    - Data only valid if before == after && even
- RCU (read-copy-update)
    - Store pointer to struct in atomic
    - To update: copy value, update it, update atomic pointer
- Lock-free linked list
    - List with atomic pointers to next element
    - Update like in RCU
    - Add / Remove items by updating amotic pointers to next item

    


### Other interesting notes

- Unsafe code can cause undefined behaviour in preceeding code:
```rust
match index {
   0 => x(),
   1 => y(),
   _ => z(index),
}

let a = [123, 456, 789];
let b = unsafe { a.get_unchecked(index) };
```
-> compiler assumes index is <=2, thus that z is only ever called iff index==2, thus can even optimize the `z` function. If index is now 3, it can cause all types of undefined behavior

