# Notes (WIP)

## Rust Language

### `no_std`

Source: <https://docs.rust-embedded.org/book/intro/no-std.html>

- No system interface (e.g. OS)
- Baremetal: not code has been loaded
- `libcore`: platform-agnostic parts of `std`
- Standard libraries (`libstd`) depend on system interface
  - Provides common way of accessing OS abstractions
  - Runtime: stack overflow protection, spawn main thread, read CLI args
- `#[entry]`, `![no_std]`, `![no_main]`
- No memory allocator: use `alloc` crate with suitable allocator instead (for heap and collections)
- Panic handling needs to be implemented: `#[panic_handler]`
- Layers (low to high):
  - Micro-architecture Crate: handles periphery, calls to processor call (e.g. interrupts), e.g. declare exception & interrupt handling trait
  - Peripheral Access Crate (PAC): this wrapper over memory-wrapper registers
  - HAL Crate: user-friendly API for processor, often impl embedded-hal
  - Board Crate: abstracts HW details: pre-config. peripherals, GPIO -> provided functionalities differ a lot between boards
- concurrency, critical sections, atomic access
  - `UnsafeCell`, `Send`, `Sync`, `Mutex`
  - RTIC framework: Real TIme Itnerrupt-driven Concurrency
- Best-practices compared to C"
  - Compile-time codeselection
  - Compile-time Sizes and Computations
  - Macros (even needed?)
  - Iterator vs Array Access
  - Reference vs pointers
  - Volatile Access
  - PAcked and Aligned Types
- Interoperability
- bsp/ and aarch/ folders

#### embedded-hal

- traits: GPIO, serial comm, I2C, SPI, timer, ADC
- users of the HAL: HAL implementation, Driver, Application
- used by hubris & thesus for serial wrapper


## Segmented Stacks

_(DroneOS also uses a segmented stack -> compare?)_

<https://link.springer.com/article/10.1007/BF00365393>

- Problems with contiguous stacks:
  - Inefficient if no virtual memory
    - physical memory is statically allocated to them
  - Without virtual memory: stack overflow difficult to detect
    - With MPU: can be detected through MPU
    - else: add stack probing -> hgiher overhead
- segmented stack: stacklets that are dynamicall allocated from heap
  - grows on demands, frees memory when offering function returns
  - downsides: memory inefficiency; performance
- Analysis of case study:
  - Stack use substantial memory
  - Maximum stack usage is significantly smaller than allocated stack size
  - Task spend mist time blocked with min stack usage
  - Large arrays & strcuts are commonly declared as static variables -> don't need to be static in practice


## An overview of the embedded rust ecosystem

<https://www.youtube.com/watch?v=vLYit_HHPaY>

- micro controllers control I/O through registers: typically memory mapped
- svd files: 
  - published my microcontroller manufacturers
  - describe function of each register
  - use svd2rust to generate rust crate
- PAC crates: some patches, then svd2rust
  - mostly safe interface, abstraction
  - no check for dependencies between peripheries
  - no initialisation
- HAL: high level interface around PAC
  - higher abstraction than a HAL in C
  - setup common structs (e.g. RCC to manange system and peripheral clocks), aquire and configure peripheral (which take the common struct as input to guarentee that is was set up)
  -> guided by type system (e.g. via Builder pattern)
- embedded HAL: traits for common peripherals
- Board support crates: utilities for breakout boards
