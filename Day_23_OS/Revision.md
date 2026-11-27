# Day 23/30 — Revision: Days 20 to 22

This is a revision day. Days 20 to 22 covered deadlock in depth, from the dining philosophers scenario to the full set of handling methods. Use this as a one-page recap, and try each self-check before reading the answer in brackets.

---

## Day 20: Dining Philosophers Problem

- Five philosophers alternate between **thinking** and **eating**, and each needs **two forks** to eat.
- Each fork is a **binary semaphore**, `fork[5] = {1, 1, 1, 1, 1}`. This stops two neighbors eating together.
- **Deadlock:** if all five pick up their left fork at once, every fork is 0, and each waits forever for the right fork.
- **Three fixes:**
  - **(a)** Allow at most **4** philosophers to sit at once.
  - **(b)** Pick up **both** forks **atomically**, inside a critical section.
  - **(c)** **Odd-even rule:** odd philosophers pick up left first, even philosophers pick up right first.
- **Key point:** semaphores alone aren't enough. The solution needs an extra rule about the order of requests.

**Self-check:** Why does the odd-even rule break the deadlock? [The circular wait can't form, because at least one pair of neighbors requests forks in opposite orders.]

---

## Day 21: Deadlock Part 1

- **Deadlock:** two or more processes wait for resources that are held by each other, so none can ever finish.
- **Resources** can be sharable or non-sharable, and can have multiple instances.
- **Resource allocation graph:** a **cycle** of request and allocated edges indicates deadlock.
- **Four necessary conditions**, all of which must hold at once:
  - Mutual exclusion
  - Hold and wait
  - No preemption
  - Circular wait
- **Prevention** breaks one of the four:
  - Mutual exclusion: lock only non-sharable resources.
  - Hold and wait: request all resources upfront (A), or request only when holding none (B).
  - No preemption: take resources away from waiting processes.
  - Circular wait: impose a fixed global ordering on resources.

**Self-check:** Which condition does "lock R1 then R2 in the same order" break? [Circular wait.]

---

## Day 22: Deadlock Part 2

- **Avoidance (Banker's algorithm):** approve a request only if the resulting state is **safe**, meaning a safe sequence exists.
- **Unsafe state:** may lead to deadlock. The request makes the process wait.
- **Detection:** with one instance per resource type, **deadlock exists if and only if there is a cycle** in the **wait-for graph**. The system checks periodically.
- **Recovery:**
  - Terminate all deadlocked processes, or terminate one at a time until the cycle breaks.
  - Preempt resources from processes until the cycle breaks.

**Self-check:** A wait-for graph has a cycle P1 → P2 → P1. Is there a deadlock? [Yes, with one instance per resource type.]

---

## Comparing the Approaches

| Approach | Idea | Cost |
|---|---|---|
| Prevention | Break one of the four conditions | Low utilization |
| Avoidance (Banker's) | Approve only safe requests | Needs maximum needs in advance |
| Detection and recovery | Find the cycle, then abort or preempt | Lost work |
| Ignore (Ostrich) | Pretend it never happens | Dangerous if deadlock occurs |

---

## Quick Recall Table (Days 20–22)

| Topic | One-line memory hook |
|---|---|
| Dining philosophers | Semaphores aren't enough; order matters |
| Deadlock | Waiting forever for resources held by each other |
| Four conditions | ME, hold and wait, no preemption, circular wait |
| Resource allocation graph | Cycle means deadlock |
| Safe state | A safe sequence exists |
| Banker's algorithm | Grant only if still safe |
| Wait-for graph | Single instance: cycle means deadlock |
| Recovery | Terminate or preempt |

---

**Next up — Day 24:** Contiguous Memory Allocation, then Day 25: Free Space Management.
