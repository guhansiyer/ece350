# Caching

The goal of caching is to speed up operations; it takes less time to fetch data from cache -> CPU, than from memory -> CPU.

Caches are useful anytime a large resource is divided into pieces, some of which are used more often than others.

Consider a CPU generating memory addresses for read/write operations.

The address will be mapped to a page. If the page is in the cache, it is a *cache hit*. If it is not in th ecache, it is a *miss*, in which case the page must be loaded from memory. A page miss is also called a *page fault*. The percentage of time that a page is found in the cache is the *hit ratio*. The effective access time can be computed as:

$\text{Effective Access Time} = h \times t_c + (1 - h) \times t_m$

Where $h$ is the hit ratio, $t_c$ is the time required to load a page from the cache, and $t_m$ is the time to load a page from memory.

Caches for memory are often multileveled, which changes the effective access time formula.

## Page Replacement Algorithms

When a page fault occurs, the OS needs to choose which page to evict from the cache, in order to accommodate the new page, assuming the cache is full. There are a variety of algorithms that can dictate which page to remove.

### Optimal

* Replace the page that will be most distantly used in the future.
* For each page, make a determination about how many instructions in the future that page will be accessed. The one with the highest value is selected.
* This is a **benchmark**, the algorithm itself is impossible to implement.

### Not Recently Uesd

* Computers may have two status bits $R$ and $M$ associated with each page. $R$ is set when a page is read/written to (referenced), $M$ is set when the page is strictly written to (modified). A bit can only be changed by the OS after being set.
* Initially, $R = M = 0$. Periodically, $R$ is cleared to distinguish which pages have been recently referenced. When a replacement must happen, the OS sorts the pages into 4 buckets based on the bits, in the following order of precedence:

    1. Not referenced or modified.
    2. Not referenced, modified.
    3. Referenced, not modified.
    4. Referenced and modified.

* The OS will remove a page from the lowest numbered class, when possible.

### First In First Out

* For $N$ frames, keep track of the frame to be replaced with a counter ranging from $0$ to $N - 1$.
* Sometimes works, but often the same page is repeatedly referenced, which this algorithm doesn't take into account.

### Clock Algorithm

* Improvement on FIFO.
* Give pages a "second chance" depending on the $R$ bit.
* If the oldest page hasn't been recently referenced, it is removed.
* If it has been recently referenced, the bit is cleared.
* Go to the next oldest page and repeat, until a page is removed.

### Least Recently Used

* The page to be replaced is the one accessed most distantly in the past.
* This can be implemented with a cyclic, doubly linked list.
* When a page in the cache is accessed, move it to the back of the list.
* When a page is not found in the cache, the page at the front is removed, and the new page is added to the back of the list.

### Not Frequently Used

* Similar to LRU, but can be implemented strictly in software (no reference and modification bits).
* Each page gets a software counter, starting at 0.
* When the $R$ bit would've been updated to 1, the counter is incremented.
* When a page must be replaced, the page with the lowest counter value is replaced.
* Problem: counters never decrease.
  * A page that may have been accessed frequently at the start of a program will have a high counter value, and may never be evicted despite not being accessed again.
* Solution: *aging*.
  * Counters are shifted to the right by 1 bit before they are incremented, and instead of actually incrementing, the leftmost bit is set to 1.

## Choosing an algorithm

* Optimal: Impossible to implement
* NRU: Subpar performance
* FIFO: Suboptimal
* Second Chance: Better than FIFO, but just adequate
* LRU: Best performance, hardest implementation
* NFU: LRU approximation
* NFU + aging: Better approximation of LRU

## Local and Global Algorithms

It is important to consider how multiple processes affect page replacement algorithms.

For example, when a process switch occurs, we could dump the contents of the cache for the next process, but this is often unnecessary work.

For the page replacement algorithms, it is important to consider whether it should care about which process a page belongs to.

Suppose we use LRU. If a process has a page fault, do we replace the least recently used page in all caches (global), or just in the cache that belongs to the process (local)?

In multilevel cache systems, we may have different strategies. If L1 is 16 KB and page are 4 KB, there can be 4 pages in the cache, so global replacement makes sense. If L2 is 256 KB, there are 64 pages, so local replacement could work.

Local algorithms give each process a fixed number of pages in the cache. Global algorithms dynamically allocate cache space to processes.

Suppose we have a large cache, and a global algorithm with some capability to give processes the "correct" number of pages in the cacche. To accomplish this, we may track the number of page faults in a measure called *PFF* (page fault frequency).

If PFF is above some upper threshold, more cache space is needed for a process. If it is below a lower threshold, the process has too much cache space allocated.

PFF relies on the assumption that the selected algorithm has the property that fewer page faults will occur if a process has more pages. This is true for some algorithms (LRU), but not others (FIFO).
