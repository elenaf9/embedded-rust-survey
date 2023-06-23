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

## Git repo

<https://github.com/oxidecomputer/hubris/blob/master/doc/interrupts.adoc>

- design is based on ARM


## Kernel Interface

### IPC

- Virtual kernel task:
  - "Illusion created by the kernel'
  - any message sent to virtual kernel task will be processed by kernel
  - accepts any message sent to `TaskId::KERNEL`
  - currently no leases/borrows 
  - kernel guarentees to respon sync, without blocking

### Syscalls 

- implemented by kernel
- arch specific
- use arch's superviso-call isntruction

## Server

- implement functionaility as server vs as crate
- servers are tasks
- use servers when: multiple clients; only one should be able to performan operation at a time
- use crate when: if you and anoither task will never be runnable at same time
- low-level: receive messages through sys calls
- high-level: `userlib::hl` wrappers for comppon patterns in server implementations
- ideally also a convenienve API wrapper crate

server impl example:
<https://github.com/oxidecomputer/hubris/blob/8651b98457c27a7eb7b5484feb3d2f2a7bc99231/drv/user-leds/src/main.rs#L182>
user leds:

```rs
impl idol_runtime::NotificationHandler for ServerImpl {
    fn current_notification_mask(&self) -> u32 {
        ...
    }

    fn handle_notification(&mut self, _bits: u32) {
        ...
    }
}

#[export_name = "main"]
fn main() -> ! {
    enable_led_pins();

    // Handle messages.
    let mut incoming = [0u8; idl::INCOMING_SIZE];
    let mut server = ServerImpl {
        blinking: Default::default(),
        blink_state: false,
    };
    loop {
        // Use generic server dispatch routine for servers that use notifications
        // This calls the sys_recv sys call, 
        // `handle_notification` if there are any, 
        // and the handler for received operation codes
        idol_runtime::dispatch_n(&mut incoming, &mut server);
    }
}
```

another example:
<https://github.com/oxidecomputer/hubris/blob/8651b98457c27a7eb7b5484feb3d2f2a7bc99231/drv/stm32fx-rcc/src/main.rs#L108>
```rs
#[export_name = "main"]
fn main() -> ! {
    let rcc = unsafe { &*device::RCC::ptr() };

    // Any global setup we required would go here.

    // Field messages.
    // Ensure our buffer is aligned properly for a u32 by declaring it as one.
    let mut buffer = [0u32; 1];
    loop {
        hl::recv_without_notification(
            buffer.as_bytes_mut(),
            |op, msg| -> Result<(), ResponseCode> {
                // Every incoming message uses the same payload type and
                // response type: it's always u32 -> (). So we can do the
                // check-and-convert here:
                let (msg, caller) =
                    msg.fixed::<u32, ()>().ok_or(ResponseCode::BadArg)?;
                let bus =
                    Bus::from_u32(msg / 32).ok_or(ResponseCode::BadArg)?;

                match op {
                    Op::EnableClock => {..}
                    Op::DisableClock => {..}
                    Op::EnterReset => {..},
                    Op::LeaveReset => {..},
                }

                caller.reply(());
                Ok(())
            },
        );
    }
}
```

### HAL

- chip/ peripgeral definitons & support fiels for chips
- drivers:
  - <driver>-api: platform independent API Wrapper for a server(= interface to server task)
  - <driver>-devices: Devices drivers for <drivers> (e.g. all i2c device drivers)
  - <platform>-server: server for a specific platform
  - <platform>-<driver>(-{api,server}): same as above, for specific platforms
    - rely on PAC crates
- sys/kerne/arch/: arch specific kernel logic
- feature flags in drivers

From FAQ: <https://github.com/oxidecomputer/hubris/blob/6c0c483c47f5050696032b7166f4b650ca66c993/FAQ.mkdn?plain=1#L238>

- Hubris requires MPU
- targets ARM-M processor; RISC-V not supported
- does not allow embedded-hal to write tasks:
  - embedded hal (e.g. cortex-m) assumpt that it runs in processor's privilege mode and can disable interrupts
  - unsound in unprivileged mode
  - using lower level PACs is generally safe, apart from the parts that rely on global mutexes like `Peripherals::take()`
- no async support


## Talk

<https://talks.osfc.io/osfc2021/talk/JTWYEH/>

- tasks are separatly compiled and isolated from one another using the mpu
  - separate parts of RAM and flash
- drivers run in tasks, are unpriviledged, can claim interrupts from kernel
- "agressivly static" for all allocatable or routable ressources (memory, interrupts, etc)
- everything at compile time
- only task in-place re-initialization supported
- syncronocity -> simplicity
  - allows cross-task memory lending
- state machines:
  - describe computer how to perform a flowchart 
  - conceptional indirection
  - enable asynchronity; resource limitation; multiplex
  - tasks see interrupts as synchronous events
- how is asynchronity handled? e.g. through events
  - state machines 
- Types:
  - Encode Task States
    - Usually: flatten all necessary info into the tasks struct: `Task { state, previous_state, fault_kind, fault_address, ..}`
    - Instead: Use task state `enum TaskState {Healthy(SchedState), Faulted {FaultInfo, SchedState} }`
  - Implement security parameters in function signature
    - treat input validation as a parsing problem
    - reflect in the type that validation & parsing has occured  
    - Usually: pointers get validated, types don't change; no support from the kernel; where should these checks occur?
    - Instead: change type when doing a check
  -> not common yet in systems ecosystem because not possible in C

### Humility

- Debugger developed alongside with Hubris
- "Application Debugger Co-Design"

## Networking stack

### task::net for STM32H7

- Use smoltcp for Network IPC server implementation
- only UDP
- `smol::phy::Device` implementations:
  - `VLanEthernet` & `Smol`:
    - Wrapper around stm32h7 ethernet driver
- The task's main loop continously polls the server, which wraps the Devices, and smoltcp sockets and interface and polls said interface
  - leverage socket waker API: iterate through sockets, if pending incoming or outgoing packet notify ethernet driver 

Sending a packet:

```mermaid
sequenceDiagram
  participant task::Client as C
  participant task::Server as Svr
  participant smoltcp::Socket as Soc
  participant smoltcp::Interface as I
  participant drv::Ethernet as D
    C-->Svr: send_packet()
    Svr-->Soc: send()
    Soc-->Soc: <enqueue packet in tx buffer>
    Note over Svr,I: In main loop
    Svr-->I: poll()
    I-->Soc: dispatch()   
    Soc-->Soc: <dequeue packet from tx buffer>
    Soc-->Svr: <packet>
    Svr-->D: transmit()
```




