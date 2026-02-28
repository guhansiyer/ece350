# Uniprocessor Scheduling

There are four types of scheduling:

1. Long-Term
2. Medium-Term
3. Short-Term
4. I/O (not covered until later)

> Analogy: Long-term scheduling is deciding what courses to take in a degree. Medium-term is deciding what courses to take in a given term. Short-term is deciding what course to study right now.

## Long-Term Scheduling

The long-term scheduler is responsible for determining what programs will run at all. This does not occur often on consumer systems, since the user broadly controls how many processes are running at a given time.

Sometimes, long-term scheduling is used by remote services trying to manage resources. An example is a game on launch day giving an error saying that the server is busy or full.

## Medium-Term Scheduling

Medium-term schedulers revolve around swapping processes to-and-from the disk. When a process is swapped to the disk, it is out of the realm of the short-term scheduler, and it comes back into that realm when it is back in memory.

Medium-term schedulers serve less purpose with the advent of SSDs, since disk transfer times are better than with HDDs.

## Short-Term Scheduling

Short-term schedulers, also called dispatchers, are invoked frequently by the OS.

With co-operative multitasking, short-term schedulers only takes place if the currently executing thread yields or terminates.

Co-operative multitasking is a bit problematic because of this, so we will discuss pre-emptive multitasking, where the OS is responsible for deciding when to switch threads, not the running thread (exception: thread voluntarily terminates).

There are certain times to make a scheduling decision. The dispatcher will run when a thread becomes blocked. It will also run after handling an interrupt (the original thread is suspended so why not leave it in that state?).

There is also time slicing. If time slices are defined as $t$ units, and a thread executes for $t$ time units, the clock will generate an interrupt to run the short term scheduler and choose a thread to run.

## Scheduling Criteria

In order to evaluate scheduling algorithms, we require some criteria:

1. Turnaround Time: amount of time between a thread starting and finishing (execution time + time waiting for resources).
2. Response Time: time between putting in a request and getting some answer back.
3. Deadlines: more of a concern for real-time systems that have specific deadlines.
4. Predictability: jobs should run in a fairly consistent amount of time.
5. Throughput: the number of threads that complete in a given amount of time should be maximized.
6. Processor Utilization: how much of the time the CPU is busy.
7. Fairness: threads should get a basic level of fairness (no starvation).
8. **Priorities**: processes and threads can be assigned priorities, which a scheduler should respect within limits.
9. Balancing Resources: CPU and I/O should be kept equally busy. The more information about what I/O a thread requires, the better.

## Scheduling Algorithm Goals

Different systems have different priorities:

### All Systems

* Fairness
* Priorities
* Balancing Resources

### Batch Systems

* Throughput
* Turnaround time
* CPU Utilization

### Interactive Systems

* Response Time
* Predictability
