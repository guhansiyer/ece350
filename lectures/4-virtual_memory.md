# Virtual Memory

Even with paging and swapping, there are still limits on memory. Consider programs that require more memory than is physically available (server/supercomputers), multiple programs and multiprocessor systems (sum of memory requirements > available memory).

A process must entirely be in memory or on disk. If we could execute programs that are partly in memory, there would be 3 major benefits:

1. The size of physical memory no longer constrains programs.
2. Each program could use less physical memory.
3. Less I/O is needed to swap programs.

Many of the key ideas involved in caching apply to making such a system work: main memory can be viewed as yet another level of cache, and the disk is the last level. If a page is referenced and is not in main memory, it is a page fault, and the page is loaded from disk, which may require a page to be evicted.

The approach is that of the cache, which is demand paging; a page is loaded into memory only if referenced or needed.

Because the disk is slow, we can get into a state called *thrashing*, where the OS is spending most or all of its time swapping pages, so other work cannot be done.

With virtual memory, each reference is as follows:

1. Check if the reference is valid.
2. If it is invalid, terminate the program. If it is valid, but the page referenced is not in memory, continue.
3. Find a free frame, or make one.
4. Request a disk read to bring in the new page.
5. When the disk read is done, update the reords to show the new page in memory.
6. Restart the instruction that referenced the page that needed to be brought into memory.

## Performance

Recall the effective access time formula:

$\text{Effective Access Time} = h \times t_c + (1 - h) \times t_m$

If we replace $t_c$ and $t_m$ with $t_m$ and $t_d$ (time to retrieve from disk), and redefine $h$ as $p$, then for virtual memory we have:

$\text{Effective Access Time} = p \times t_m + (1 - p) \times t_d$

We can combine the caching and disk read formulas for true effective access time for a single-level cache:

$\text{Effective Access Time} = h \times t_c + (1 - h)(p \times t_m + (1 - p) \times t_d)$

## Frame Allocation

If there are $n$ free frames in a simple system, we demand page them all. Initially, all frames are empty, and pages are read into them as needed. When all $n$ frames are filled, page $n+1$ must replace a page already in a frame. When a process terminates, all its frames are marked as free. We can build from this.

We may reserve a few pages to be free at all times. When we want to move a page into a frame, if all frames are full, we select a victim and write it to disk if necessary. If we reserve free pages, then we can read the new page into the frame and write the old page at a convienient time.

If there are $m$ frames and the OS reserves $k$ of them, there are $m-k$ available frames for processes.

### Equal Allocation

If there are $n$ processes, each process gets $\frac{m-k}{n}$ frames. The leftover frames can be kept as free frames if there is a remainder. But as seen with caching, not all processes need the same amount of frames.

### Proportional Allocation

Let the virtual memory size of a process $p_i$ be defined as $s_i$, with $S = \sum s_i$. We then allocate $a_i$ frames to $p_i$, where $a_i = \frac{s_i}{S \times (m - k)}$.

$a_i$ is only an estimate; it may not divide evenly or may be below the minimum number of frames, so the value can be raised or lowered a bit accordingly.

This strategy has no regard for process priority.

## More on Thrashing

> How can we get out of thrashing?

We need fewer programs in memory at a time.

One solution is to force a local replacement policy rather than global replacement. If a process is thrashing, it cannot steal pages from another process. But if it is spending all its time paging to and from the disk, any other process that wants to use the disk will have to wait.

It is best to be proactive and deal with managing processes before thashing occurs.

Memory accesses tend to obey locality, temporally (memory locations recently accessed are likely to be accessed soon) and spatially (memory locations close by are likely to be accessed when one is accessed).

### Working-Set Model

We retain the last $n$ pages in memory, representing the locale of a program.

This is called $\Delta$, the working set window. Pages that have been recently used are in the working set, and pages will drop out of the set after $\Delta$ time units since its last reference, if not recently accessed.

Suppose the window is defined as ten accesses. Any page accessed in the last ten requests is part of the working set. If they are all in page $k$, then the working set will only contain $k$. So, the size of the set will change over time, and can be anywhere from 1 to $\Delta$ pages.

If $\Delta$ is too small, the set will not encompass the whole locale. If it is too big, the set will cover multiple locales.

Once a value of $\Delta$ is determined, the OS will monitor the working set and use it to figure out if the system is overloaded. If it is, the OS picks a victim and suspends it to prevent thrashing.

The page fault of a process varies over time. At the start of its lifecycle, there will be many page faults as the program starts up, then the rate will drop when established in a locale. When it moves to a new locale, the rate will rise again until it is "settled" in that locale.
