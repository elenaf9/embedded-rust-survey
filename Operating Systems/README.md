# All Operating Systems in Rust

## Embedded 

(deep-dived in this report)
- Tock OS
- Hubris OS
- Drone OS
- Xous


## Non-embedded OSes


### Unix-Like OSes, target x86

- Redox OS: fully featured, unix-like OS, micro-kernel, target x86
- Theseus OS: maximal leveraging of Rust programming language
  - many tiny compoentn with clearely-defined runtime-persistent bound itneract without holding states for each other
  - use language-level machanisms, let compiler enforce invariants about OS semantics

### Kernels

- rust_os/ Tifflin
    - everything feature-gate
- Moros
  - targets x86 with BIOS

## Rust OS on top of C kernels

- Cantrip OS:
  - from Google
  - on top of seL4 kernel
- FreeRTOS.rs: rust interface for FreeTROS API

## Unmaintained OSes

- Reenis
- RustOS
- QuiltOS
- bkernel
- Stupid Operating System

## TBD

- Aero
- Kerla
- Intermexxo
- Felix
- RCore
- ArceOS
- zCore
- RustyHermit
- stardust-oxide
- axle PS
- Pluggable Itnerrupt OS
-  RedLeaf
- nebulet: "A proof-of-concept microkernel that implements a WebAssembly "usermode" that runs in Ring 0. "
- rxinu: "Rust implementation of Xinu educational operating system"
- helenos: "A portable microkernel-based multiserver operating system written from scratch."
- Poplar: "Microkernel and userspace written in Rust exploring modern ideas"

