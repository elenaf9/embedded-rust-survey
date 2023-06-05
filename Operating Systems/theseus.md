# Theseus OS

_Theseus does not target embedded systems, but is nevertheless intersting due to their "intralingual" design, i.e. leveraging of the Rust programming language_

## Theseus: an experiment in operating system structure and state management

source: <https://dl.acm.org/doi/10.5555/3488766.3488767>

- Goal: improve OS modularity
  - reduce states that one components holds for another
- Problem (that it tries to solve): State spill
  - after an interaction, the correctness of one components depends on the state of another
  - "fate sharing" between otherwise modularized components
- Design principles:
  - many tiny components with clearly-defined runtime-persisten bounds interact without holding states for each other
  - intralinagual approach: use language-level mechnisms; compiler enforces invariants about OS semantics
    - Empower compiler to apply safety checks
    - shift seminartic errors from runtime- to compile-time errors
    - allows to assume resource bookkeeping duties
    -> reduces states the OS must maintain
    - reduce state spilling
- [`no_std`]

### Design

- Cells: small distinct components
  - can be composed & interchanged at runtime
  - core building block of OS
  - coexist alongside core OS in single address sapce
  - all execute at single privilege level
  - isolation realized through language provided type & memory safety (not hardware protection)
- Principles:
  - require runtim-persistent bounds for all cells
  - Maximize power of language & compiler
  - minimize state spill between cells
- Cells have clear bounds:
  - Implementation time: crate
  - Compile time: single object file
  - Runtime: set of loaded memory regions with per-section bounds & dependency metadata
  -> foundation for strong data/fault isolation & state management
- Theseus loads & links all cells into system on demand at runtime
- Each module is its own crate! split functionailty into many tiny crates until circular dependencies halts further decomposition
- Linkage is deferred to runtime:
  - compilation process is split at linker stage
  - raw cell objects are placed directly into OS image
  - jump-start thesus with `nano_core`:
    - set of normal cells, statically linked into a tiny executable "base kernel" image
    -> only component needed to bootstrap bare-minimum environment
    - supports virtual memory & loading/linking object files
    - `nano_core` fully replaces itself at final bootstrap state by dyn loading it cells one by one
      -> to preserve it bounds and dependencies that would otherwise be lost for statically linked cells
    - can be safely unloaded after bootstrap

### Intralingual

- match Rust runtime model
  - onle single address space (SAS)
    - 1:1 virtual-to-physical mapping
  - singel privilege level (SPL_
  - single allocator instance
    - global heap serves all allocation requests
    - multiple heaps within that single instance are supported
- Use Rust's build-in reference types (&T, Arc)
- Lossless interfaces: preserve language-level context:
  - lifetimes, type, ownership/ borrowed status
- Language-level knowledge must not be lost (i.e. not convert an type into a raw integer and then reconstruct)
- Implement all cleanup semantics in drop handlers
- Custom unwinder:
  - employ stack unwinding to always release resources (both, in normal & exceptional execution)
  - custim unwinding logic based on DWARD standard
- Supports resource revokation:
  - by killing & unwiding an uncooperative task
  - cooperatively, e.g. via Option or weak reference

### Memory Management

- `MappedPages`: region of virtually-contiguous pages, mapped to real physical frames
  - sole way to acess memory 
  - backing representation for stacks / heaps / memory regions
-   Invariants:
  1.  mapping from virtual pages to physical frames must be one-to-one, or bijective
    - no aliasing (sharing)
  2. memory beyond mapped region's bounds not accessible
    - access only thorugh overlay struct/ slice
  3. memory unmappend only once, when no remaining ref
    - realized through drop handler
  4. memory region only mutable / executable if mapped as such
    - `MappedPagesMut` offers `as_type_mut()`
    - `MappedPagesExec` offers `as_function()`
- No unexpected invalid page faults can occur

### Task management

- set of functions to handl each stage of task lifecycle
  - consistent set of generic type parameters
  - `spawn_task`: sets up stack so that it will jump to task_wrapper upon first context switch
  - `task_wrapper`: invokes entry function normally
  -  `task_cleanup_success`, `task_cleanup_failure`: depending on return value from task function
  - `task_cleanup_final`: remove task from runqueue, deschedule
- Invariants:
  - Spawning new task mus not violate memory safety
    - not used std, own task abstraction
    - doesn't offer `fork` -> unsafe and unsuitable for SAS systems

```rust
pub trait TFunc<A,R> = FnOnce(A) -> R;
pub trait TArg = Send + 'static;
pub trait TRet = Send + 'static;
```
  - all task states must be release in all possible execution paths
    - different end stages require different cleanup actions (`_success`, `_failure`)
  - memory transitively reachable from a task’s entry function must outlive that task.
    - chain of ownership: 
      - task owns cell with entry function
      - cell owns any cell it depends on
- only sawpping stack pointer registers during contex switch is not intralinugal

#### Inter-task communication channels

<https://scholarship.rice.edu/bitstream/handle/1911/109201/BOOS-DOCUMENT-2020.pdf?sequence=1&isAllowed=y>

- realised through simple shared memory location (e.g. ref to mutex-protected heap obejct)
    - direct function calls; no IPC mechanism
- additionally implements ITC channels:
  - synchronous rendezvous-based bufferless channel
  - async buffered channe;


### State Management

- Opaque exportation in client-server interaction
  - each client is responsible owning the progress state
  - clients can not introspect into or modify server state
- Server does not booking what ressources a client owns, instead the client directly owns underlying ressource
- clients do not need to clean uo their owned ressources
  - clean-up via the object's drop function
- joint ownerhsip between multiple ressources:
  - realized via heap-allocated object with ARCs
  - state spill into heap unavoidable
 

