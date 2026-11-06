# Day 18/30 — Revision Part 1: Days 1 to 9

This is a revision day. Days 1 to 9 covered the foundations: what an OS is, how it's structured, how it starts, and what a process is. Read this as a one-page recap. Each section ends with a short self-check, so you can test yourself before moving on.

---

## Day 1: Introduction to Operating Systems

- **Application software** performs a task for the user. **System software** operates and controls the computer and provides the platform for applications.
- An **OS** manages all hardware and software resources and provides a convenient environment by **hiding complexity**.
- Without an OS: bulky apps, one app hogging resources, and no memory protection.
- Core OS functions: **access to hardware**, **interface** between user and hardware, **resource management (arbitration)**, **abstraction**, and **isolation and protection**.

**Self-check:** Why does an OS need to hide hardware complexity? (So programmers don't write hardware code in every app.)

---

## Day 2: Types of Operating Systems

- All OS types chase three goals: **maximum CPU utilization**, **less starvation**, and **higher-priority jobs first**.
- **Single process OS**: one process at a time, run to completion or wait.
- **Batch processing OS**: the operator groups jobs with similar needs into batches. Priorities can't be set, starvation is possible, and the CPU is idle during I/O.
- **Multiprogramming OS**: keep several jobs in memory, so the CPU switches when a job waits. Single CPU, reduces idle time.
- **Multitasking OS**: a logical extension of multiprogramming with **time sharing**, which improves responsiveness.
- **Multi-processing OS**: more than one CPU in one computer. More reliable, better throughput.
- **Distributed OS**: many loosely connected, autonomous computers coordinated as one system.
- **Real-time OS**: computations must finish within strict time limits (air traffic control, robots).

**Self-check:** What does multiprogramming fix that a single process OS doesn't? (CPU idle time during I/O.)

---

## Day 3: Multitasking vs Multithreading

- **Program**: compiled code on disk. **Process**: a program in execution, in RAM. **Thread**: a light-weight path of execution inside a process.
- **Multitasking**: several isolated **processes** share a CPU. Each has its own memory.
- **Multithreading**: several **threads of one process** share its memory and resources.
- **Thread context switch** is faster than process context switch, because the address space isn't switched and the cache is preserved.

**Self-check:** Which one shares memory, multitasking or multithreading? (Multithreading.)

---

## Day 4: Components of an OS

- **User space** runs apps with no privileged hardware access. **Kernel space** is privileged. The **kernel** is the bridge.
- The **kernel** is the heart of the OS, the first part loaded at startup, and controls who can access hardware.
- The **shell** is the command interpreter. GUI and CLI are user interfaces.
- The kernel's four management jobs: **process, memory, file, and I/O management**.
- I/O techniques:
  - **Spooling**: queue jobs on disk so a slow device works through them one by one (print spooling).
  - **Buffering**: temporary memory storage while data moves between devices or programs at different speeds (video buffering).
  - **Caching**: keep frequently used data in a small, fast store.
- **Kernel types**:
  - **Monolithic**: everything in the kernel. Fast, but one crash brings the whole kernel down. Linux, MS-DOS.
  - **Micro**: only memory and process management in the kernel. Reliable and stable, but slower because of mode-switch overhead. L4, MINIX.
  - **Hybrid**: a mix. macOS, Windows NT.
- **Device driver**: software that lets the kernel talk to a specific device.
- **IPC** (inter-process communication) crosses the protection boundary through **shared memory** (fast, needs coordination) or **message passing** (safer, slower).

**Self-check:** Which kernel type is most reliable but slowest? (Micro.)

---

## Day 5: System Calls

- A **system call** is the only sanctioned way for a user program to request a kernel service it can't perform itself.
- Transitions from user mode to kernel mode happen through a **software interrupt**.
- Wrappers like `mkdir` and C library functions (glibc) call system calls underneath.
- Five categories: **process control, file management, device management, information maintenance, and communication**.
- Examples: Unix `fork()`, `open()`, `pipe()`; Windows `CreateProcess()`, `CreateFile()`, `CreatePipe()`.

**Self-check:** Name one Unix and one Windows system call for creating a process. (`fork()` and `CreateProcess()`.)

---

## Day 6: Boot Process

- **Firmware** is software stored inside hardware, usually in non-volatile memory. Example: keyboard firmware turns a key press into a signal.
- Boot chain: power on → CPU finds **BIOS** (or **UEFI** on modern PCs) → **POST** tests and initializes hardware (stops with an error if something essential, like RAM, is missing) → firmware finds the **bootloader** in the **MBR** or **EFI system partition** → bootloader loads the **kernel** → user space starts.
- Bootloaders: Windows Boot Manager (`Bootmgr.exe`), GRUB (most Linux), `boot.efi` (macOS).

**Self-check:** What does POST do? (Tests and initializes hardware.)

---

## Day 7: 32-bit vs 64-bit OS

- The "bit" number is the width of the **CPU registers**, also called the **word size**.
- Addresses: 32-bit gives 2^32 addresses, which is **4 GB** of RAM. 64-bit gives 2^64, far beyond any real RAM.
- A 64-bit processor handles **8 bytes** per instruction cycle versus **4 bytes** for 32-bit.
- A **64-bit CPU** can run both 32-bit and 64-bit OS. A **32-bit CPU** runs only 32-bit OS.
- Calculations happen in registers: RAM → registers → ALU → registers → RAM.

**Self-check:** Why can't a 32-bit OS use 8 GB of RAM? (Only 2^32 addresses, so 4 GB is the ceiling.)

---

## Day 8: Storage Device Basics (Memory Hierarchy)

- **Primary memory** (CPU can access directly): **registers** (smallest, fastest, inside the CPU), **cache** (frequently used data), **main memory / RAM**.
- **Secondary memory** (long-term storage): electronic disks (SSD), magnetic disks (HDD), optical disks, magnetic tape.
- Comparison:
  - Primary: costly, faster, smaller, **volatile**.
  - Secondary: cheaper, slower, larger, **non-volatile**.
- The hierarchy exists because no single storage type is fast, big, cheap, and permanent at once.

**Self-check:** Why is secondary memory non-volatile? (Data stays after power is off.)

---

## Day 9: Introduction to Process

- A **process** is a program under execution, residing in RAM. A **program** is compiled code ready to run, stored on disk.
- Steps the OS takes to create a process: load code and static data, allocate the **runtime stack**, allocate **heap** memory, set up I/O, and hand control to `main()`.
- A process has four memory sections: **text (code), data (globals and statics), heap (dynamic memory), stack (locals, function arguments, return values)**.
- Each process has a unique **Process ID (PID)**. The OS tracks all processes in a **process table**, and each entry is a **PCB (Process Control Block)**.
- PCB fields: process ID, **program counter**, **process state**, **priority**, **registers**, list of open files, list of open devices.
- Registers are saved in the PCB when a process is switched out, and restored when it runs again. The PCB lives in main memory.

**Self-check:** What is the difference between a program and a process? (Program: compiled code on disk. Process: that code running in memory.)

---

## Quick Recall Table (Days 1–9)

| Topic | One-line memory hook |
|---|---|
| OS | Manages resources, hides complexity |
| OS types | Single → batch → multiprogramming → multitasking → multi-processing → distributed → real-time |
| Multithreading | Threads share memory, processes don't |
| Kernel | Heart of OS; monolithic fast, micro reliable, hybrid balanced |
| System call | The door from user mode to kernel mode |
| Boot | Firmware → POST → bootloader → kernel |
| 32 vs 64 bit | Register width; 4 GB limit vs huge |
| Memory hierarchy | Fast and small close to CPU; big and cheap far away |
| PCB | The OS's record of each process |

---

**Next up — Day 19:** Revision Part 2, covering Days 10 to 17.
