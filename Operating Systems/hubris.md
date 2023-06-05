# Hubris Operating System

## Hubris Reference

<https://hubris.oxide.computer/reference/>

- application consists of collection of task + hubris kernel
- `app.toml` config file defines structure of application
- Features:
  - For 32b microcontrollers with region-based memeory protection
  - tasks: separately-compiled programms & drivers
    - "physically addressed": isolated in memory; non-overlapping sections of adress space; kernel has own space
      -> no virtual memory
      - vs Unix/ Windows: each programm believes it own whole address space
    - run in unprivileged processor mode
    - component-level reboot possible; under app control
    - "supervisor task" implements task management & crash recovery
  - IPC
  - small kernel in priviledged mode
  - drivers implemented as tasks
  - application shiped as one firmware image (with kernel & tasks)
    - piecewise update of tasks or dynamic creation not possible
-> Philosophy:
  - More memory safety
  - Fault isolation
  - Holistic deployment
- Pragmatism:
  - interrupts may be handled by app in priviledged mode
  - sections may be writable AND executable
  - fixed system-level resoruce allocation
- Few low layer features -> even if it results in more high-level work
- Avoid "being clever" -> instead try to be correct

### Tasks

- have priorities fixed at build time
- task's executable code is immutable
- scheduling:
  - strict (preemptive) priority scheduling
  - otherwise cooperative (not time-slicing/ round-robin)
- tasks don't share code (=> no shared libraries)
- no dynamic task management: tasks can't be created / destroyed
  -> avoid DoS or fork-bomb attacks
- On task fault: looses CPU, state is preserved, fault record in kernel
  - supervisor is notified and may take action
    - reads fault from kernel
    - may reinitialize failed task
  - can't change memory protection
  - doesn't initialize/ zero data

#### Servers

- tasks that implement some API

#### Supervisor task

- implemented as task, index 0, with highest priority
- kernel recognizes supervisor task
  - informs it if a task crashes
  - any task can SEND to it
  - supervisor can only SEND to kernel
```
     +-----------+  +-----------+  +-----------+
+----+ app task  |  | app task  |  | app task  +-----+
|    +--+----+---+  +--+-+---+--+  +-+---------+     |
|       |    |         | |   |       |               |
|       |    |  +------+ |   +-+  +--+               |
|       |    |  |        |     |  |                  |
|       v    v  v        v     v  v                  |
| +------+ +------+ +------+ +------+ +----------+   |
| |server| |server| |server| |server| |supervisor|   |
| +---+--+ +--+---+ +---+--+ +--+---+ +----+-----+   |
|.....|.......|.........|.......|..........|.......  |
| +---v-------v---------v-------v----------v-----+   |
| |                                              |   |
+->+                  kernel                      +<-+
  |                                              |
  +----------------------------------------------+
```

### Inter-process communication

- synchronous messaging; kernel copies payload from sender to recipient -> "rendezvous"/ "handoff"
    - single message copy
    - no queues
    - limits power of task: can't spam
    - receivers can coordinate when sender's (not) run -> helps mutual exklusion
- rules around messaging & task priority to avoid deadlocks & priority inversion
   - can only SEND to higher priority taskss
- Sending returns response: response code & data written into the provided buffer
- messages limited to 256 bytes
- leases: small descriptors that tell the memory to allow receiver to access part of sender's memory
  - e.g. for sending large messages over uart
  - max 255 lenses per send
- e.g. `read`:

```rust
fn read(task: TaskId, fd: u32, buffer: &mut [u8]) -> Result<usize, IoError> {
    let mut response = [0; 4];
    let (rc, len) = sys_send(
        task,
        0,
        &fd.to_le_bytes(),
        &mut response,
        &[Lease::from(buffer)],
    );
    if let Some(err) = IoError::from_u32(rc) {
        Err(err)
    } else {
        assert_eq!(len, 4);
        Ok(u32::from_le_bytes(&response))
    }
}
```

- Receiving side:
  - recv: gets next pending msg
  - reply: unblocks sender (optional response payload)

#### Notifications

- Limited async communication; alternative to sync messaging
- used for interrupts & signals
- task has 32b notification set that can be set
  - don't interrupt the task
  - checked with `recv` function
- used by the kernel to route hardware interrupts
- used by other tasks to notify without blokcing
  - e.g. to not block a server that needs to send data to a client
- "pingback" pattern

### Interrupts

- delivered as notifications
- task has exclusive control over any itnerrupts it handles
- tasks can use `irq_control` syscall to (un)mask ienrrupts
- some interrupts reserved for kernel

### Syscalls

- Supported:
  - IPC pimriitves:
    - send, receive, reply
    - acess to memory borrowed from senders
    - looking up the correct generation number for a task
  - Access to multiplexed per-task timer
  - Control of the current task's interrupt mask
  - Crashing current task
  -> operatiosn that every task should have access to
- **not** supproted:
  - checking state / fault info of a task
  - (re)starting a task
  - forcing faults on other tasks
  -> done by supervisor task
- `SEND`, `RECV`, `REPLY`, `SET_TIMER`, `BORROW_READ`, `BORROW_WRITE`, `BORROW_INFO`, `IRQ_CONTROL`, `PANIC`, `GET_TIMER`, `REFRESH_TASK_ID`, `POST`, `REPLY_FAULT

### Kernel IPC Interface

- `read_task_status`, `reinit_task`, `fault_task`, `read_image_id`, `reset`, `read_caboose_pos`, `get_task_dump_region`, `read_task_dump_region`

### Drivers

- unpriviledged; don't live in kernel
- hardware drivers exist as "servers"
- driver crate
  - interface for dealing with a device
  - may directly access hardware or do IPC calls to other server
- driver server:
  - wraps driver crate
  - provides IPC interface

## Other

- Website: <https://hubris.oxide.computer/>
> There are no operations for creating or destroying tasks at runtime, no dynamic resource allocation, no driver code running in privileged mode

- Comment on Reddit about Hubris vs Tock by Bryan Cantrill (Hubris Maintainer): <https://www.reddit.com/r/rust/comments/r5l97n/comment/hmonirr/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button>
> We spent some time with Tock; it has a very different design center: Tock uses dynamic loading extensively, where Hubris is static with respect to tasks; Tock is very asynchronous where Hubris is strictly synchronous; Tock has drivers in the same protection domain as the kernel (albeit partitioned by the system) where Hubris has drivers in disjoint projection domains. These aren't necessarily problems with one system or the other, but rather reflect their different design centers -- and thankfully, the world is big enough for many systems solving different problems! 

