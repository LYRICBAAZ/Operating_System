# Day 29/30 — Page Replacement Algorithms

Day 28 showed how a page fault brings a missing page in from swap space. But that step assumes a **free frame** exists. When memory is busy and every frame is in use, the OS has to **give up** one page to make room. Today's lecture covers which page to give up. The goal of every algorithm is the same: **minimize page faults**.

> **Note on numbering:** this post follows Lecture 29 in your notes ("Page Replacement Algorithms"), and it's Day 29 in this series. The next lecture is thrashing, which is Day 30.

---

## 1. Why Replacement Is Needed

- A **page fault** happens when a process accesses a page that isn't in a frame. The OS must bring the page from swap space into a frame.
- If a **free frame** exists, the page goes there.
- If the system is under high utilization and **all frames are busy**, the OS must **replace** one of the pages already in a frame with the new page.
- The **page replacement algorithm** decides which page to remove. The chosen page is swapped out of its frame, and the new page is swapped into the freed frame.

**Aim of every algorithm:** minimum page faults.

---

## 2. FIFO (First In, First Out)

- Allocate a frame to each page as it comes into memory, and **replace the oldest page**.
- **Easy to implement:** the pages are kept in a queue in arrival order.
- **Performance is not always good:**
  1. The page replaced may be an **initialization module** that was used long ago. This is a good replacement candidate.
  2. But the page may also hold a **heavily used variable** that was set up early and is still in use. Replacing it will cause another page fault soon.
- **Belady's anomaly:** for **LRU** and **Optimal**, adding more frames always reduces page faults. For **FIFO**, Belady found that adding frames can **increase** page faults in some cases. This is strange, and it's why FIFO can't be trusted to improve with more memory.

### Worked Example: Belady's Anomaly (not in the notes)

Reference string: **1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5**

| Frames | FIFO page faults |
|---|---|
| 3 | 9 |
| 4 | 10 |

With **4 frames**, FIFO faults **more** than with 3 frames. That's Belady's anomaly.

---

## 3. Optimal Page Replacement

- Look ahead in the reference string:
  1. If a page **is never referenced again**, replace it.
  2. Otherwise, replace the page that is **referenced farthest in the future**.
- **Lowest page fault rate** of any algorithm.
- **Difficult to implement:** the OS would need **future knowledge** of the reference string, which is essentially impossible. The notes compare it to **SJF scheduling**, which also needs future information.
- It's used as a **benchmark**: other algorithms are measured against it.

---

## 4. LRU (Least Recently Used)

- Use the **recent past as an approximation of the near future**. Replace the page that **has not been used for the longest period**.
- Can be implemented in two ways:

**(1) Counters**
- Associate a **time field** with each page-table entry.
- When a page is referenced, update its time.
- To replace, choose the page with the **smallest time value**.

**(2) Stack**
- Keep a **stack of page numbers**.
- Whenever a page is referenced, **remove it from the stack and put it on top**.
- So the **most recently used** page is always on top, and the **least recently used** page is always on the bottom.
- Entries can be removed from the middle of the stack, so a **doubly linked list** is used.

### Worked Example: 3 Frames (not in the notes)

Same reference string as above, with 3 frames:

| Algorithm | Page faults |
|---|---|
| FIFO | 9 |
| LRU | 10 |
| Optimal | 7 |

Optimal gives the fewest faults, as expected. LRU here is slightly worse than FIFO, which shows that no algorithm wins on every string.

---

## 5. Counting-Based Page Replacement

- Keep a **counter** of the number of references made to **each page**. This is called **reference counting**.

**(i) LFU (Least Frequently Used)**
- Actively used pages should have a **large** reference count.
- **Replace the page with the smallest count.**

**(ii) MFU (Most Frequently Used)**
- The argument: a page with the **smallest count** was probably **just brought in** and **hasn't been used yet**. So it's the one that should stay.
- **Replace the page with the largest count.**

**(iii)** In practice, **neither MFU nor LFU is common**, because the counts don't capture recent behavior well.

---

## 6. Comparison

| Algorithm | Replaces | Strength | Weakness |
|---|---|---|---|
| FIFO | Oldest page | Easy to implement | Can replace heavily used pages; Belady's anomaly |
| Optimal | Page used farthest in future | Fewest faults | Needs future knowledge; not implementable |
| LRU | Least recently used page | Good approximation of Optimal | Counters or stacks cost time and memory |
| LFU | Least frequently used page | Keeps busy pages | Old, once-busy pages keep high counts |
| MFU | Most frequently used page | Keeps new pages | Rarely common |

---

## Key Takeaways

- A **page replacement algorithm** runs only when there is **no free frame**. It picks a page to swap out, and the goal is to **minimize page faults**.
- **FIFO** is simple, but it replaces the oldest page even when that page is busy, and it shows **Belady's anomaly**.
- **Optimal** gives the **lowest fault rate**, but it needs future knowledge, so it's used only as a benchmark.
- **LRU** approximates Optimal by using recent history. It's implemented with **counters** or a **stack**.
- **LFU** and **MFU** use **reference counts**, and neither is common in practice.

---

**Next up — Day 30:** **Thrashing**: what happens when the system spends most of its time swapping pages instead of doing work, and how to avoid it.
