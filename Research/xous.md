# Xous Operating System

## Xous Book

<https://betrusted.io/xous-book/>

- microkernel OS for Betrusted, as secure & private communication system
  - targets Precursor Hardware (RISC-V)
- own rust target `riscv32imac-unknown-xous-elf`
- requires MMU: <https://xobs.io/announcing-xous-the-betrusted-operating-system/>

### Architecture

- "microkernel message-passing OS"
  - similar to BackBerry QNX
- collect of small, single puporse servers
  - respond to messages
  - contains central loop to receive messages
  - while waiting for message, release processing capacity
  - every application is implemented as a server
- kernel:
  - delivers messages to servers
  - allocated processing time to servers
  - transfers memory ownership from one server to another
  - "well known" servers:
    - xous-name-server: maintains list of registered all servers by name
    - ticktimer-server: time & timeout related services
    - xous-log-server: loggin services
    - timeserverpublic: RT clock services

#### Messages/ IPC

- Connection ID: "delivery address" for recipient sevrer
- Opcode: specifies oepration provided by recipient sevrer
- two flavors:
  - scalar messages: simple & fast, 4x u32 sized args
  - memory messages: larger structures: transmit page-sized (4096B) memory chunks
    - returned to callers on drop
    - structus can be serialized into bytes and send as memory messages
- non-synchronizing: pass ownership of message mempory from sender to receipient
  - asynchronous messages: non-sync message for which kernel will amend a "return token"; recipient will reply with non-sync message to return token
- synchronous messages: blocking untill recipient responds; message memory only lent to reciepient (r or rw)
  - sync message with deferred response: recipient server parks request until original request can be satisifed; recipient not allowed to block
- connect to server via `xous::connect`, by name/ server id
- disconnecting from a server is unsafe

### Server Architecture

- "basically just a 128-bit ID number that serves as a mailbox for incoming messages"
    - limit of 128 servers
- a thread can have zero, one or multiple servers
- if a message is received, thread is unblocked
- round-robin pre-emptive scheduler + schedule if a message is received
-  Synchronization provided via Ticktimer server
  - mutexes, process sleeping, condvars
- expected organzation:
  - `lib.rs`: caller-side API that formats native rust data into IPC messages
  - `main.rs`: server-side API that unpacks IPC messages and acts on them
  - `api.rs`: data structures & definitions shared between caller and callee

### Kernel

- processes are isolated; can only interact via messages
- dribvers implememnted as serversa
- Syscalls:
  - `MapMemory` optain memory (either specified physical address, or random)
  - `UnmapMemory` free memory
  - `ClaimInterrupt`: allocate interrupt
    - specifies address of function to call
    - interrupts can not be disabled; only inside interrupt handler
    - only limited set of functions that can be called

### Memory management

- can allocated physical or virtual memory
- liballoc: bundles heap management

#### Processes

<https://github.com/betrusted-io/xous-core/blob/main/docs/processes.md>

- processes have their own address space; heavyweight
- potentially multiple thread in one process
- no priorities / runqueue
  - multitasking is co-operative
  - round-robin through `susres` server for pre-emption interrupt
  -> pre-emption implemented entirely in userspace
- can start message servers:
    - `server_register`, `server_receive`, `server_reply`, `message_send`
- created through `process_create` syscall


#### Other notes

- unsafe functions that are exposed to the user


## Hardware abstraction

<https://github.com/betrusted-io/xous-core/tree/main/utralib>

- hardware entirely memory-mapped
- utralib: map hardware target specification to set of physical memory locations used by the hardware
  -> platform-specific artifacts
  - derived from SVD files
- support different arch's:
  - arm, riscv, hosted
  - cfg-flag gated
  - xous-rs (userspace-library) implement types `ProcessArgs`, `ProcessInit`, `ProcessKey`, `ProcessStartup`, `ThreadInit`
    - syscalls => platform specific representation
    - processes
    - nameserver
    - `carton`: warp object for shipping accross kernel boundary -> to be sent as message
  - kernel-rs: support different platforms & arch's
    - handle syscalls

### HAL servers

<https://github.com/betrusted-io/xous-core/tree/main/services>
- Use `utralib` for hardware specific memory locations

- `com`: manages requests to and from the EC
- `graphics-server`: manages the frame buffer and basic drawing primitives. Talks to the MEMLCD
- `keyboard`: key matrix management; debounce; ScanCode conversion. Keyboard layouts (qwerty, dvorak, azerty, qwertz, braille) are interpreted in this server
- `trng`: manages the TRNG hardware, provides TRNGs for other processes
- `llio`: manages I2C, RTC, GPIO, pin interrupts, soft reboot, and power pins. Also home for info, build IDs, etc.
- `codec`: basic buffering of frames into and out of the audio CODEC
- `jtag`: manages the JTAG interface (used for key fusing)
- `keys`: management of cryptographic key store
- `engine25519`: Engine25519 interface
- `sha512`: SHA512 interface
- `spinor`: manages the erasure and programming of the SPINOR
- `ram`: manages allocation of RAM for applications
- `scheduler`: manages thread scheduling and priorities

