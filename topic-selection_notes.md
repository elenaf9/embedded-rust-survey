# Notes on seminar topic

- Survey
- Goal: network stack for RIOT-rs <https://github.com/future-proof-iot/RIOT-rs>
- Research rust OSes for IoT, specifically focusing on network stack
- Libp2p for IoT?
  - Specs probably not optimized for IoT
  - no-std, specs compartible version of Lip2p-rs?
- RIOT vs Tock:
  - Tock: one thread per application, apps run in user space, stronger isolation
  - RIOT: focus portability, more like a library

Resources:

- QUIC for IoT: <https://summit.riot-os.org/2020/wp-content/uploads/sites/15/2020/09/s1-2-lars-eggert.pdf>
- Tock: <https://github.com/tock/tock>, <https://www.tockos.org/documentation/>
- RIOT on FPGA: <https://forum.riot-os.org/t/riot-on-a-vexriscv-and-spinalhdl-fpga-softcore/360>

## Topic Variants

- Survey of (embedded) Rust applications, focus on how the networking stack is implemented
  - Survey of open-source low-level / embedded rust project
  - Discussion of general architecture / designs, best practices, Rust-idiomacy etc
  - Discussion of which / how networking stacks are used /integrated / implemented
  
- Survey of network programming in the Rust ecosystem
  - Survey of rust networking stacks & libraries in general, best-practices (e.g. sans-io for networking protocols), rust-idiomacy
    - Existing Protocol implementations (QUIC, tcp, ...)
    - High level networking libraries (e.g. libp2p)
  - Deep-dive into crates for low-level networking programming; APIs, abstraction layers, socket implementations
  - Discuss how they are used/ integrated by reviewing a couple of (embedded rust) projects, preferably OSes
