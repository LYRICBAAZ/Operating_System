
# Day 21/30 — Deadlock (Part 1)

Day 20 ended with the dining philosophers, where five forks and five hungry philosophers created a circle of waiting. That circle is a **deadlock**. Today's lecture generalizes it: what deadlock is, the four conditions that must all hold for it to happen, and the ways an OS can deal with it. Part 2 will go further into detection and recovery.

> **Note on numbering:** this post follows Lecture 21 in your notes ("Deadlock Part-1"), and it's Day 21 in this series. Deadlock Part 2 is Day 22.

---

## 1. Why Processes Compete

- In a **multiprogramming** environment, several processes compete for a **finite number of resources**.
- A process **requests** a resource. If the resource is free, it gets it. If another process holds it, the requesting process **waits**.
- Usually the wait is temporary. Sometimes the waiting process **never changes state**, because the resource it needs is held by a process that is also waiting. That situation is called a **deadlock (DL)**.

> A **deadlock** is when two or more processes wait for a resource that will never become available, because it is held by another process that is also waiting.

In a deadlock, processes never finish executing, and the system resources they hold stay tied up, which blocks other jobs from starting. Deadlock is a bug in the **process/thread synchronization** method.

---

## 2. What Counts as a Resource

- **Examples:** memory space, CPU cycles, files, locks, sockets, I/O devices.
- A resource can have **multiple instances**. For example, a system with 2 CPUs has two instances of the CPU resource.

### How a Process Uses a Resource

1. **Request:** ask for the resource. If it's free, lock it. Otherwise, wait until it's available.
2. **Use:** work with the resource.
3. **Release:** give the resource back, so other processes can use it.

---

## 3. Resource Allocation Graph

The notes draw deadlock as a graph with two kinds of nodes (threads T and resources R) and two kinds of edges.

- **Allocated** edge: resource → thread (the resource is held by that thread).
- **Request** edge: thread → resource (the thread is waiting for that resource).

```
   T1 ←—— allocated ——— R1
   │                     ▲
   │ request             │ request
   ▼                     │
   R2 ——— allocated ———→ T2
```

Reading the graph:
- T1 holds R1 and requests R2.
- T2 holds R2 and requests R1.
- The edges form a **cycle**: T1 → R2 → T2 → R1 → T1. Neither thread can move forward, so this is a deadlock.

**Rule of thumb:** a cycle in the graph is a sign of deadlock. (The notes show this graph; Part 2 will cover exactly when a cycle means deadlock and when it doesn't.)

---

## 4. The Four Necessary Conditions

Deadlock can happen only if **all four conditions hold at the same time**. If even one is false, deadlock is impossible.

| Condition | Meaning | Example |
|---|---|---|
| **Mutual exclusion** | Only one process at a time can use the resource. If another process requests it, the requester must wait until it's released. | A printer |
| **Hold and wait** | A process holds at least one resource and waits to acquire more that other processes hold. | P1 holds R1 and waits for R2 |
| **No preemption** | A resource can be released only voluntarily by the process holding it, after it finishes. The OS can't take it away. | A lock held until the thread unlocks it |
| **Circular wait** | A set {P0, P1, …, Pn} of waiting processes exists, where P0 waits for a resource held by P1, P1 waits for one held by P2, and so on, until Pn waits for one held by P0. | The cycle in Section 3 |

**Key point:** deadlock needs all four. The methods in Section 5 work by breaking one of them.

---

## 5. Methods for Handling Deadlocks

The notes list three broad approaches:

- **(a) Prevent or avoid:** use a protocol that ensures the system never enters a deadlocked state.
- **(b) Detect and recover:** let the system enter a deadlocked state, detect it, and recover.
- **(c) Ignore the problem:** pretend deadlocks never occur. This is the **Ostrich algorithm**, also called **deadlock ignorance**.

To ensure deadlocks never occur, the system uses either a **deadlock prevention** scheme or a **deadlock avoidance** scheme. Prevention is covered next, and avoidance and detection are in Part 2.

---

## 6. Deadlock Prevention: Breaking Each Condition

Prevention means making sure at least one of the four conditions can never hold.

### (a) Breaking Mutual Exclusion

- Use locks only for **non-sharable** resources.
- **Sharable** resources, such as read-only files, can be accessed by many processes or threads at once, so they don't need locks.
- **Limit:** mutual exclusion can't always be removed, because some resources are **intrinsically non-sharable**. A printer can't print two jobs at once. So this condition can't be denied in general.

### (b) Breaking Hold and Wait

To guarantee that a process never holds a resource while requesting another, use one of two protocols:

- **Protocol A:** each process must request and be allocated **all** its resources **before** it starts executing.
- **Protocol B:** a process can request resources **only when it holds none**. To request more, it must first **release all** the resources it currently holds.

```
Protocol A:  request(R1, R2, R3) all at once → run → release all
Protocol B:  request(R1) → run → release R1 → request(R2) → run → release R2
```

### (c) Breaking No Preemption

Two approaches from the notes:

- **Approach 1:** if a process holds some resources and requests one that can't be allocated immediately, then **all** the resources it currently holds are **preempted** (taken away). The process restarts only when it can regain its old resources and the new one.
  - **Caution:** the notes warn that **livelock** may occur. Processes keep preempting each other and make no progress, even though they're not blocked.
- **Approach 2:** when a process requests a resource:
  1. If it's free, allocate it.
  2. If it's held by another process that is **itself waiting** for more resources, **preempt** it from that waiting process and give it to the requester.
  3. Otherwise, the requester waits.

### (d) Breaking Circular Wait

- Impose a **proper ordering** on resource allocation. Every process must request resources in the same global order.
- Example from the notes: P1 and P2 both need R1 and R2. If both **lock R1 first, then R2**, then whichever process locks R1 first also gets R2. The other process waits at R1, so no cycle can form.

This is the same fix as the **odd-even rule** from Day 20, applied as a general rule.

---

## 7. Summary of Prevention

| Condition broken | Method | Catch |
|---|---|---|
| Mutual exclusion | Lock only non-sharable resources | Some resources are intrinsically non-sharable |
| Hold and wait | Request all upfront (A), or request only when holding none (B) | Low resource utilization; possible starvation |
| No preemption | Take resources away from waiting processes | Livelock is possible |
| Circular wait | Fixed global ordering of resources | Processes must follow the same order |

The first two protocols under hold and wait are usually criticized for wasting resources, because a process can hold a resource it doesn't need yet. That's why prevention is a trade-off between safety and efficiency.

---

## Key Takeaways

- A **deadlock** is a set of processes waiting forever for resources held by each other.
- Resources can be **sharable** or **non-sharable**, and can have **multiple instances**.
- **Four conditions must all hold:** mutual exclusion, hold and wait, no preemption, and circular wait.
- The **resource allocation graph** shows deadlock as a cycle of request and allocated edges.
- **Prevention** works by breaking one of the four conditions. Each fix has a cost.
- Other approaches: **avoidance** (Part 2), **detection and recovery** (Part 2), and **ignoring** it (the Ostrich algorithm).

---

**Next up — Day 22:** **Deadlock Part 2**: deadlock avoidance, detection, and recovery.
