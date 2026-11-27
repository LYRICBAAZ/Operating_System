# Day 30/30 — Thrashing

Day 28 introduced demand paging, and Day 29 covered how to pick a page to replace when memory is full. Today's lecture is the last one in this series, and it covers the failure that those ideas can cause. When a process doesn't have enough frames, it keeps faulting, and when enough processes do this, the whole system can slow to a crawl. This is **thrashing**. The lecture also covers two techniques to prevent it: the **working set model** and **page-fault frequency control**.

> **Note on numbering:** this post follows Lecture 30 in your notes ("Thrashing"), and it's Day 30 in this series. This is the final day of the 30-day plan.

---

## 1. What Is Thrashing?

- If a process **doesn't have enough frames** to support the pages it is actively using, it quickly **page-faults**.
- At that point, it must replace some page. But every page it has is in active use, so it **replaces a page that will be needed again right away**.
- The process then faults again, and again, and again. Each time, it replaces pages that it must bring back in immediately.
- This **high paging activity** is called **thrashing**.

> A system is **thrashing** when it **spends more time servicing page faults than executing processes**.

---

## 2. Thrashing and CPU Utilization

The notes plot **CPU utilization** against the **degree of multiprogramming**:

```
CPU utilization
    ▲
    │            ╭─────╮
    │         ╭──╯     ╰╮
    │      ╭──╯          ╰──
    │   ╭──╯               thrashing ◄──────►
    │ ╭─╯
    └──────────────────────────────────────► degree of multiprogramming
```

- At first, adding more processes **raises** CPU utilization, because more processes are available to run.
- Past a certain point, each process gets too few frames, so faults rise and CPU utilization **falls**.
- The region on the right, where utilization collapses, is **thrashing**.

This is the key trap. The OS might respond to low CPU utilization by starting **more** processes, which makes thrashing worse.

---

## 3. Technique 1: The Working Set Model

The working set model is based on the **locality model** from earlier lectures: at any time, a process uses a small group of pages that are close together in use.

> **Basic principle:** if the OS allocates enough frames to a process to hold its **current locality**, the process will fault **only when it moves to a new locality**.

- If the allocated frames are **fewer than the size of the current locality**, the process **is bound to thrash**. It can't keep its active pages in memory, so it faults constantly.

So the OS should give each process enough frames for its working set, and it should not run more processes than the frames can support.

### Worked Example (illustrative, not in the notes)

A process loops repeatedly over **5 pages**: 1, 2, 3, 4, 5, 1, 2, 3, 4, 5, and so on. This is its locality.

- With **5 frames**: the first five references each fault once, to load the pages. After that, every reference is a hit. Faults stop.
- With **4 frames**: under LRU, every reference faults. The page replaced is always the one needed next, so the process thrashes.

This matches the basic principle. Enough frames for the locality means few faults, and too few means constant faults.

---

## 4. Technique 2: Page-Fault Frequency (PFF)

- **Thrashing has a high page-fault rate.** So the OS should **control the page-fault rate** directly.
- The logic:
  - If the rate is **too high**, the process **needs more frames**.
  - If the rate is **too low**, the process **may have too many frames**, which wastes memory that other processes could use.
- **Method:**
  1. Set an **upper bound** and a **lower bound** on the desired page-fault rate.
  2. If the rate **exceeds the upper bound**, **allocate another frame** to the process.
  3. If the rate **falls below the lower bound**, **remove a frame** from the process.
- By keeping the page-fault rate between the two bounds, **thrashing can be prevented**.

### Worked Example (illustrative, not in the notes)

Suppose the bounds are 2% and 10% faults per reference.
- A process faulting on 15% of its references is above the upper bound, so the OS gives it another frame.
- A process faulting on 0.5% of its references is below the lower bound, so the OS takes a frame away and gives it to another process.

The exact numbers vary by system. The point is the feedback loop.

---

## 5. Comparing the Two Techniques

| | Working set model | Page-fault frequency |
|---|---|---|
| Based on | Locality of reference | Observed fault rate |
| Question it answers | Are there enough frames for the current locality? | Is the process faulting too much or too little? |
| Action | Allocate frames for the working set, or suspend processes | Add or remove a frame when the rate crosses a bound |

---

## Key Takeaways

- **Thrashing** happens when a process lacks the frames for its active pages, so it faults repeatedly. A system is thrashing when it spends more time on page faults than on execution.
- On the CPU utilization graph, utilization **rises**, peaks, and then **collapses** as the degree of multiprogramming increases.
- The **working set model** says a process runs well when it has frames for its **current locality**. It thrashes when it has fewer frames than that.
- **Page-fault frequency control** keeps each process's fault rate between an upper and a lower bound. It adds frames above the upper bound and removes them below the lower bound.
- Controlling the fault rate prevents thrashing.

---

## Course Wrap-Up: 30 Days of Operating Systems

| Day | Topic |
|---|---|
| 1 | Introduction to OS |
| 2 | Types of OS |
| 3 | Multitasking vs multithreading |
| 4 | Components of an OS |
| 5 | System calls |
| 6 | Boot process |
| 7 | 32-bit vs 64-bit OS |
| 8 | Storage devices and the memory hierarchy |
| 9 | Introduction to process (PCB, process table) |
| 10 | Process states and queues |
| 11 | Swapping, context switching, orphan and zombie processes |
| 12 | Scheduling, FCFS, convoy effect |
| 13 | SJF, priority, round robin |
| 14 | MLQ and MLFQ |
| 15 | Introduction to concurrency |
| 16 | Critical section problem |
| 17 | Condition variables and semaphores |
| 18 | Revision, Days 1 to 9 |
| 19 | Revision, Days 10 to 17 |
| 20 | Dining philosophers problem |
| 21 | Deadlock, part 1 |
| 22 | Deadlock, part 2 |
| 23 | Revision, Days 20 to 22 |
| 24 | Contiguous memory allocation |
| 25 | Free space management |
| 26 | Paging |
| 27 | Segmentation |
| 28 | Virtual memory, demand paging, page faults |
| 29 | Page replacement algorithms |
| 30 | Thrashing |

---

**That's the final day of the series.** Thank you for following along.
