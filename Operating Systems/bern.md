# Bern RTOS

- Master of science project from Stefan Lüthi: https://gitlab.com/bern-rtos/doc/book-bern-rtos-kernel/-/jobs/artifacts/main/raw/Bern_RTOS_Kernel.pdf?job=latex

Real-time operaitng system for microcontrollers

Design principles
- insolated processes
- stackful threads
- kernel only depends on CPU core, any/ no HAL can be use
- priority scheduling, round-robin for same prio
- real-time; interrupt handling without kernel interation

---

(_Interesting OS, but from what I saw it is very C-like in the sense that the Kernel, Processes, the Scheduler etc are all static; and a lot of unsafe code is used to cirumvent Rust's type system and ownership rules_)
 
