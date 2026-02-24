# Scheduling Algorithms

A simple scheduling algorithm would be to always pick the highest priority, non-blocked task, and execute it. While simple, this approach has drawbacks. We examine the following options for scheduling algorithms:

1. **Highest Priority, Period**
    * Not difficult to implement; keep priority queues for each priority level and put non-blocked tasks in appropriate queue, if priority changes then move the task appropriately.
    * Drawback has been widely identified: starvation. If we have a bunch of low priority tasks, they may never get the chance to run.

2. **First Come, First Served**
    * Whichever process requests the CPU first, get the CPU (all processes are equal). If the current process finishes or gets blocked, select the next ready process.
    * The drawback results from different processes having vastly different completion times. If a process with a heavy completion time is at the front of the queue, it will raise the average completion time no matter how small the following processes completion time is.
    * Favours CPU-Bound processes over I/O-Bound. Because the disk is slow, we'd like to keep it busy. If a disk write is completed and there is a process that wants to read from disk, the ideal is to select that process.

3. **Round Robin**
    * Round Robin is time slicing combined with first come, first served. Every $t$ units of time, an interrupt is generated to run the dispatcher.
    * The question is how big we should make $t$?
        * If it is too long, one process may seem unresponsive while another process has the CPU.
        * If it is too short, the system spends too much time handling the interrupt and running the dispatcher/algorithm, and not enough time running a process.
    * If we know that the average process runs for $r$ units of time before getting blocked, then we should choose $t$ so that it is slightly larger than $r$. If $t$ is smaller than $r$, processes will be frequently interrupted by the time slice.
    * Just like first come first served, Round Robin tends to favour CPU-bound processes. I/O-bound processes run for a short time, block on I/O, and when I/O finished, the process gets back in the ready queue.
    * Round Robin can be improved to Virtual Round Robin.
        * A process that gets unblocked after I/O gets higher priority.
        * Instead of rejoining the general queue, there is an auxilary queue for processes that were previously blocked on I/O.
        * When the scheduler chooses a process to run, it takes from the auxilary queue if possible.
        * If a process simply ran out of time, it goes into the normal ready queue.

4. **Shortest Process Next**
    * If we know anything about the total length of execution, we might want to give short processes priority.
    * We let them go first to free up the queue for bigger processes.
    * This is essentially addressing the main drawback of first come, first served.
    * Of course, the drawback is that the OS does not know much about the total length of execution.

5. **Shortest Job First**
    * A better name is "shortest next CPU burst".
    * The objective, then, is to pick the process likely to have the smallest CPU burst.
    * The problem is predicting the times. The best thing to do is gather information about the past to guess about the future, which gives us this formula, where $T_i$ is the burst time for the i-th instance of the process, $S_i$ is the predicted value for the instance, and $S_1$ is a guess at the first value:

    $$S_{n+1} = \frac{1}{n} \sum_{i=1}^{n}T_i$$

    * Instead of picturing it as a sum, we can just update the value:

    $$S_{n+1} = \frac{1}{n}T_n + \frac{n-1}{n}S_n$$

    * We want to give more weight to recent values, so instead of using the equal weight (1/n), we use exponential averaging. Define $\alpha$, a weighting factor between 0 and 1.

    $$S_{n+1} = \alpha T_n + (1 - \alpha) S_n$$

    * The older an observation, the less it is counted, and the largeer $\alpha$ we have, the more recent observations matter.
    * There is a chance that longer processes will starve if there is a steady stream of shorter processes.

6. **Shortest Remaining Time**
    * This is a modification of the previous strategy.
    * When a new process is scheduled or an old one is unblocked, the scheduler evalutes if there is a shorter predicted running time than the currently running process.
        * If there is one, the available process will be ran, displacing the currently running process.
    * There is a chance, just like the last algorithm. If we choose $S_1$ to be 0 for a new process, they will always displace the current process.
    * One advantage is we no longer need time slicing. Instead of interrupting the process every $t$ units of time, the other interrupts (user programs launching, hardware operations, etc.) will be what prompts the scheduler, which is a net performance increase.

7. **Highest Response Ratio Next**
    * Introduce normalized turnaround time: the ratio fo the turnaround time to the service time.
    * The goal of the HRRN stratey is to minimize the normalized turnaround time average across all processes.
    * We calculate the response ration $R = \frac{w+s}{s}$, where $w$ is the waiting time and $s$ is the service time (a guess).
    * When we need to select a new process to run, we pick the process with the highest $R$-value.
    * Jobs with a small $s$ are likely to get scheduled quickly.
    * HRRN introduces something we have not had yet: factoring in a process's age. $w$ indicates how long a process has spent waiting.
    * A process that has spent a long time waiting will rise in priority over time until it eventually runs.
    * HRRN thus has no starvation, because all processes will have a high enough $R$ over time.
    * The major problem here is that we need a way to estimate $s$.

8. **Multilevel Queue (Feedback)**
    * We have treated processes more or less equally (excluding priority) thus far.
    * While fair, it is not always ideal, especially for situations where processes behave differently (user/kernel programs).
    * We could apply different algorithms to different process types.
    * The multilevel queue takes the ready queue and splits it into several queues. A process can only be in one of the queues base on some attribute of the process. One queue could be scheduled by round robin, and another by first come, first served, for example.
    * We also need a way of choosing which queue to take from. This depends on our goals. We might dictate that some queues have absolute priority over others, or we might have time slicing between the queues.

9. **Guaranteed Scheduling**
    * The idea is to promise the users something and fulfill that promise.
    * For example, if we have $n$ processes, we could promise that each gets an equal share ($1/n$) of the CPU time.
    * The system, in this case, must track how much CPU time each process has received, and consider how this compares to the ideal (time since creation over $n$).
    * If a process has a value of 0.5, it has had only half the CPU it "should" have received.
    * The goal is to run the process with the lowest score, to try to keep all values as close to 1.0 as possible.

10. **Lottery**
    * Every process gets some number of "lottery tickets" for each resource. When a decision has to be made, a ticket is randomly selected, and the process holding that ticket gets the resource.
    * This system provides some clarity over others. If a process has a fraction $f$ of the total tickets, we can expect it to get $f\%$ of the resource(s).

## The Idle Task

Sometimes, scheduling algorithms cannot produce a new process to run (because there is nothing to do). In this case, we rely on the idle task. The implementation of the task may vary; it could be repeated invocations of the scheduler, meaningless addition, or a lot of `NOP` instructions.

The idle task is useful to prevent special cases in the scheduler. It also provides some information about how much time the CPU spends doing "nothing". There are usually some housekeeping tasks that the CPU can be doing when it has nothing else to do, such as collecting statistical data or defragmenting the hard drive.

## Bumping the Priority

Sometimes we get into a situation called a priority inversion; a high priority process ($P_1$) waiting for a low priority process ($P_2$). Suppose $P_1$ is blocked on a semaphore, while $P_2$ is in its critical section. $P_1$ cannot run, because it is blocked, and may remain blocked for a while. Meanwhile, other lower priority processes can run (higher priority than $P_2$). This is undesirable.

> Solution: Priority Inheritance

We temporarily bump the priority of $P_2$ to be equal to that of $P_1$, so that $P_1$ can be unblocked as quickly as possible.

To generalize, a low priority process should inherit the higher priority if a higher priority process is waiting for a resource the lower priority process holds, so that it will get selected and exit the critical section. Then, its priority falls down to normal, so the high priority process will be selected and continue.

## Multiprocessor Scheduling

We can classify multiprocessor systems into three buckets:

1. **Distributed**: A collection of relatively autonomous systems who interact.
2. **Functionally Specialized**: Specialized chips working on specific areas.
3. **Tightly Coupled**: A set of processors sharing a common main memory, under the control of the operating system.

We then have to consider the interactions of various processes:

| Grain Size  | Description               | Interval (Instructions) |
|-------------|---------------------------|-------------------------|
| Fine        | Single instruction stream | < 20                    |
| Medium      | Single application        | 20 - 200                |
| Coarse      | Multiple processes        | 200 - 2000              |
| Very Coarse | Distributed system        | 2000 - 1M               |
| Independent | Unrelated processes       | N/A                     |

The finer-grained the parallelism, the more care and attention needs to be given to scheduling.

## Processor Affinity

Let us imagine every processor has its own cache. In that case, we want *processor affinity*. After some period of time executing on a processor, its cache will have a lot of data. If the process begins executing on another processor, the data is in the "wrong" cache. Ideally, we keep executing on the same processor, when possible.

If the OS is going to make an effort but not guarantee that a process runs on a given processor, it is called *soft affinity*. A process can move, but will not if it can be avoided.

If it is guaranteed to run on one processor or a specified set of processors, it is called *hard affinity*.

Another reason we may want to lock a process to a certain processor is when memory accesses are non-uniform. If the CPU can access some parts of memory faster than others, the system has non-uniform memory access (ie: processors have faster access to their own local memory than another processor's memory).

## Load Balancing

If we have 4 processors, it is less than ideal to keep one at 100% utilization and have the other 3 do nothing. We want to keep the workload balanced across all systems. This is *load balancing*.

Load balancing is necessary when each processor has its own private queue of processes to run. If there is a common ready queue then load balancing will happen on its own.

There are two, non-exclusive approaches to redistributing the load: *push* and *pull* migration.

* With push migration, a task periodically checks how busy each processor is and moves around to balance things out.
* With pull migration, a processor with nothing to do "steals" from the queue of a busy processor.
