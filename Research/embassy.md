# Embassy

<https://embassy.dev/book/dev/index.html>


Problem:

- operations are asynchronous, we don't want to block on them
    - either use threads
    - or non-blocking operations, that need to be polled
- in Rust: 
    - use async-await for non-blocking operations -> future
    - executor (scheduler) decides what future to execute
- **Alternative to RTOS**

## Embassy Executor

- Fixed number of tasks, allocated at start-up
- implementations of blocking and non-blocking APIs
- provides system timer
- for <1ms block instead of switch due to context switching costs
- `#[embassy_executor::task] async fn ..()` declare tasks that should be run by application
- `#[embassy_executor_main] async fn ..()` entry pointer of app
    - Creates an Embassy Executor
    - Initializes the microcontroller HAL to get the Peripherals
    - Defines a main task for the entry point
    - Runs the executor spawning the main task
- Interrupt driven scheduling: `#[interrupt]` macro  
    - Only notify application on interrupt
    - (requires platform-specific impl of an interrupt)
    - `embassy_cortex_m::interrupt`
- Use HAL instead of interrupt macro:
    - main macro inits peripheries
    - Types internally configure interrupt handler
- Executor features:
    - No alloc, no heap needed. Task are statically allocated.
    - No "fixed capacity" data structures, executor works with 1 or 1000 tasks without needing config/tuning.
    - Integrated timer queue: (e.g. `Timer::after(Duration::from_secs(1)).await`)
    - No busy-loop polling: CPU sleeps, using interrupts or WFE/SEV.
    - Efficient polling: a wake will only poll the woken task, not all of them.
    - Fair: a task can’t monopolize CPU time even if it’s constantly being woken. All other tasks get a chance to run before a given task gets polled for the second time.
    - Creating multiple executor instances is supported, to run tasks with multiple priority levels. This allows higher-priority tasks to preempt lower-priority tasks.

## Embassy HAL

- Embassy projects maintains HALs for selected hardware
- Periphery Access Layer (PAC)
    - Access peripherals and registers
    - unsafe code
    - Should usually not be used directly, except for functionality that is not provided by the upper layer
    - Board specific
- HAL is one layer on top of PAC
- Hardware specific

