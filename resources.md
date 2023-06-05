# Resources

## Primary Resources

### Papers / Articles

- Tock OS:
  - "The Tock Embedded Operating System": <https://dl.acm.org/doi/pdf/10.1145/3131672.3136988>
  - "Multiprogramming a 64 kB Computer Safely and Efficiently" <https://dl.acm.org/doi/pdf/10.1145/3132747.3132786>
  - "Tiered trust for useful embedded systems security" <https://dl.acm.org/doi/10.1145/3517208.3523752>
  - "Energy Harvesting Systems Need an Operating System Too": <https://dl.acm.org/doi/pdf/10.1145/3417308.3430274>
  - The case for writing a Kernel in Rust: <https://dl.acm.org/doi/abs/10.1145/3124680.3124717>
- Theseus OS:
  - "Theseus: an Experiment in Operating System Structure and State Management" <https://www.usenix.org/system/files/osdi20-boos.pdf>
  - "Theseus: Rethinking Operating Systems Structure and State Management" <https://dl.acm.org/doi/10.5555/AAI28735941>
- RustyHermit:
  - "Exploring Rust for Unikernel Development" <https://dl.acm.org/doi/abs/10.1145/3365137.3365395>  
- Embedded Rust:
  - Tighten Rust’s Belt: Shrinking Embedded Rust Binaries: <https://dl.acm.org/doi/pdf/10.1145/3519941.3535075>
  - On Utilizing Rust Programming Language for Internet of Things: <https://ieeexplore.ieee.org/abstract/document/8319363o>
  - Towards Rust for Critical Systems: <https://ieeexplore.ieee.org/abstract/document/8990314>
  - System Programming in Rust: Beyond Safety <https://dl.acm.org/doi/pdf/10.1145/3102980.3103006>
  - Learning and programming challenges of rust: a mixed-methods study: <https://dl.acm.org/doi/10.1145/3510003.3510164>
- Readleaf:
  - RedLeaf: Towards An Operating System for Safe and Verified Firmware: <https://dl.acm.org/doi/pdf/10.1145/3317550.3321449>

### Rust Crates / Implementations

#### Embedded Rust

Frameworks:

- embassy: framework for embedded applications, <https://embassy.dev/>
- rtic: rEal-Time Interrupt-driven Concurrency (for ARM only?)

OSes:

- Tock: Embedded OS wioth support for concurrent applications                               [embedded, networking]
- Hubris: Lightweight kernel for embedded                                                   [embedded, networking]
- Redox: Unix-like Operating system                                                         [!embedded, networking]
- xous: designed for extremely small kernel                                                 [embedded, networking]
- Theseus OS: novel OS structure                                                            [!embedded, networking]
- DroneOS: Embedded Operating System for writing real-time applications in Rust.            [embedded, !networking]


#### Drivers / I/O

- mio: fast, low-level I/O library for Rust
- message-io: handles the OS socket internally and offers a simple event message API to the user.
- embedded-hal(-{async, nb}): build platform agnostic drivers

#### Asynchronous Programming

- futures
- tokio
- smol: A small and fast async runtime

#### Networking

- smoltcp
- pnet: Cross-platform, low level networking using the Rust programming language.
- embassy-net: embassy networking functionalities
- embedded-nal: embedded network abstraction layer

### (Online-)Books

- Network Programming with Rust by Abhishek Chanda
- The embedded Rust book
- The embedonmonicon: create `#[no_std]` Rust application from scratch
- Redox OS: <https://doc.redox-os.org/book/>
- xous-book: <https://betrusted.io/xous-book/>

### Blog Posts

- Writing an OS in rust: <https://os.phil-opp.com/>
- Writing embedded drivers in rust: <https://hboeving.dev/blog/rust-2c-driver-p1/>
  - Part 2: <https://hboeving.dev/blog/rust-i2c-driver-p2/>
- bkernel: a Rust Operating System: <https://www.alexeyshmalko.com/2015/bkernel-a-rust-operating-system/>
- Should I use Embassy or RTIC: <https://cohost.org/jamesmunns/post/67555-should-i-use-embassy>
- rust-raspberrypi-OS-tutorials: write embedded OS in Rust

## "Maybe-resources"

_Not primary resources, but potentially still relevant._

OSes (that are either not for embedded, or don't include a networking stack, or both):

- Tifflin (rust_os): experimental kernel [!embedded, networking]
- moros: requires BIOS, only x86-64 arch [!embedded, networking]
- aero: 64 bit, higher half kernel [!embedded, networking]
- kerla, [!embedded, networking]
- intermezzo: for learning how to write an OS [!embedded, !networking]
- felix: targets x86 [!embedded, !networking]
- rCore: OS on RISCV [?embedded, networking]
- ArceOS: unikernel [?embedded, networking]
- zCore: reimplementation of Zircon microkernel in Rust
  - unfortunately almost all docs on this are in chinese
- RustyHermit: x86 unikernel [embedded, networking]
- stardust-oxide: builds on Xen Hypervisor that manages the physical ressources [!embedded, networking]

Networking stacks/ utilities (all require rust std library?):

- ntex: framework for composable networking services
- net2
- socket2
- libc
- nix: implement libc in rust
- naia: networkign architecture for interactive applications (multiplayer)

Other:

- CHERI: Capability Hardware Enhanced Risc Instructions
- zbus: Rust API for D-Bus communication -> inter-process communication solution
- CantripOS /KataOS / Project Sparrow: Secure OS from google: <https://github.com/AmbiML/sparrow-manifest> [embedded, !networking]
  - runs on top of seL4 kernel (C)
- <https://github.com/flosse/rust-os-comparison>
- RIOT
- FreeRTOS.rs: Rust interface for the FreeRTOS API
- Internet of Streams - IoT with Embedded Rustlang: <https://www.youtube.com/playlist?list=PLX44HkctSkTewrL9frlUz0yeKLKecebT1>
- Featherweight Rust: <https://link.springer.com/chapter/10.1007/978-3-031-06773-0_22>
- <https://github.com/rust-embedded/not-yet-awesome-embedded-rust>

## "Non-resources"

_What will **not** be focus of the paper._

- game networking libraries, e.g. laminar, amethyst,
- High level networking libraries, e.g. libp2p
- oreboot: coreboot without c (written in Rust instead) -> BIOS
- Tooling: cargo-binutils, gdb, cross-compiling
- Writing drivers
- seL4-sys: Build rust applications for seL4 microkernel
- Maybe: Rust vs C; Interoperability

Inactive / Unmaintained OSes:

- Reenix                                [!embedded, !networking]
- RustOS                                [!embedded, !networking]
- QuiltOS                               [!embedded, ~networking]
- bkernel                               [embedded, !networking]
  - blog post linked above might still be relevant
- Stupid Operating System               [!embedded, !networking]
