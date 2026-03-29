# Reliability

It is entirely possible that a system's hard drives could die. If this happens, what do we do? Backups are a solution but not the only solution against data loss.

Backups are not always up-to-date, writing data from the backup to a new disk takes a nontrivial amount of time, and they are really only good if you've tested that you can restore from them successfully.

There is a solution that avoids these problems of reduced reliability and availability: RAID (Redundant Array of Independent Disks).

## RAID

The idea is to have multiple independent disks that work together but appear to the user as one volume.

If we want improved reliability, the cost is in redundancy; having more than one copy of the data stored in multiple mediums. Either we spend more money for more disks, or with a fixed budget, we trade capacity for redundancy. 

With two drives of the same size, we can mirror writes to both disks. If one disk dies, the other disk has a complete copy of the data.

## Levels

RAID is described as having different levels, each being different configurations of the drives. Some have more redundancy at the cost of space, others have less. There are also performance benefits to choosing one configuration over another.

### RAID -1

Not a real level; just the "do nothing" option. We have a bunch of disks that we choose not to configure into a RAID array, instead leaving them as individual disks.

### RAID 0

Disk striping. The data and files are split across multiple disks. 

This is fast, as we can exploit parallelism across the disks, but there is no redundancy. If one drive dies, all data is lost. 

This sounds counterintuitive, but the high throughput is the draw, so for any systems or use cases where we have temporary data or data replicated elsewhere, this may be a useful configuration.

### RAID 1

Mirroring. As discussed previously, writes to a disk are replicated across another. This works for an even number of disks.

Read speed is improved, since data can be found on one of two disks, so the less-busy disk can be accessed. Larger requests can read in parallel.

Write speed is no worse than writing directly to one disk. If one is busier than the other, the write is not fully complete until it has been written to both disks. Alternatively, one disk can be written to and the replication can be asynchronously done.

If the replication fails, we have solutions: atomic operations, recovery, non-volatile RAM, etc. If one disk is unavailable, we can get whatever data we need from the surviving disk.

If we have two disks, our usable capacity is half the purchased capacity, because each disk is a copy of the other. If we have four disks, two disks are copies of the other two. This is great for reliability, but if we are guarding against the failure of one drive, why do we have so many backups?

### RAID 2

Bit Parity; essentially data striping at the bit-level. If the first bit of each byte is stored on disk 1, second bit on disk 2, and so on up to bit 8, and an additional disk stores parity bits, then in the event that one disk is lost, the parity bit informs the lost bit value. Recall that with more parity bits, we have greater ability to detect and possibly correct errors (meaning more drives).

A read or write will engage all disks in the array allowing for quick reads and writes, but it is painful to split up reads and writes to the bit level.

### RAID 3

Byte Parity. This is similar to RAID 2, but the bytes are spread across all disks and the parity bits are computed for the bits at the same position on all disks.

Like RAID 2, all reads and writes are synchronous across all disks, but unlike RAID 2, only one extra disk is required for the parity information.

### RAID 4

Block Parity. Block-level striping like RAID 9 with parity blocks on a dedicated parity disk. Unlike RAID 2 or 3, reads and writes need not be synchronous because we are working with whole blocks.

### RAID 5

Distributed Block Parity. Instead of one disk holding all the parity information, it is spread out across all disks. One disk does not hold its own parity information because its data would be unrecoverable if the disk is lost.

RAID 5 is considered the most common parity RAID system.

### RAID 6

RAID 5 with extra information to survive disk failures.


## Combining RAID Levels

RAID can be combined, examples being RAID 01, 10, 50. It is better to think of these like 1 + 0 instead of ten; we are combining them at different levels.

Consider RAID 0 + 1: we have two sets of disks combined in RAID 0. On top of that, we build in mirroring. So, if we have four disks in total, we have a RAID 0 of disks 1 and 2, another RAID 0 of disks 3 and 4, and a RAID 1 array of disks 1+2 and 3+4. 

## Downtime

In the context of real-time systems, downtime may be intolerable in the case of life/safety-critical systems. 

We'd generally like a real-time system to have resiliency and fail-soft operation; a system that is more resilient is capable of carrying on in the event of failure, and fail-soft operation says that if things fails, the system will preserve as much capability as it can or terminate gracefully if it must.


## Resiliency

Let's imagine that something has gone wrong within a system. There are a few ways to respond or handle it.

The best option for resiliency is if the system can correct the problem and carry on. Fixing some things may be easier for software problems (ex: deadlock). 

During recovery, however, the system is probably not at full capacity. In this case, the next best thing is for the system to do the best it can with its capabilities.

Suppose the system has 4 CPU cores and one dies, but the CPU was never used at more than 50% capacity even when all cores were healthy. The system is worse off and has reduced maximum capacity, but in practice is running at its original capacity.

If the system is running at reduced capacity until external forces come to repair it, in the context of an RTOS, this may mean it is no longer possible to meet all deadlines. The system is considered *stable* if it will always meet the deadlines of its most critical tasks, even if lower priority tasks don't get completed.

It is possible to do an orderly shutdown when things go wrong instead of throwing an emergency stop. In Unix, if kernel data corruption is detected, the contents of memory are written to disk for analysis and execution is stopped.

## Faults and Fault Tolerance

A failure is when the response deviates from the specification as a result of an error, and an error is a manifestation of a fault. A fault, then, is an erroneous hardware or software state of some variety:

* Permanent: always present after occurring, can only be fixed by replacing or repairing the faulty component.
* Intermittent: occurs at random, unpredictable times.
* Transient: occurs once but does not recur.

### Fault Tolerance

Some things we've discussed earlier on play a role in supporting system resilience against software faults:

* Process Isolation: the OS prevents one process from accessing the memory of another process or the memory of the OS. If something goes wrong in one process, it does not affect other processes or the OS.
* Dual-Mode Operation: some things can only be done in a privileged mode.
* Preemptive, Priority-Based Scheduling: misbehaving processes or threads have limited impact on other threads that want to run.
* Checkpoints, Atomic Operations, Rollback