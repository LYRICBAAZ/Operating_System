# Day 28/30 — Virtual Memory, Demand Paging & Page Faults

Day 26 and 27 assumed that a whole process must be in physical memory before it can run. That limits every program to the size of RAM, and it wastes memory on parts of a program that aren't being used. **Virtual memory** removes that assumption. A process can run while only part of it is in RAM, and the rest stays on disk until it's needed. **Demand paging** is the main way to implement this, and a **page fault** is the event that brings a missing page in.

> **Note on numbering:** this post follows Lecture 28 in your notes ("What is Virtual Memory? || Demand Paging || Page Faults"), and it's Day 28 in this series. The next lecture is page replacement algorithms, which is Day 29.

---

## 1. What Is Virtual Memory?

> **Virtual memory** is a technique that allows the execution of processes that are not completely in memory. It gives the user the illusion of having a very big main memory, by treating part of secondary memory as if it were main memory. That part of secondary memory is called the **swap space**.

- **Advantage:** a program can be **larger than physical memory**.

---

## 2. Why Partial Execution Is Needed

- Instructions must be in physical memory to be executed. Without virtual memory, that **limits a program to the size of physical memory**.
- In many cases, the **entire program is not needed at the same time**. Error-handling code, rarely used options, and large data arrays are often idle.
- So the goal is to execute a program that is only **partially** in memory.

### Benefits (from the notes)

1. **No constraint from physical memory.** A program is no longer limited by how much physical memory is available.
2. **More programs at once.** Each program takes less physical memory, so more programs can run at the same time. This increases **CPU utilization and throughput**.
3. **Both sides benefit.** Running a program that isn't entirely in memory helps both the **system** and the **user**.

The programmer also gets a very large virtual memory, even when only a smaller physical memory is available.

---

## 3. Demand Paging

> **Demand paging** is a popular method of **virtual memory management**.

- Pages of a process that are **least used** get stored in secondary memory.
- A page is copied into main memory **only when its demand is made**, that is, when a **page fault** occurs.
- When memory is full, **page replacement algorithms** decide which page to remove. These are covered in Day 29.

### Lazy Swapper

- Instead of swapping in the whole process, the system uses a **lazy swapper**.
- A lazy swapper **never swaps a page into memory unless that page will be needed**.

### Pager vs Swapper

- A process is viewed as a **sequence of pages**, not one large contiguous address space.
- So the term **swapper** is technically incorrect in this context. A **swapper** manipulates **entire processes**. A **pager** deals with **individual pages** of a process.

---

## 4. How Demand Paging Works

1. When a process is to be swapped in, the **pager guesses** which pages will be used.
2. Only those pages are brought into memory. Pages that won't be used are never read.
3. This decreases the **swap time** and the **amount of physical memory needed**.
4. The **valid-invalid bit** in each page-table entry tells the system which pages are in memory:
   - **Valid-invalid bit 1:** the page is **legal and in memory**.
   - **Valid-invalid bit 0:** either the page is **not valid** (not in the process's logical address space), or it is **valid but currently on disk**.

If a process never accesses an invalid-bit page, it runs successfully, and the pages on swap space are never needed.

---

## 5. Page Faults

- If the process tries to access a page that isn't in memory, the access hits a page marked invalid. This causes a **page fault**.
- The paging hardware sees the invalid bit for a demanded page, and it causes a **trap to the OS**.

### Steps to Handle a Page Fault

1. **Check an internal table** (kept in the process's PCB) to see whether the reference was **valid** or an **invalid memory access**.
2. **If invalid,** the process gets an **exception** and is terminated.
   **If valid,** the pager swaps in the page.
3. **Find a free frame** from the free-frame list.
4. **Schedule a disk operation** to read the desired page into that frame.
5. **When the disk read completes,** update the page table, so the page is marked as now in memory.
6. **Restart the instruction** that was interrupted by the trap. The process can now access the page as if it had always been in memory.

### Worked Example (illustrative, not in the notes)

From the notes' page-table diagram, a process has pages A to H in logical memory. Pages A, C, and F are in physical memory, mapped to frames 4, 6, and 9 with valid bits 1. Pages B, D, E, G, and H are on the swap space, with valid bit 0.

Suppose the process touches **page D**, which has valid bit 0:
- The hardware traps to the OS.
- The OS checks the PCB. D is a valid page, so the reference is legal.
- The OS finds a free frame, say frame 0.
- The OS reads D from swap space into frame 0.
- The page table entry for D is updated to frame 0 with valid bit 1.
- The interrupted instruction restarts, and this time D is found in memory.

---

## 6. Pure Demand Paging

- In the extreme case, a process can start executing with **no pages in memory**.
- The OS sets the instruction pointer to the first instruction of the process. That instruction's page isn't in memory, so the process **immediately faults**, and the page is brought in.
- **Rule:** never bring a page into memory until it's required.
- Pure demand paging works reasonably well because of **locality of reference**. Programs tend to use the same nearby pages repeatedly, so faults happen less often than the worst case.

---

## 7. Advantages and Disadvantages

**Advantages of virtual memory:**
1. The **degree of multiprogramming** increases.
2. Users can run **large applications** with less real physical memory.

**Disadvantages of virtual memory:**
1. The system can become **slower**, because swapping takes time.
2. **Thrashing** may occur. This is when the system spends so much time swapping pages that very little useful work gets done. *(Thrashing is named in the notes. Its details come in later lectures.)*

---

## Key Takeaways

- **Virtual memory** lets a process run while only part of it is in physical memory. The rest sits in **swap space** on disk.
- **Demand paging** loads a page only when it's needed. A **lazy swapper** never brings in an unneeded page.
- The **valid-invalid bit** in the page table shows whether a page is legal and in memory.
- A **page fault** traps to the OS. The OS checks the reference, loads the page into a free frame, updates the page table, and **restarts the instruction**.
- **Pure demand paging** starts a process with no pages loaded, and it relies on **locality of reference** for good performance.
- The costs are slower swapping and the risk of **thrashing**.

---

**Next up — Day 29:** **Page Replacement Algorithms**: what the OS does when there is no free frame, and how it chooses which page to remove.
