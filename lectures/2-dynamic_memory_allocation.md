# Dynamic Memory Allocation

In Java, the `new` keyword is used to create a new instance of an object. The runtime will garbage collect the object later on.

In C++, `new` and `delete` are used to allocate and deallocate objects respectively.

In C, memory allocation is done on a lower level with `malloc()` and `free()`.

For example, to allocate an integer, you call `malloc(sizeof(int))`. This creates a new integer in memory and returns its address, which can be stored in a pointer.

When `free()` is called on a pointer, the memory it occupied is marked as available, but might not be cleared or reused immediately after.

## Fulfilling Memory Requests

Note that `free()` does not specify how much memory is returned. This means that (a) the OS keeps track of each allocated block's size and (b) it is not possible to return part of a block.

We now turn our attention to fulfilling memory allocation requests.

The operating system will try to find some free memory to meet a request.

Although running out of memory in modern computers is a rare occurrence, there is the possibility that some request may not be fulfilled because no block meeting the need of the request is available.

## Fixed Block Sizes

One possibility for how to allocate memory is fixed block sizes; allocating all blocks of memory as the same size.

This doesn't mean requests aren't of varying size, just that all blocks allocated are the same size.

With this method, of course, some memory is wasted. Consider that if 1.5 blocks are requested, 2 blocks are returned. The extra 0.5 blocks can't be used for anything useful (it will show as allocated). This is *internal fragmentation*.

### One Block Size

If a system has only one block size, we can divide up the memory and maintain a linked list of available block addresses. When a block is allocated or freed, it is removed or added to the linked list accordingly.

### Fixed Block Sizes, Multiple Size Options

To accommodate different sizes of memory allocations, we can have different block sizes. Unfortunately, this still suffers from internal fragmentation.

## Variable Block Sizes

### Bitmaps

We can divide memory into $M$ units of $n$ bits and then make a bit array of size $M$ to store the unit statuses. If a bit has a status of 0, it is unallocated, if it has a status of 1, it is allocated.

### Linked Lists

Similar to fixed block sizes, linked lists can be used to allocate memory.

At startup, the list contains one entry, since all available memory is in a contiguous block.

When a request is allocated, the block is divided accordingly. Suppose we allocate a 128 byte request. A new entry gets added at 128 bytes in the list. The node contains the start address, block length, and an bit indicating that it is allocated.

The unallocated (remaining memory) node contains this updated entry: smaller size, new start address, bit indicating that it is unallocated.

When the block is deallocated, we find it in the linked list and set the bit to zero to indicate it is available.

After enough time allocating and deallocating memory, we may end up with the free blocks being small and spreadout. In this case, it may be that there is a contiguous block of free memory of size $N$ available, but the corresponding request can't be fulfilled because that memory is split up into smaller pieces.

To solve this, we need a way to recombine split blocks.

### Coalescence

This is the process of merging adjacent free blocks into larger blocks. This also makes it a good idea to maintain memory blocks in a doubly-linked list.

Even with coalescence, $N$ free bytes may exist in the system but are spread out over many pieces, so an $N$ byte request cannot be satisfied.

When free memory is spread into small fragments, this is *external fragmentation*.

### External Fragmentation

One way to reduce this is to increase internal fragmentation. If we get a request for $N$ bytes and there is a $N+k$ byte block available, where $k$ is very small (and unlikely to be independently allocated), it makes sense to allocate that block and accept that the $k$ bytes will be lost to internal fragmentation. This reduces small, scattered blocks of memory.

Another idea is *compaction* or *relocation*. The goal is to move allocated sections of memory next to each other in memory, creating a large contiguous block of free space. This, however, is very expensive.

The final way to prevent or deal with external fragmentation is different allocation strategies.

## Variable Allocation Strategies

1. **First fit**
    * Check each block from the beginning of memory onwards.
    * If the block is of sufficient size (at least $N$), split it as needed, and return the balance to the unallocated list.
2. **Next fit**
    * Instead of starting at the beginning of memory, keep track of where the last block was allocated, and begin searching from there for the next search.
    * This prevents external fragmentation at the beginning of memory.
3. **Best fit**
    * Instead of splitting up the first block equal to or larger than $N$, we choose the smallest block of all blocks that is at least as big as $N$. This way, we have the smallest remaining unallocated space.
    * This requires either checking every available block or keeping them sorted in increasing size.
4. **Worst fit**
    * With best fit, the leftover bits of memory are likely too small to be useful.
    * Instead, choose the largest block of free memory, so that the remaining split block is likely large enough to be useful.
5. **Quick fit**
    * Not a solution, but an optimization.
    * If memory requests of a certain size are known to be common, it could be ideal to keep a separate list of blocks that are within a suitable range of that size (ie: 1-1.1 MB for 1 MB requests) to satisfy those quickly.

Generally, first fit performs the fastest and best.

## Advanced Strategy: Binary Buddy

In a buddy system, blocks are of size $2^K$ where $L \leq K \leq U$, $2^L$ is the smallest block size and $2^U$ is the largest block size we can allocate.

Initially memory is treated as a single block of size $2^U$. If a request of size $n$ occurs such that $2^{U-1} < n \leq 2^{U}$ then the whole block is allocated. Otherwise, the block is split into two "buddies" of size $2^{U-1}$. If $2^{U-2} < n \leq 2^{U-1}$, allocate one of the blocks of $2^{U-1}$ to the request. Otherwise, subdivide again. This repeats until the smallest block greater than or equal to $n$ is allocated. 

In subsequent allocations, we can search the data structure to find either a block of appropriate size or a block that can be subdivided to meet the allocation.

Whenver a pair of buddies are both free, they can be coalesced.
