# All Operating Systems in Rust

## Embedded 

(deep-dived in this report)
- Tock OS
- Hubris OS
- Drone OS
- Xous

Not sure:
- rxinu: "Rust implementation of Xinu educational operating system"
  - only supported target is x86, but the original Xinu OS targets embedded systems

## Non-embedded OSes


### Unix-Like OSes, target x86 (and/ or aarch64)

- Redox OS: fully featured, unix-like OS, micro-kernel, target x86
- Theseus OS: maximal leveraging of Rust programming language
  - many tiny compoentn with clearely-defined runtime-persistent bound itneract without holding states for each other
  - use language-level machanisms, let compiler enforce invariants about OS semantics
- Aero: monolithic
- Kerla: monolithic
- rCore: Rust version of THU uCore OS
  - maybe also for embedded? supports RISCV & targets without MMU

## x86 / 64bit; not unix

- Poplar: "Microkernel and userspace written in Rust exploring modern ideas"
- Pluggable Interrupt OS
- RedLeaf
- rust_os/ Tifflin
    - HAL: everything feature-gate
- Moros: requires external bootloader
- Felix: no external dependencies

### Unikernel

- RustyHermit: lighweight unikernel (for high-performance / could-computing)
  - kernel is libhermit-rs
- ArceOS: modular OS
- stardust-oxide: relies / builds upon XEN hypervisor

### Kernels

- zCore: Reimplementation of Zircon microkernel (used for google's Fuchsia OS)

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
- Intermezzo
- nebulet: "A proof-of-concept microkernel that implements a WebAssembly "usermode" that runs in Ring 0. "

