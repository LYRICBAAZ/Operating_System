# Day 22/30 — Deadlock (Part 2): Avoidance, Detection & Recovery

Day 21 showed what deadlock is and how to **prevent** it by breaking one of the four conditions. Prevention is strict, and it wastes resources. Today covers the other approaches: **avoidance** (decide each request carefully, using the Banker's algorithm), **detection** (let deadlock happen, then find it), and **recovery** (break the cycle).

> **Note on numbering:** this post follows Lecture 22 in your notes ("Deadlock Part-2"), and it's Day 22 in this series. The next lecture is contiguous memory allocation, which is Day 23.

---

## 1. Deadlock Avoidance

**Idea:** the kernel looks at each resource request before granting it. It decides whether to grant the request or make the process wait, based on:
- the resources **currently available**,
- the resources **currently allocated** to each process,
- the **future requests and releases** of each process.

The goal is to schedule process and resource allocation so that deadlock never occurs.

### Safe State and Unsafe State

- **Safe state:** a state is safe if the system can allocate resources to each process (up to its maximum) in **some order**, and still avoid deadlock. That order is called a **safe sequence**.
- **Unsafe state:** the system can't guarantee that every process can finish. An unsafe state **may** lead to deadlock. It isn't always a deadlock yet.
- A system is in a safe state **only if a safe sequence exists**.
- If the system can't fulfill the request of all processes, the state is called **unsafe**.

> **Important point from the notes:** the OS can't prevent processes from requesting resources in a way that drives the system into an unsafe state. So the main key is this: **whenever a request is made, approve it only if the resulting state is safe.**

---

## 2. Banker's Algorithm

The Banker's algorithm is the scheduling algorithm that uses this idea to avoid deadlock.

1. A process requests a set of resources.
2. The system pretends to allocate them and checks whether the resulting state is **safe**.
3. If yes, the allocation is made.
4. If no, the process **must wait** until other processes release enough resources.

The name comes from a banker who never lends money in a way that leaves them unable to cover all their customers' needs.

### Extra Example (not in the notes)

Suppose there are **12 instances** of one resource type. The table shows each process's maximum need, current allocation, and what's still needed.

| Process | Max | Allocated | Need (Max − Allocated) |
|---|---|---|---|
| P0 | 10 | 5 | 5 |
| P1 | 4 | 2 | 2 |
| P2 | 9 | 2 | 7 |

Available = 12 − (5 + 2 + 2) = **3**.

**Is this state safe?** Try the order P1, P0, P2:
- P1 needs 2, and 3 are available. P1 gets its 2, finishes, and releases all 4 it holds. Available = 3 − 2 + 4 = **5**.
- P0 needs 5, and 5 are available. P0 gets its 5, finishes, and releases all 10. Available = 5 − 5 + 10 = **10**.
- P2 needs 7, and 10 are available. P2 gets its 7 and finishes. Everything is released.

Safe sequence: **⟨P1, P0, P2⟩**, so the state is safe.

**Now the risky request:** P2 asks for one more instance. Grant it, and the new state has Available = 2 and P2 needs 6.
- P1 needs 2, so it can finish, and Available becomes 4.
- P0 needs 5 and P2 needs 6. Neither fits in 4.

No safe sequence exists, so the state is **unsafe**, and the Banker's algorithm makes P2 wait.

---

## 3. Deadlock Detection

If the system has **no prevention or avoidance** technique, it may use **detection and then recovery** instead. The system lets deadlock happen, finds it, and fixes it.

### (a) Single Instance of Each Resource Type: Wait-for Graph

- Each resource type has only one instance.
- Build a **wait-for graph**, which is the resource allocation graph with the resource nodes removed. Only process nodes remain. An edge **Pi → Pj** means Pi is waiting for a resource held by Pj.
- **Rule:** a deadlock exists **if and only if there is a cycle** in the wait-for graph.
- To detect deadlock, the system maintains the wait-for graph and **periodically** runs an algorithm that searches for a cycle.

### (b) Multiple Instances of Each Resource Type

- Use the **Banker's algorithm** style check (Section 2). It's a more complex version of the same idea, because one cycle doesn't prove deadlock when a resource type has several instances.

---

## 4. Recovery from Deadlock

Once a deadlock is detected, there are two main ways to break it.

### (a) Process Termination

- **Abort all deadlocked processes.** This breaks the deadlock quickly, but all the work done by those processes is lost.
- **Abort one process at a time** until the deadlock cycle is eliminated. This wastes less work, but the system must check after each abort whether the cycle is gone.

### (b) Resource Preemption

- **Preempt** some resources from processes and give them to other processes.
- Keep doing this, one resource at a time, until the deadlock cycle is broken.
- This needs care: the preempted process must be rolled back to a safe earlier point, and the system must avoid **starvation**, where the same process keeps losing its resources.

---

## 5. Comparing the Approaches

| Approach | Idea | Cost |
|---|---|---|
| **Prevention** (Day 21) | Break one of the four conditions | Low utilization, strict rules |
| **Avoidance** (Banker's) | Approve only requests that keep the state safe | Needs advance knowledge of maximum needs |
| **Detection and recovery** | Let deadlock happen, find it, then abort or preempt | Detection runs cost time, and recovery loses work |
| **Ignore** (Ostrich algorithm) | Pretend deadlock never happens | Fine if rare, dangerous if not |

---

## Key Takeaways

- **Avoidance** approves a request only if the resulting state is **safe**, meaning a safe sequence exists.
- An **unsafe** state may lead to deadlock, so the Banker's algorithm makes a process wait instead.
- With a **single instance** per resource type, **deadlock exists if and only if there is a cycle** in the wait-for graph.
- **Recovery** either **terminates** processes (all at once, or one at a time) or **preempts** resources until the cycle breaks.
- Every approach has a trade-off: prevention and avoidance restrict resources, detection and recovery lose work.

---

**Next up — Day 23:** **Contiguous Memory Allocation**: how the OS assigns continuous blocks of memory to processes, and the problems that come with it.
