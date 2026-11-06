# Day 19/30 — Revision Part 2: Days 10 to 17

This is the second half of the revision. Days 10 to 17 covered process life cycles, scheduling, concurrency, and synchronization. This is the densest part of the course, so the formulas and comparison tables matter most here. Each section ends with a self-check.

---

## Day 10: Process States & Queues

- Five states: **new, ready, running, waiting, terminated**.
- Transitions:
  - New → Ready: **admitted**
  - Ready → Running: **scheduler dispatch**
  - Running → Ready: **interrupt** (time quantum ends)
  - Running → Waiting: **I/O or event wait**
  - Waiting → Ready: **I/O or event completion** (goes to ready first, not running)
  - Running → Terminated: **exit**
- Queues: **job queue** (new, on disk), **ready queue** (ready, in memory), **waiting queue** (waiting for I/O).
- **Long-Term Scheduler (LTS)**: job queue → memory. Controls the **degree of multiprogramming**.
- **Short-Term Scheduler (STS)**: ready queue → CPU. Runs very often.
- **Dispatcher**: carries out the handover to the chosen process.

**Self-check:** Which scheduler controls the degree of multiprogramming? (LTS.)

---

## Day 11: Swapping, Context Switching, Orphan & Zombie

- **Medium-Term Scheduler (MTS)** swaps processes out to secondary storage (**swap-out**) and brings them back (**swap-in**) to reduce the degree of multiprogramming when memory is tight.
- **Context switch**: save the old process's state into its PCB, load the new one's state. It's pure overhead. Its speed depends on memory speed and the number of registers to copy.
- **Orphan process**: parent has terminated, child is still running. It's adopted by **init**, the first process of the OS.
- **Zombie process**: child has finished, but its entry remains in the process table until the parent calls `wait()` and reads the exit status. This is called **reaping**.

**Self-check:** Who adopts an orphan process? (Init.)

---

## Day 12: Scheduling Basics, FCFS, Convoy Effect & Starvation

- **CPU scheduler** (STS) picks a process from the ready queue whenever the CPU is idle.
- **Non-preemptive**: the CPU stays with a process until it terminates or waits. **Preemptive**: the CPU can also be taken when the time quantum expires.
- Five goals: max CPU utilization, min TAT, min WT, min response time, max throughput.
- **Formulas:**
  - **TAT = CT − AT**
  - **WT = TAT − BT**
- **FCFS**: first come, first served, non-preemptive. Simple, but the arrival order changes the average wait a lot.
- **Convoy effect**: many short processes wait behind one long process that holds the resource. Common in FCFS. It causes poor resource management.
- **Starvation**: one process is ignored for a long time. Common in priority scheduling. Fixed by **aging**.

**Self-check:** A process has AT = 2, CT = 10, BT = 5. What is its TAT and WT? (TAT = 8, WT = 3.)

---

## Day 13: SJF, Priority & Round Robin

- **SJF (Shortest Job First)**: picks the shortest burst time. Burst times must be **estimated**.
  - Non-preemptive: runs until done.
  - Preemptive (SRTF): a shorter new arrival can preempt. Lowest average wait, less starvation, no convoy effect.
- **Priority scheduling**: each process has a priority at creation. **SJF is a special case** where priority is inversely proportional to burst time.
  - Starvation can happen. Fix: **aging**, which raises the priority of long-waiting processes.
- **Round Robin (RR)**: preemptive FCFS with a **time quantum**. Fair, very low starvation, no convoy effect. Designed for time-sharing.
  - Small TQ → more context switches and more overhead.
  - Large TQ → behaves more like FCFS.

**Self-check:** Which algorithm is designed for time-sharing systems? (Round Robin.)

---

## Day 14: MLQ & MLFQ

- **Process types**: user (foreground, background), system; CPU-bound and I/O-bound.
- **MLQ (Multilevel Queue)**: ready queue split into fixed queues, each with its own algorithm (for example SP → RR, IP → RR, BP → FCFS). A process is **permanently** assigned to one queue. Scheduling between queues is fixed-priority preemptive.
  - Problems: starvation of lower queues, convoy effect in FCFS queues, inflexibility.
- **MLFQ (Multilevel Feedback Queue)**: processes **move between queues**.
  - A process that uses too much CPU is **demoted**.
  - I/O-bound and interactive processes stay high.
  - **Aging** promotes a long-waiting process, which prevents starvation.
  - Sample design: TQ = 2, TQ = 4, TQ = 8, then FCFS.

**Self-check:** What is the key difference between MLQ and MLFQ? (In MLQ a process stays in its queue. In MLFQ it can move.)

---

## Day 15: Introduction to Concurrency

- **Concurrency**: multiple instruction sequences executing at the same time.
- **Threads**: light-weight, independent paths of execution. Each has its own **program counter** and **stack**. They share the **heap, data, and text**.
- **Thread control block (TCB)** is the thread-level version of the PCB.
- **A single CPU gains nothing from multithreading.** Real parallelism needs more than one CPU.
- Benefits: responsiveness, resource sharing, economy, scalability on multiprocessors.

**Self-check:** Which parts of memory do threads share? (Heap, data, and text. Each has its own stack.)

---

## Day 16: Critical Section Problem

- **Critical section**: code that accesses shared resources and writes to them.
- **Race condition**: two or more threads change shared data at the same time, and the result depends on scheduling. `count++` is really `temp = count + 1; count = temp;`, so updates can be lost.
- Solutions:
  - **Atomic operations**: run the critical code as one indivisible step.
  - **Mutex / locks**: only one thread in the critical section at a time.
  - **Semaphores** (Day 17).
- A **simple flag variable does not solve it**, because checking and setting the flag are two separate steps.
- **Peterson's solution**: a software algorithm using `flag[2]` and `turn`. It works for **only two** processes or threads.
- **Lock disadvantages**: contention, deadlock, debugging difficulty, and starvation of high-priority threads.

**Self-check:** Why is `count++` not safe with two threads? (It's a read, an add, and a write. The steps can interleave and lose an update.)

---

## Day 17: Condition Variables & Semaphores

- **Condition variable**: lets a thread wait until a condition occurs. It works with a lock. The thread releases the lock while waiting and re-acquires it when woken. Its main purpose is to **avoid busy waiting**.
- **Semaphore**: an integer equal to the number of available resources.
  - Allows multiple threads into a finite-resource section. A mutex allows one at a time.
  - **Binary semaphore**: 0 or 1, like a mutex (but with no owner).
  - **Counting semaphore**: any non-negative range, for a finite number of instances.
- **wait()** decrements the value, and if the value goes negative, the process **blocks** into a waiting queue. **signal()** increments the value and wakes a blocked process into the ready queue.

**Self-check:** A semaphore starts at 2. Three threads call `wait()`. What happens to the third? (Its value goes to −1, so it blocks.)

---

## Scheduling Comparison (Days 12–14)

| Algorithm | Preemptive? | Starvation | Convoy effect | Notes |
|---|---|---|---|---|
| FCFS | No | Low | Yes | Simple, order-dependent |
| SJF | No | Possible | Yes | Needs burst-time estimates |
| PSJF (SRTF) | Yes | Less | No | Lowest average wait |
| Priority | Depends | Possible | Yes | Fix with aging |
| Round Robin | Yes | Very low | No | Fair; TQ controls overhead |
| MLQ | Yes | Possible | Yes | Fixed queues |
| MLFQ | Yes | Less (aging) | Yes | Processes move between queues |

---

## Key Formulas to Remember

- **TAT = CT − AT**
- **WT = TAT − BT**
- **Average WT = sum of WT / number of processes**

---

**Next up — Day 20:** the Dining Philosophers Problem, followed by Day 21: Deadlock.
