# File Systems

The implementation of the file system is relatively complicated; to keep things manageable we are concerned with storing files on hard disks.

The layers of the file system design consist of:

* User Interface
* I/O Control
* Basic File System
* File Organization Module
* Logical File System

## Disk Organization

There are a lot of different file systems that are significantly different, but there are some general principals among them. A file system will need to track the total number of blocks, the number and location of free blocks, the directory structure, and the file themselves.

On at least one disk in the system, there needs to be some information on how to boot the system. If there is an OS, the boot loader is usually put in the first block. When the system is powered up, the BIOS starts up and transfers control to whatever is at the first block.

Disks may be split into different logical areas, called *partitions*. There will also be a partition table that indicates what part of the disk belongs to which partition. In Windows, the whole disk is usually in one partition (C: drive). In Linux there are often various partitions for different purposes.

There are a few structures that are likely to be in memory for performance reasons:
* Mount Table: information about each mounted volume (disks and partitions)
* Cache: directory info for recently accessed directories
* Global Open File Table: Copy of the FCB (file control block) for each open file.
* Process Open File Table: process references to the global open file table.
* Buffers

The logical file system is responsible for creating new files by allocating or reusing FCBs. A typical FCB might contain file permissions, dates (create, access, write, etc.), owners, size and data blocks.

If a user wants to make use of a file, the `open` system call is needed, as it operates on names. The file system must then check the global table to see if the file is already open somewhere in the system. If so, and the file is open for exclusive access, then the open routine returns with error. If the file is open for non-exclusive access, we can make another reference in the process table. If the file is not open, it needs to be retrieved, its FCB needs to be added to the global table, and a reference in the process table is added.

The process table can contain more info, like the next section to read or write, the access mode when the file is open, etc. `open` returns a pointer to the file table, which is where all file operations are handled through. This pointer is referred to as a *file descriptor* in Unix systems.

The opposite operation is to close a file. When a process closes a file, the reference in the process table is removed. If this is the last reference to the file in the table, then the global table entry is also removed.

## Virtual File System

There are a lot of different file systems, which may co-exist in a system, but a partition will only be formatted in one (can't be split across file systems). To the user, these operate identically, through a layer of abstraction called the *virtual file system (VFS)*.

The VFS has two main purposes: separate file system operations (open, close, read, write) from their actual implementations, and to provide a mechanism for representing a file throughout a network.

The representation structure is called a *vnode* which is like an *inode*, but are generic to allow interopability across file systems.

## Directory Implementation

A directory is not much beyond a set of files. But there is a choice to be made regarding implementation. Some immediate choices include linear lists and hash tables. Linear lists are simple but the searching operations they require are taxing. Hash tables are more efficient but will require a collision strategy. The better suited structure is a tree (B-Tree in our instance).