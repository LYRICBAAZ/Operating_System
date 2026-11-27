# Day 25/30 — Free Space Management

Day 24 showed the main problem with contiguous allocation: dynamic partitioning leaves free holes scattered through memory, and a new process may not fit in any single hole even when the total free space is large enough. Today covers the two halves of the fix. First, **compaction** (defragmentation) squeezes the holes together. Second, the OS needs a way to **represent** the free holes and **choose** one for each new request, which is where the placement algorithms come in.

> **Note on numbering:** this post follows Lecture 25 in your notes ("Free Space Management"), and it's Day 25 in this series. The next lecture is paging and non-contiguous memory allocation, which is Day 26.

---

## 1. Defragmentation (Compaction)

- Dynamic partitioning suffers from **external fragmentation**.
- **Compaction** reduces the probability of external fragmentation.
- How it works:
  - All the **free partitions** are made **contiguous**.
  - All the **loaded partitions** are moved together.
- Result: the free partitions merge into one large block, which can be allocated to new processes. This lets the system store **bigger processes** in memory.
- The technique is also called **defragmentation**.
- **Cost:** compaction reduces efficiency, because all the free space has to be moved from several places into a single place, and processes have to be copied to new addresses.

```
Before:  [P1][ free 2MB ][P2][ free 3MB ][P3][ free 1MB ]
After:   [P1][P2][P3][ free 6MB ]
```

*(The second line is a simplified illustration of the idea, not a figure from the notes.)*

---

## 2. How Free Space Is Represented

- Free holes in memory are represented by a **free list**, which is a **linked list** data structure.
- Each node describes one hole: where it starts and how big it is.
- When a process is loaded or ends, the OS updates this list.

---

## 3. Satisfying a Request of Size n

When a process needs `n` units of memory, the OS searches the free list for a hole big enough. Four algorithms do this search differently.

### (a) First Fit

- Allocate the **first hole that is big enough**.
- Simple and easy to implement.
- Fast, because the search stops at the first match.

### (b) Next Fit

- An enhancement of First Fit.
- The search **starts from the last allocated hole**, not from the beginning of the list.
- Has the same advantages as First Fit.

### (c) Best Fit

- Allocate the **smallest hole that is big enough**.
- Leaves the **least internal fragmentation** in the chosen hole.
- **Drawbacks:** it tends to create many very small holes, which causes **major external fragmentation**. It is also **slow**, because the whole free list must be searched.

### (d) Worst Fit

- Allocate the **largest hole that is big enough**.
- Leaves **larger leftover holes**, which may still accommodate other processes.
- **Drawback:** it is **slow**, because the whole free list must be searched.

---

## 4. Worked Example (not in the notes)

Free holes, in list order: **100 KB, 500 KB, 200 KB, 300 KB, 600 KB**. A new process requests **212 KB**.

| Algorithm | Hole chosen | Why |
|---|---|---|
| First Fit | 500 KB | First hole in the list that is ≥ 212 KB |
| Best Fit | 300 KB | Smallest hole that is ≥ 212 KB (leftover 88 KB) |
| Worst Fit | 600 KB | Largest hole (leftover 388 KB) |

Next Fit depends on where the last allocation happened. If the search last ended at the 500 KB hole, Next Fit also picks 500 KB.

---

## 5. Comparison

| Algorithm | Picks | Speed | Main effect |
|---|---|---|---|
| First Fit | First big enough | Fast | Simple, may split early holes |
| Next Fit | First big enough, from last position | Fast | Spreads allocations across memory |
| Best Fit | Smallest big enough | Slow (full scan) | Least leftover per allocation, but many tiny holes |
| Worst Fit | Largest big enough | Slow (full scan) | Keeps big holes available, but breaks up large areas |

---

## Key Takeaways

- **Compaction** (defragmentation) moves loaded processes together so free space becomes one block. It fixes external fragmentation, but costs time.
- The OS keeps track of free holes in a **free list**, implemented as a **linked list**.
- **First Fit** is fast and simple. **Next Fit** is First Fit starting from the last allocation.
- **Best Fit** wastes the least space in each hole, but creates tiny leftover holes and is slow.
- **Worst Fit** keeps the largest holes available, but it is slow and can break up large free areas.

---

**Next up — Day 26:** **Paging and Non-Contiguous Memory Allocation**: how a process can be split across memory without needing one contiguous block.
