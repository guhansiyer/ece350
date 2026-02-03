# Scheduling Algorithms

A simple scheduling algorithm would be to always pick the highest priority, non-blocked task, and execute it. While simple, this approach has drawbacks. We examine the following options for scheduling algorithms:

1. Highest Priority, Period
    * Not difficult to implement; keep priority queues for each priority level and put non-blocked tasks in appropriate queue, if priority changes then move the task appropriately.
    * Drawback has been widely identified: starvation. If we have a bunch of low priority tasks, they may never get the chance to run.

2. First Come, First Served
    * Whichever process requests the CPU first, get the CPU (all processes are equal). If the current process finishes or gets blocked, select the next ready process.
    * The drawback results from different processes having vastly different completion times. If a process with a heavy completion time is at the front of the queue, it will raise the average completion time no matter how small the following processes completion time is.
    * Favours CPU-Bound processes over I/O-Bound. Because the disk is slow, we'd like to keep it busy. If a disk write is completed and there is a process that wants to read from disk, the ideal is to select that process.

3. Round Robin
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

4. Shortest Process Next
    * If we know anything about the total length of execution, we might want to give short processes priority.
    * We let them go first to free up the queue for bigger processes.
    * This is essentially addressing the main drawback of first come, first served.
    * Of course, the drawback is that the OS does not know much about the total length of execution.

5. Shortest Job First
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

6. Shortest Remaining Time
    * This is a modification of the previous strategy.
    * When a new process is scheduled or an old one is unblocked, the scheduler evalutes if there is a shorter predicted running time than the currently running process.
        * If there is one, the available process will be ran, displacing the currently running process.
    * There is a chance, just like the last algorithm. If we choose $S_1$ to be 0 for a new process, they will always displace the current process.
    * One advantage is we no longer need time slicing. Instead of interrupting the process every $t$ units of time, the other interrupts (user programs launching, hardware operations, etc.) will be what prompts the scheduler, which is a net performance increase.
