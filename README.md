# Connected Embedded Rust: A Survey of Software Platforms and Network Stacks

## Preliminary Report Skeleton

1. Introduction
    - Systems programming in Rust
      - Brief intro to Rust Programming Language
      - Brief discussion of embedded rust layers: Micro-architecture crates, PAC crates, HAL crates, Board Crates
    - Baremetal programming: Rust standard library; `no_std`, allocation, unsafe rust
2. I/O, Concurrency and Asynchronous programming
    - _futures_ in Rust
    - libraries for I/O & async programming: `tokio`, `smol`, `mio`
3. Survey of Embedded Software Platforms  
    - 3.1. Frameworks
      - _Embassy_: Framework for embedded applications
      - _RTIC_: Concurrency framework  
    - 3.2. Rust Operating Systems
      - Well-known OSes: Tock, Redox, Theseus OS, RustyHermit
      - Prominent OSes for embedded systems:
        - _Tock_: Well know Rust OS for embedded systems
        - _RustyHermit_: unikernel
        - _Hubris_: Lightweight kernel
        - _xous_: extremely small kernel
        - _stardust-oxide_: unikernel
        - _DroneOS_: embedded OS for real-time applications
        - _(this list is subject to change. Other Rust OS that might also be discussed are: Tifflini, moros, aero, karla, intermezzo, felix, rcore)_
      - Common patterns, best-practices, etc.
4. Networking stacks in Rust systems software  
    - 4.1. smoltcp networking stack
      - standalone, event-driven TCP/IP stack for bare-metal systems  
      -> predominant choice in the ecosystem for networking stacks in embedded systems software  
    - 4.2 Deep-dive: usage & integration of smoltcp (in the aforementioned OSes)
    - 4.3. Discussion: alternatives to smoltcp
      - libpnet: Cross-platform, low level networking
      - embassy-net, tokio-net
5. Comparison and Evaluation
6. Conclusion
