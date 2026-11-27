# Day 27/30 — Segmentation: Non-Contiguous Memory Allocation

Day 26 introduced paging, which splits memory into fixed-size pages and frames. Paging solves external fragmentation, but it creates a new problem: it cuts the program without caring what the program means. Today's lecture covers **segmentation**, a scheme that splits the process the way the **programmer** sees it, into logical units, called segments. It fits the user's view of memory, and it has its own trade-offs.

> **Note on numbering:** this post follows Lecture 27 in your notes ("Segmentation | Non-Contiguous Memory Allocation"), and it's Day 27 in this series. The next lecture is virtual memory, demand paging, and page faults, which is Day 28.

---

## 1. The Problem: User View vs Physical Memory

- An important aspect of memory management, which becomes unavoidable with paging, is the **separation of the user's view of memory from the actual physical memory**.
- Paging splits a process into equal pages. The OS decides where the cuts go, and the programmer has no say.

---

## 2. What Segmentation Is

> **Segmentation** is a memory-management technique that supports the **user's view of memory**.

- A **logical address space** is a **collection of segments**. The segments are based on the user's view of logical memory.
- A process is divided into **variable-size segments**, based on the logical units of the program. Segment sizes differ, unlike pages.
- Each segment is named by a **segment number** and an **offset**, written as:

```
<segment-number, offset>  =  {s, d}
```

- `s` selects the segment. `d` is the position inside that segment.

### Why Paging Doesn't Respect the Program's Structure

- Paging is closer to the **OS** than the **user**. It divides every process into equal pages.
- A process may have related parts of one function that must be loaded together, but paging can put them on **different pages**.
- The OS doesn't care about the user's view. It may **split the same function** across several pages, and those pages may or may not be loaded at the same time.
- The notes say this **decreases the efficiency** of the system.

### Why Segmentation Is Better for the Program

- Each segment holds **one type of function**. For example:
  - The **main** function goes in one segment.
  - **Library** functions go in another segment.
- The compiler can keep related code and data together, so loading one unit doesn't drag in unrelated pieces.

---

## 3. Segmentation Hardware

The notes show how a logical address is translated.

```
CPU → logical address {s, d}
        │
        ├── s → segment table[s] → (limit, base)
        │
        └── compare d with limit:
              d < limit (Yes) → physical address = base + d → physical memory
              d ≥ limit (No)  → trap: addressing error
```

- The **segment table** has one row per segment, with two fields:
  - **Base:** the starting physical address of the segment.
  - **Limit:** the length of the segment.
- The offset `d` must be **less than the limit**. This is the same protection check you saw with the limit register in Day 24.

### Worked Example (not in the notes)

Segment table for a process:

| Segment | Base | Limit |
|---|---|---|
| 0 | 1400 | 1000 |
| 1 | 6300 | 400 |

- Logical address **(1, 53)**: 53 < 400, so it's valid. Physical address = 6300 + 53 = **6353**.
- Logical address **(1, 500)**: 500 ≥ 400, so it's out of range. The hardware traps with an addressing error.

---

## 4. Advantages of Segmentation

1. **No internal fragmentation.** Each segment is sized to its content, so no space is wasted inside a fixed-size unit.
2. **Contiguous allocation within a segment.** One segment is stored in one contiguous block, which keeps work inside a segment efficient.
3. **Smaller segment table.** The segment table is generally smaller than a page table, because a program has far fewer segments than pages.
4. **More efficient system.** The compiler keeps the same type of functions in one segment, so the working set stays small and cohesive.

---

## 5. Disadvantages of Segmentation

1. **External fragmentation.** Segments have different sizes, so free holes between them can be too small to use. This is the same problem as dynamic partitioning in Day 24–25.
2. **Swapping is slower.** Because segments differ in size, moving them to and from disk takes varying amounts of time. The notes say this is "not good".

---

## 6. Paging vs Segmentation

| | Paging (Day 26) | Segmentation (Day 27) |
|---|---|---|
| Unit size | Fixed (page = frame) | Variable |
| Who decides the split | OS | User or compiler (logical units) |
| Address form | Page number + offset | Segment number + offset |
| Table | Page table | Segment table (base and limit) |
| Internal fragmentation | Yes, in last page | No |
| External fragmentation | No | Yes |
| Matches program structure | No | Yes |

---

## 7. Modern Systems

- Modern system architecture usually provides **both** segmentation and paging, in some **hybrid** approach.
- The program's logical structure is handled by segments, and each segment is then split into pages for physical memory.
- This combines the user view of segmentation with the fixed-size frames of paging.

---

## Key Takeaways

- Paging splits memory by size, not by meaning. That separation of the **user's view** from **physical memory** is unavoidable with paging.
- **Segmentation** divides a process into **variable-size segments** based on the logical structure of the program, and it addresses them as **{segment number, offset}**.
- The **segment table** holds a **base** and a **limit** for each segment. An offset at or above the limit traps.
- Segmentation avoids internal fragmentation and keeps related code together, but it suffers from **external fragmentation** and slower swapping.
- Modern systems use **hybrid** designs that combine segmentation and paging.

---

**Next up — Day 28:** **Virtual Memory, Demand Paging & Page Faults**: how a process can run even when only part of it is in memory, and what happens when the CPU needs a page that isn't there yet.
