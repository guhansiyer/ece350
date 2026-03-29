# File Allocation Methods

There are three major ways we could allocate disk space to files: contiguous, linked, and indexed.

## Contiguous Allocation

A file occupies a set of contiguous blocks on a disk, starting at block $b$ and is $n$ blocks in size. So the file takes up all blocks in the range $\{b, b+1, b+2, ..., b+(n-2), b+(n-1)\}$. This is advantageous because if we want to access block $b$, accessing $b+1$ requires no head movement, so seek time is nonexistant to minimal.

We only require $b$ and $n$, the start location and file size, to be maintained.

One problem is the same as in memory allocation, if we need a block of size $N$, can we find a contiguous block of at least that size to meet that allocation, and if there are multiple, which do we choose? Additionally, external fragmentation is a concern. Finally, how much space is a file going to take? If the user opens a new document, how does the OS know how big the file will be? If we allocate too little space, we might be able to tack on additional blocks, or the file might have to be reallocated. If we allocate too much space, then significant space will be wasted.

## Linked Allocation

This is a solution to contiguous allocation. We maintain a linked list of the blocks, which might be located anywhere on the disk. The directory listing has pointers to the head and tail (first and last blocks).

A new file will be created with size zero and null head/tail pointers. When a new block is needed, it can come from anywhere and be added to the linked list. Accessing a random file block is not as simple as before, since instead of adding an offset to the first block, we have to follow the pointers to the desired block.

A solution to the problem of following and maintaining so many pointers is to group the blocks into clusters. Then, we waste less memory maintaining pointers, and there is less seeking back and forth to different disk locations.

One variation of this is the File Allocation Table (FAT), which was used in Windows operating systems up until NTFS was developed for NT and onwards. FAT32 hangs on as the file system of choice for USB flash drives, since it is supported across Windows, MacOS and Linux. 

FAT works as follows: a table for file allocation data is allocated at the beginning of the list, containing one entry per block and indexed by block number. It works like a linked list; the directory entry has the first block of the file and the table entry for that block number has the next block's index, and so on until the last block which has a end-of-file value. An unused block has a value of 0, so to allocate a new block, find the first 0-value entry and replace the previous end-of-file value with the new block address. The FAT itself should be cached in memory for performance.

## Indexed Allocation

Linked allocation, as discussed, makes random file-block access difficult. The idea of indexed allocation is to hold on to all the pointers and put them into one index block at the start of the file. To get to block $i$, go to index $i$ in the index block and we can get the location of block $i$ much more efficiently than in linked allocation.

Like the other systems, we must make a decision about block size. If a file only needs 1-2 blocks, one block is allocated for the pointers, which contain 1-2 entries. This suggests we want a small index, but what if there are more pointers than fit into one block? There are a few mechanisms for this:

1. Linked Scheme: index blocks can be linked together. The last entry is either null or a pointer to the next block.
2. Multilevel Index: linked scheme with multiple levels. The first level blocks point to the second, the second to the actual file data.
3. Combined Scheme: all-of-the-above; Unix's solution. Keep the first 15 index block pointers in the inode structure; 12 point directly to file data, 3 point to indirect blocks. The 13th is an index block containing the addresses of blocks with data. The 14th points to a double indirect block, the 15th to a triple indirect block.

## Consistency Checking and Journalling

An error, crash, power failure, or something similar may result in the loss of data or inconsistent file system data. The directory structures, pointers, inodes, FCBs and all are data structures that, if corrupted, may cause serious problems.

We could periodically check for inconsistent data, which many operating systems do, but this is a large operation that will consume a lot of time while the whole disk is scanned.

Recall atomic operations: either they succeed completely or do not take place. This is the approach that Windows NTFS and Apple HFS+ use: journalling.

All metadata changes are written sequentially to a log file. Once the changes are written to the log, the log entries are carried out and control may return to the requesting program. As changes are made, a pointer is updated to indicate which log entries have happened and which have not. When changes are completed in entirety, they are removed from the log file. If the system crashes, any changes remaining in the log file are carried out. If a change was aborted, we walk backwards through the log entries, undoing any completed operations to go back to the state before the start of said change.

Although a particular write might not have occurred because of a crash, resulting in some data loss, the system will always be in a consistent state.

In NTFS, journalling is used to make sure that system-maintained metadata is in a consistent state; user data can still be lost. This was an intentional design decision by Microsoft to simplify and speedup recovery operations.

