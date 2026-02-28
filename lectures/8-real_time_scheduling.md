# Real Time Scheduling

The term "task" refers to something that needs doing. The scheduler operates on threads rather than tasks, but we will say that a task corresponds to one thread.

A task is *hard real-time* if it has a deadline that must be met to prevent an error, prevent some damage to the system, or for the answer to make sense. A *soft real-time* task has a deadline that is not mandatory; missing it degrades the response quality but the result will be relatively useful.

There is also *firm real-time*, which is when the response is useless if it arrives a little too late.

## Properties of Real-Time Systems

Real-time systems are considered to be unique in five key areas:

1. **Determinism**
    * Operations are predictable, either performed at fixed times or within time limits.
    * Perfect determinism will not happen with concurrency.
    * For most RTOS scenarios, it is sufficient to know things will happen in some fixed time period.
    * No matter how unlucky the sequence of events, we can still start the task on time to successfully complete it.

2. **Responsiveness**
    * Critical tasks and external events can be handled within strict time constraints, guaranteed with high predictability and low latency.
    * Determinism is how long it takes before the OS acknowledges the request/interrupt. Responsiveness is how long it takes after acknowledgement to handle it.

3. **User (administrator) control**
    * There are two ways admin control can go in an RTOS: no control or more control.
        * With no control, the systems runs as programmed and configured; no input from users or administrators.
        * With more control, administrators take on more responsibility in the OS; it does not know what tasks are real-time and which are not, nor does it know which are soft/firm/hard.
            * The administrator must specify what is what, and make other choices if needed, such as what scheduling algorithm is used.

4. **Reliability**
5. **Fail-soft operation**
    * If it's somehow not possible to succeed in completing all tasks, the system will try its best to complete as many tasks as it can, with priorty given to hard real-time tasks.

## Scheduling Is Central

The objective with scheduling in a real-time system is to ensure all hard real-time tasks are complete before their deadline, and as many as possible of the soft real-time tasks finish.

Any non-preemptive scheduling algorithms are not suitable in this case, because if a hard real-time task arrives while an unimportant task is in progress, it is not sensible for it to wait until the currently executing task finishes. Similarly, time-slicing based routines do not work.

## What kind of task is it?

1. **Fixed-Instance Tasks**: something that executes a fixed number of times and that is frequently used just once, for init or cleanup purposes.
2. **Periodic Tasks**: a task that recurs at regular intervals (ie: check a sensor, keep a wifi connection alive).
3. **Aperiodic Tasks**: tasks that respond to irregularly occurring events. These are very rarely hard real-time because it is difficult to make a guarantee we will finish one task before the next one occurs.
4. **Sporadic Tasks**: aperiodic tasks that require meeting deadlines. We need a guarantee that there is a minimum time between occurrences of the event in order to make such a promise.

## It takes how long?

We have talked about execution time like it is a known quantity, but we need to know where these come from. When the goal is to ensure that hard deadline are met, we need to know how long tasks take assuming the worst-case scenario. We consider two ways to estimate the worst-case execution time: source code analysis and empirical testing.

We want to overestimate the expected execution time in either strategy. We may calculate needing much more capacity than actually needed, but it is not necessarily a bad thing to have more capacity than needed.

### Code Analysis

This involves looking at the algorithm, possible inputs and execution paths, and try to figure out how long the longest path could be. It will be an overestimate since it doesn't take into account CPU pipelining or compiler optimizations.

### Empirical Observation

The other approach is to measure instead of estimate. If you have version $n$ of a system and want to build $n+1$, it may be easy to look at existing data and extrapolate, but if you want a new system then you need a simulation.

By preparing and running a simulation (with proper assumptions and construction), you'll (presumably) get a distribution of task runtimes that, with the confidence interval, can estimate the worst case runtime.

## Real-Time Scheduling Algorithms

For now we consider a uniprocessor view.

1. **Earliest Deadline First**
    * Choose the task with the soonest deadline.
    * Can be implemented with a priority queue where priority is determined by the deadline, in ascending order ot time of deadline.
        * Doesn't account for soft-real time tasks having sooner deadlines than firm or hard-real time time tasks.
    1. **Deadline Interchange**
        * The above algorithm is subject to a problem identical to priority inversion.
        * Suppose task A has locked a mutex, and  task B arrives that needs the mutex, and has a sooner deadline.
        * Task A would be preempted for B, at least until B gets blocked.
        * If there are multiple tasks that want the resource, then task A could be waiting a while to proceed, so long that task B could miss its deadline.
        * Task A needs to finish its critical section and release the mutex. To do so, task A is assigned a new deadline; the soonest deadline of all the tasks waiting for it.
2. **Least Slack First**
    * *Slack* is how long a task can wait before it must be scheduled to meet its deadline.
    * This algorithm is similar to EDF, but the priority is determined by slack.
3. **Rate Monotonic Scheduling**
    * Based around the idea of basing priority on the period; tasks that execute more often are given higher priority.
    * It's possible to fail to schedule things in such a way that all tasks meet their deadlines even if utilization < 1.
    * Suppose we want to schedule $n$ periodic tasks with the form $(C_k, \tau_k)$: (1,4), (3,7), (3, 10).
    * If we try to schedule these, the third task fails to meet its deadline.
    * Task 1 runs, then 2, then 1, then 3, then 2, which is interrupted by 1, so 1 runs to completion, then 2 resumes, then 3 misses its deadline.
    1. **Deadline-Monotonic Scheduling**: priority is assigned based on the shortest deadlines.
4. **Aperiodic Servers**
    * A task with a soft deadline is hard to schedule because the scheduler struggles to discern soft and hard real-time tasks.
    * A polling server is a small container where aperiodic tasks occur. The server itself is a hard-deadline periodic task with a fixed execution time and a deadline equal to its period.
    * During the execution time of the server task, the aperiodic tasks waiting will run sequentially (the server is a trojan horse).
    * If there are too many tasks or they do not finish, they carry over to the next time the server runs.
    * If there are not enough tasks and there is remaining time, the execution ends.
    * We can have multiple servers for different types of aperiodic tasks.
    * This does not affect the ability of the server to complete hard-deadline tasks.
    1. **Deadline-Deferrable**
        * Instead of giving up unused execution time, it gets saved in case something else arrives. The time is either used up or expires.
    2. **Deadline-Sporadic**
        * Instead of giving up unused execution time, it can be indefinitely saved, and execution is forced to be spread out more evenly.
        * The execution time for the server is broken down into chunks.
        * When the server runs an aperiodic task, the amount of time it runs for is deducted from the budget of chunks.
        * When a chunk is depleted, it's scheduled for replenishment at a fixed future time. If chunks are exhausted, the server is suspended until the next period.
    3. **Deadline-Exchange**
        * Builds on Deadline-Deferrable, but with simpler chunk tracking.
        * Any remaining execution time is discarded when the request queue is empty, and the wait for replenishment is based on the last chunk of execution time used.

## Multiprocessor Math

If there are multiple processors in the system, we need to decide whether tasks have dedicated CPUs or can move between them. There are three decisions that are relevant:

1. Whether preemption is permitted.
2. Whether job migration is permitted.
3. Whether job parallelism is forbidden.

If tasks have dedicated CPUs, the second decision is a definitive no.

No migration means that there is a possibility that tasks are failing to meet their deadlines on CPU $X$ for lack of CPU time even when CPU $Y$ and $Z$ have idle capacity.

We have a few options. The first is global scheduling, where all tasks go into one queue and the scheduler assigns them to available CPUs. There is potentially some overhead of managing the queue, and migration is allowed. The second is partitioned scheduling, when tasks are statically assigned to a CPU and the CPU managed its own queue.

None of these are actually solutions that ensure optimal scheduling for multicore systems.

### P-Fairness

The *P* in P-Fair scheduling stands for proportional. The goal is to allocate CPU time in a way that tasks make progress as steady rates. An application can request time $x_i$ time units every $y_i$ time quanta (slices). The system guarantees that over any $T$ slices, a continuously-running application receives between $x_i / y_i \times T$ time and quanta of service.

To restate, time is divided up into small pieces, and each process gets a proportional share of the CPU time, which is never more than one time slice away from the amount of time it "should" receive.

The term "lag" is used to represent the difference between the allocated time and the time a task should have. If lag is greater than zero, the task is behind on the CPU time it should have. If it is below zero, the task had more CPU time than it should have.

A task is considered *urgent* if its lag is greater than 0, and trivial if its lag is negative.

The algorithm then consists of three parts:

1. Schedule all urgent tasks.
2. Do not schedule trivial tasks.
3. Schedule other tasks in order of highest to lowest lag until capacity is filled.

The goal is to keep the lag between -1 and +1 for all tasks.

Each task is broken into small slices and each subtask can execute when it can, before its deadline. Wasted space is limited, and P-Fair is always periodic.

The algorithm does require a lot of computation per timeslice, but it is worthwhile for multicore systems. Whatever the overhead of P-Fair, it's better than losing 50% of CPU capacity.
