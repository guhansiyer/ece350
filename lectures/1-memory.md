# Memory

## Memory as a resource

Application developers behave as if:

1. Main memory is unlimited
2. All of main memory is at a program's disposal

> Why do program developers behave as if main memory is unlimited and unshared?

Compared to the early days of computing, available memory is huge. Early systems (ie: Commodore 64; 64 KB of memory) had very limited memory. In these times, developers had to think about and use memory very efficiently.

This is why languages like C and Java support types like `short`. In modern times, will anyone notice if we waste 1000 integers?

This is "kicking the can down the road". Even though memory might be big enough for every process, it still runs into problems if every developer assumes memory is unshared.

The solution to this is that modern OS's managed shared memory for the applications. This is one of the major objectives of the modern operating system, to manage shared resources, which **is** main memory.

## Memory Management

The simplest way to manage memory is to not manage it at all.

Programs in early computers would operate directly on memory addresses.

The section of memory that is accessible to a program depended heavily on the OS (if any), so we'd need to know the start and end addresses to write a program. This was not consistent across machines, so it was very difficult to write a cross-platform program.

One solution is as follows. On every process switch, save the entire contents of memory to the disk. Then, restore the contents of the next process to run. This is very expensive, but avoids the problem posed by running more than 1 process at once.

Another problem is that there is no protection for the OS. The OS is placed in either low or high memory, so an errant access could overwrite parts of the OS.

### Base and Limit

A solution to this problem is to add some additional information. The IBM 360 divided memory into 2 KB block and assigned each block a 4-bit key. The Program Status Word (PSW) also had a 4-bit key. If the hardware identified a discrepancy between the two, it would prevent the memory access.

To generalize this, we maintain a *base* and *limit* address to define the start and end of a program's memory. Every access is compared to the base and base + limit addresses. If the access falls outside that range, it is in error. Because of how often this operation is executed, the comparison is done in hardware and the base and limit addresses are held in registers.

However, this still does not solve the problem.

### Base and Limit Problem

Imagine we have two programs, each 16 KB in size:

> Program 1

```nasm
0           13680
.
.
.
ADD         28
MOV         24
            20
            16
            12
            8
            4
JMP 24      0

```

> Program 2

```nasm
0           13680
.
.
.
CMP         28
            24
            20
            16
            12
            8
            4
JMP 28      0

```

We load them into consecutive areas of memory:

```nasm

0           32764
.
.
.
CMP         16412
            16408
            16404
            16400
            16396
            16392
            16388
JMP 28      16384
0           13680
.
.
.
ADD         28
MOV         24
            20
            16
            12
            8
            4
JMP 24      0
```

The problem is that both programs have been written assuming they begin at address `0`.

The intent of the instruction at address `16384` is to jump to the instruction at offset `28` from the current address, but with no relocation, the machine will jump to address `28`, which is part of program 1's memory.

### Static Relocation

The IBM 360 solved this with static relocation; a program loaded into a base address beyond the start of memory would have that starting address act as a constant to be added to every program address while loading (ie: a program loaded to `16384` would have `16384` added to all addresses in the program).

This solution does not pose more problems to overcome, but is nonetheless challenging. For example `JMP 28` must be relocated, but in `ADD R1, 28`, `28` is a constant and should not be relocated.

> How do we know what's an address and what is constant?

### Variables

When writing programs, variables are not usually referred to by their location in memory.

When we write `x = 5`, we know `x` is stored in memory, but we do not know when it is assigned a place in memory.

There are three opportunities to assign a variable into memory:

1. Compile time
2. Load time
3. Execution time

## Address Space

It is clear that having no memory management leaves us with a number of problems.

We need an abstraction; a layer of indirection.

We do this with a concept called **address space**.

An address space is a set of addresses that a process can use. Each process has its own independent address space.

An analogy for address spaces can be found in phone numbers.

Phone numbers have the form XXX-XXXX. Any number in the range 000-0000 to 999-9999 could be issued, save certain reserved numbers.

There are likely more phones than available numbers if we left the numbers in this form, so we attach a three digit area code to the start of the number (ie: 226 in Southwestern Ontario, 416/647 in Toronto).

The same idea can be applied to processes. Process 1 and process 2 could write to address `1024`, and these end up being different locations, such as `21024` and `51024`.

### Efficient Relocation

The address generated by the CPU, like the `28` in `JMP 28`, is the **logical address**.

Then we add the "area code" to produce the **physical memory address**.

For performance, this is done through hardware. The "area code" is a register called the **relocation register**.

The process does not know the physical address, only the logical address. Essentially what we have is a run-time mapping of variables into memory.

This also allows us to relocate a process if we change the relocation register's value accordingly.

The cost of this effort is that every memory access now includes an addition operation (relocation register + logical address), which is costly for the CPU.

## Swapping

Processes must be in main memory to run. Given enough demanding processes, it will not be possible to keep them all in memory at once.

**Swapping** comes in handy when we have blocked processes taking up space that we want to replace with ready-to-run processes, so we can move them from memory to disk (or vice-versa).

The downside is that swapping is painful. We have to write the entirety of the process from memory to the disk, or vice-versa, which of course becomes costly as memory usage increases.

When we swap processes, it does not have to go back to the same place in memory. We just need to update the relocation register.
