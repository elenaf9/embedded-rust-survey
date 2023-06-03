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
