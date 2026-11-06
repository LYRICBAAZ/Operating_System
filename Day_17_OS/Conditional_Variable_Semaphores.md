# Day 17/30 — Condition Variables & Semaphores

Day 16 showed the problem: threads that share data need protection, and a plain flag isn't enough. Locks fix the race condition, but a thread that has to wait for something specific often ends up **busy waiting**, repeatedly checking a condition and burning CPU time. Today we look at two tools that solve this. A **condition variable** lets a thread sleep until a condition becomes true. A **semaphore** is a counter that controls access to a limited number of resources, and it can also make threads sleep instead of spin.

---

## 1. Condition Variable

> A **condition variable** is a synchronization primitive that lets a thread **wait until a certain condition occurs**.

### How It Works
- A condition variable **works together with a lock**.
- A thread can enter the **wait state only after it has acquired the lock**.
- When the thread waits, it **releases the lock** and sleeps until another thread **notifies** it that the event has occurred.
- Once the waiting thread is woken up and returns to the **running state**, it **acquires the lock again immediately** and continues executing.

The sequence in short:

```
1. Thread acquires the lock
2. Checks the condition; it's not true yet
3. wait()    → releases the lock and sleeps
4. Other thread changes the state, then calls notify/signal
5. Waiting thread wakes up, re-acquires the lock, and continues
```

### Why Use It?
- **To avoid busy waiting.** A thread that waits on a condition sleeps instead of looping and checking over and over.
- The notes also point out that **contention is not the issue here**. Each thread's lock hold is short, because a waiting thread gives the lock up before it sleeps.

> **Note:** a condition variable does not hold data. It is a way to *sleep and wake up* based on a condition. The condition itself is a predicate about shared data, which is still protected by the lock.

---

## 2. Semaphores

> A **semaphore** is a synchronization method built on an **integer**, whose value is equal to the **number of resources** available.

### The Core Idea
- Multiple threads can enter the critical section **concurrently**, as long as the semaphore allows it.
- It allows **multiple program threads to access a finite number of instances of a resource**.
- A mutex, by contrast, allows multiple threads to access a **single shared resource one at a time**.

Think of a parking lot with 3 spaces. The semaphore starts at 3. Each car that enters decrements the count, and each car that leaves increments it. When the count reaches 0, the next car has to wait.

### Two Operations: wait() and signal()
- **wait()** (also called P, or down): if the value is positive, decrement it and continue. If it isn't positive, the thread must wait.
- **signal()** (also called V, or up): increment the value, and wake up a waiting thread if there is one.

### Binary and Counting Semaphores

| | Binary Semaphore | Counting Semaphore |
|---|---|---|
| **Value** | Only 0 or 1 | Can range over an unrestricted domain |
| **Use** | Mutual exclusion, like a mutex lock | Controlling access to a resource with a finite number of instances |
| **Also called** | Mutex lock (per the notes) | — |

> **A precision note:** the notes say a binary semaphore is "aka mutex locks." They behave similarly, but they're not identical. A mutex has an **owner**, meaning only the thread that locked it can unlock it. A binary semaphore has no owner, so any thread can signal it.

---

## 3. Avoiding Busy Waiting in Semaphores

Early semaphore implementations made a waiting process spin in a loop, which wastes CPU. The fix modifies `wait()` and `signal()`:

1. A process executes `wait()` and finds that the semaphore value is **not positive**, so it must wait.
2. Instead of spinning, the process **blocks itself**.
3. The block operation places the process in a **waiting queue associated with that semaphore**, and its state changes to **waiting**.
4. Control passes to the **CPU scheduler**, which selects another process to run.
5. When another process executes `signal()`, one blocked process is **woken up** by a **wakeup()** operation. Its state changes from waiting to **ready**, and it is placed in the **ready queue**.

```
wait(S):
    S.value = S.value - 1
    if S.value < 0:
        add this process to S.queue
        block()                 // sleep, don't spin

signal(S):
    S.value = S.value + 1
    if S.value <= 0:
        remove a process P from S.queue
        wakeup(P)               // P moves to the ready queue
```

This is the link back to the process states from Day 10: a blocked process moves to the **waiting** state, and a woken process moves to **ready**, not straight to running.

---

## 4. Worked Example: 2 Resources, 3 Threads

A semaphore `S` starts at **2**, and three threads T1, T2, and T3 want to use the resource.

| Step | Action | `S.value` | Result |
|---|---|---|---|
| 1 | T1: `wait(S)` | 2 → 1 | T1 gets the resource |
| 2 | T2: `wait(S)` | 1 → 0 | T2 gets the resource |
| 3 | T3: `wait(S)` | 0 → −1 | value is negative, so **T3 blocks** and joins the queue |
| 4 | T1: `signal(S)` | −1 → 0 | T3 is **woken up** and moves to the ready queue |
| 5 | T3 runs and gets the resource | 0 | both resources are in use again |

Notice that the value goes negative. A negative value means **the number of threads waiting** on the semaphore, which is why signal wakes someone up whenever the value is still at or below zero.

---

## 5. Condition Variable vs Semaphore

Both let a thread sleep and wake up, but they're designed for different situations:

| | Condition Variable | Semaphore |
|---|---|---|
| **Holds** | No data. It's a wait/notify mechanism. | An integer count of available resources. |
| **Works with** | A lock, which must be held when waiting | Itself; no separate lock is required |
| **Wakes up when** | Another thread notifies that a condition is true | A `signal()` increases the count |
| **Typical use** | "Wait until the queue is non-empty" | "Allow at most N threads into this section" |

---

## Key Takeaways

- A **condition variable** lets a thread wait for a condition. It works with a lock, releases the lock while waiting, and re-acquires it on wakeup. Its main purpose is to **avoid busy waiting**.
- A **semaphore** is an integer that counts available resources. It allows **multiple threads** in a finite-resource section, while a mutex allows one at a time.
- A **binary semaphore** holds 0 or 1 and is similar to a mutex, but it has no owner. A **counting semaphore** can range over a larger domain.
- **wait()** decrements the value and blocks the process if the value goes negative. **signal()** increments it and wakes a blocked process.
- Blocked processes move to the **waiting** state and are placed in a **waiting queue**. A woken process moves to **ready**, not running.

---

**Next up — Day 18:** **Revision Part 1 (Days 1 to 9)**, followed by Revision Part 2 (Days 10 to 17) on Day 19. Then Day 20 covers the Dining Philosophers Problem.
