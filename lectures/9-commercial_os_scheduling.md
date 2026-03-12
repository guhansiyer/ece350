# Commercial OS Scheduling Algorithms

## Traditional Unix

Traditional Unix scheduling is a multilevel feedback queue with round robin scheduling in each priority queue. Time slicing is implemented with a 1 second slice as default.

Processor utilization for a process $j$ is calculated for an interval $i$ by keeping track of the amount of CPU time used:

$CPU_{j}(i) = \frac{CPU_j(i-1)}{2}$

The priority for a process $j$ at interval $i$ is given by:

$P_j(i) = B_j + \frac{CPU_j}{2} + N_j$

Where $B_j$ is the base priority of the process and $N_j$ is the **"nice"** value.

The "nice" value is a user-space mechanism to allow the user to voluntarily reduce a processes priority and be "nice" to other users.

The $CPU$ and $N$ components of the equation are restricted to prevent a process from migrating out of its assigned category. Processes are assigned a category which has an associated priority. In order from highest to lowest priority, the categories are:

1. Swapper (move process to and from disk).
2. Block I/O device control (eg: disk).
3. File manipulation.
4. Character I/O device control (eg: keyboard).
5. User processes

Unix puts its needs first when it comes to processes.

### SVR4

In Unix SystemV Release 4.0, the scheduling system was overhauled to give "real-time" processes the highest priority, then kernel processes, then user-mode preferences. The big differences are (1) 160 priority levels broken into three types and (2) preemption points.

The original design of the Unix kernel was not well suited to preemption because it was not expected that the kernel's own execution could be preempted at any time. So, in the new release, preemption points were added where it would be fine for the kernel to stop execution and do another operation.

### BSD

BSD is the Berkeley Software Distribution; a historic Unix-like operating system. FreeBSD is the most popular modern descendant of BSD.

FreeBSD's scheduling is similar to SVR4, but with more priority levels (256 instead of 160) and categories (5 instead of 3). This better suits multiprocessor systems.

What is also new is an interactivity scoring mechanism. Interactivity score identifies which threads are user-interactive and which are CPU-intensive. Threads that are user-interactive should get a higher priority to run (lower numbers are higher priorities). Interactivity is judged based on how often a thread gets blocked; threads waiting for the user or network for example would be considered interactive.

We define the maximum interactivity score as $m$, the runtime of the thread as $r$, and the sleep time of the thread as $s$. For threads where $s > r$, the interactivity score is calculated as $\frac{m}{2} \cdot \frac{r}{s}$, in the opposite case: $\frac{m}{2} \cdot \left(1+ \frac{r}{s}\right)$

This ensures that threads that sleep more than they run are always in the lower half, and threads that run more than they sleep the upper half of their priority bands.

Interactivity is a bit more complex than whether or not threads get blocked, but the mechanism is intentionally crude to provide less reward for gaming the system.

FreeBSD uses push and pull mechanisms for load balancing.

 The pull mechanism is relatively simple. When a CPU core has no work to do, it sets a bit in a bit mask to indicate that it is idle. If a new thread is added to a queue, the assignment mechanism checks for idle processors and sends it to them.

 The push migration is accompanied by the dispatcher twice per second, checking the highest and lowest load processors, and equalizing their run queues.

## Windows

Windows uses a priority-based, preemptive scheduling algorithm. The name of the selection routine is the *dispatcher*.

A thread will run until it is preempted, blocks, or terminates until the times expires. If a higher priority thread is unblocked, it will preempt a lower priority thread.

Windows has 32 different priority levels, regular (1-15) and real-time (16-31). Memory management tasks run at priority 0.

The dispatcher maintains a queue for each priority and goes through them from highest to lowest until it finds something to do.

There are six priority classes a process can be set to in Task Manager:

1. Realtime
2. High
3. Above Normal
4. Normal (default)
5. Below Normal
6. Low

Each of these classes has relative priority levels (in real-time, Windows ):

1. Time Critical
2. Highest
3. Above Normal
4. Normal (default)
5. Below Normal
6. Lowest
7. Idle

If a process reaches the end of a time slice, the thread is interrupted. If it is not real-time, the priority is lowered, to a minimum of the base priority of each class.

When a previously blocked process is unblocked, its priority is temporarily boosted (unless it is real-time). The boost amount depends on what the event was; a process waiting for a keyboard input gets a larger boost than one waiting for a disk operation.

Windows also gives low priority processes a temporary boost to a priority of 15 to prevent starvation and mitigate the impact of a priority inversion scenario.

Whatever process is running in the selected foreground window is given a priority boost and longer time slices. This is a key difference between the heritage of Unix and Windows; Unix was a time-sharing system with multiple users and lots of processes, Windows originally was a single-user desktop OS doing one or a few things at a time.

## Linux

Linux has two scheduling modes: Real-Time and Non-Real-Time. If the real-time scheduler is used, the system can still have non-real-time threads that are scheduled according to the normal scheduler routine.

### Linux Real-Time Scheduler

Linux's scheduler has scheduling classes for priorities to be assigned to:

* `SCHED_FIFO`: First-In, First-Out Real-Time threads
* `SCHED_RR`: Round-Robin Real-Time threads
* `SCHED_OTHER`: Non-Real-Time threads

In each class, threads can have different, relative priorities, where lower numbers are higher priorities. Real-Time priorities are in the range 0-99, other are 100-139.

`SCHED_FIFO` has the following rules:

1. The system will only interrupt a FIFO thread if one of these conditions are met:
    * Another, higher priority, FIFO thread is ready.
    * The current FIFO thread is blocked.
    * The current FIFO thread yields with `sched_yield`
2. If a FIFO thread is interrupted, it is placed in the queue associated with its priority.
3. If a FIFO thread becomes ready and that thread has higher priority than the currently-executing thread, the currently-executing thread is preempted for the highest priority FIFO thread that is ready. If there is a tie, the one waiting the longest is chosen.

These rules also apply to Round-Robin, but with time slicing implementing. 

Threads in the `SCHED_OTHER` category will only execute if no threads in the other queues are ready.

### Linux Non-Real-Time Schedulers

In earlier versions of Linux (2.4 >), the kernel used the traditional algorithm. A newer algorithm was then made called the $O(1)$ scheduler. In contrast the old scheduler ran in $O(n)$. Since version 2.6.23 of the kernel, a new algorithm has been used called the *Completely Fair Scheduler* (CFS).

The traditional Unix scheduler was not good at handling large numbers of process with its $O(n)$ performance, and its design caused significant difficulty with multiprocessor systems as it had a single run queue and lock, and no preemption. 

The single run queue means a task can and will be scheduled on any processor, but there is no processor affinity.

The single run queue lock means all processors have to wait for the lock if one processor wants to modify the run queue; they may be waiting for something to do.

Pre-emption being impossible means the scheduler would only re-evaluate what to run when something is blocked, time slices expire, or on an interrupt.

This is where the $O(1)$ scheduler came in. The kernel maintained two data structures for the processor:

```c
struct prio_array {
    int nr_active; // number of tasks in this array
    unsigned long bitmap[BITMAP_SIZE]; // priority bitmap
    struct list_head queue[MAX_PRIO]; // priority queue
}
```

There is a queue per priority level, so `MAX_PRIO` is the highest priority and number of queues. The bitmap array provides one bit per priority level, so with 140 levels and 32 bit words, `BITMAP_SIZE = 5`. There is an active and expired queue structure.

Initially all queues are empty and the bitmap is zeroed. If a process is created and enters the ready queue, it is put into the queue corresponding to its priority value. If that queue was previously empty, its bitmap value is set to 1.

All scheduling takes places from the active queue. The highest priority queue is chosen; if there are multiple tasks in that queue, they are scheduled with round robin. This continues until the active queue structure is empty, at which point the active and expired queues change places, and execution continues.

Part of the difficulty with the $O(1)$ scheduler is that is doesn't provide great performance for interactive processes. The CFS is not $O(1)$; it uses a red-black for modelling the ready queue, where processes are inserted based on a linear ordering of execution time. The leftmost node in the tree is the task that has spent the least amount of time executing, and therefore the task that will be scheduled next.

The CFS assigns a proportion of CPU time to each task based on the nice value, instead of a strict rule. The value may be in the range -20 to 19. It also does not use a particular time slice length, but instead has a target latency; an interval of time in which all ready tasks should get to run at least once. The time is given out based on this latency.

The linear ordering, `vruntime` or virtual run time, keeps track of how much time a task has been executing. There is a decay factor so that more recent history is more highly weighted in the calculation. Higher priority means faster decay. For tasks at a normal priority (nice value of zero), the virtual run time equals the physical run time.

Under this system, CPU-bound tasks will get a lower priority than tasks that are IO-bound. So a user interactive process will get to execute fairly quickly, making the system more responsive.

CFS also has group scheduling: we may designate a number of processes as belonging to a group. This is useful when a process spawns lots of threads or new processes. We can group them so that the multithreaded program is equal to other processes.

## A Decade of Wasted Cores

There are four distinct bugs in Linux multicore scheduling such that threads were waiting to run even when cores were idle, with 13-24% performance degradation on average. They all cause the same behaviour: cores are left idle for a long time when runnable threads are waiting to execute.

Load balancing is expensive and will run periodically but not often. A completely idle core will result in emergency load balancing. Load balancing is more complex than just moving things from the most busy core to the least busy, since we need to consider cache locality or non-uniform memory access. Above the level of cores, we have scheduling domains, configured by what hardware they have in common (e.g.: caches).

1. Group Imbalance Bug
    * Cores attempt to steal work from other cores if the average load of the victim scheduling group is higher than the average load of the one stealing. The problem is that averages can be misleading. The fix is to use the minimum load of the group; the load of the least loaded core of the group. Cores will steal more often but this is better than leaving them idle.
2. Scheduling Group Construction
    * Applications in Linux can be pinned to specific cores (`taskset`). If the groups are two hops apart, the load balancing thread might not steal them. All groups are constructed from the perspective of core 0. Then, if load balancing is running on core 31 for example, it might not steal from a neighboring core because it thinks its too far away.
3. Overload on Wakeup
    * If a thread on group 1 goes to sleep, and it gets unblocked later by some other thread, the scheduler will try to put it on one of the cores in group 1, even if other groups are free.
4. Missing Scheduling Domains
    * This was a refactoring error. When a core was removed and re-added, a step was skipped after refactoring changes which could cause all threads of an application to run on a single core instead of all of them.