# Real-Time Interrupt-driven Concurrency

- concurrency framework: no kernel, relies on external HALs
- Stack Resource Policy based Scheduling:
  - extends Priority Interitance Protocols
  - task run on single shared stack
  - task run-to-completion with wait free access to shared resources
  - single critical section
- static priority scheduling
- only Cortex-m devices

## Hardware Tasks

- Run as interrupt handlers
- uses hardware interrupts to schedule & run tasks
- interrupts defined by PAC or HAL crates
- assumed to run-to-completion and return


## Software tasks

- not explicitly bound to specific interrupt vector
- bound to a "dispather: interrupt vector
- may be started once and run forever
  - any loop must be broken by at least one `await` (yielding operation)
- Dispatcher: Async exector for software tasks
  - all tasks at same prio share interrupt handler
  - List of dispatcher needs to be defined, nees to cover all priotirylevels used by software tasks
- spawning an already spawned task will cause error
- Delays can be await'ed
  - can be used to implement timeouts


## Resource Usage

- system-wide resources: `local` and `shared`
-  local: onyl accessible to a specific task
  - commonly drivers or large objects
- shared: mutex trait implemented for each shared resource
  - critical section: closure for accessing the shared object
  - higher prio task can preempt critical sections if it does not contend for shared object 
  - default &mut, but can also specify only &
  - `#[lock_free]` criticcal section not required if ressource only shared with same-prio task

## Communication over channels

- "essentially a wait queue"
- mpsc, statically allocated memory
- implemented using a small global critical section
- copies payload from sender to static variable, from static variable to receiver
  - instead can send an owning pointer into the buffer

## Comparison with embassy:

- embassy provides HAL & exectuor/ runtime, RTIC only execution framework 


