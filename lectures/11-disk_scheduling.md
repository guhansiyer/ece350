# Disk Scheduling

As far as the CPU is concerned, the disk is a very slow device, so we want to avoid going to disk as much as possible. 

A hard disk consists of various platters, whose surface is divided into circular tracks, which are respectively divided into sectors. A set of tracks vertically stacked is called a cylinder. The platters are connected at their center via a rotating spindle, and a read-write head is suspended slightly above the surface of each platter. When the disk is in use, a motor spins the platters at high speed and another one manipulates the position of the arm.

The performance of the hard disk can be broken down into two values:

* Transfer rate: the speed at which data can be moved between the computer and disk.
* Random access time: how long it takes to get a particular piece of data. This is further broken down into two values
  * Seek time: how long it takes for the disk arm to be moved into the correct position.
  * Rotational latency: how long it takes to rotate the platters to the right position.
  
Seek times and rotational latency tend to be in the millisecond range.

The total average access time $T_a$ for a disk operation can be defined by:

$T_a = T_s + \frac{1}{2r} + \frac{b}{rN}$

Where $T_s$ is the average seek time, $r$ is the rotation speed, $b$ is the number of bytes to transfer, and $N$ is the number of bytes on a track.

## Disk Scheduling

Bandwidth is defined as the total number of bytes transferred divided by the total time between the service request and transfer completion. This is a measure of how much data is effectively transferred in a period of time. This, along with access time, are the measures we'd like to improve.

When a process needs to read/write to/from the disk, the system call contains this information:

1. If the operation is a read or write.
2. The disk address for the transfer.
3. The memory address for the transfer.
4. How much data to transfer (how many sectors)

Random scheduling acts as a baseline to compare various algorithms. We can assess the options based on their improvements relative to random scheduling.

1. First-Come, First-Served
2. Shortest Seek Time First
   * Instead of picking the next item in the queue, pick the request with the least seek time from the current head position.
   * Subject to starvation in many circumstances (i.e.: many low-number requests starving high-number requests).
3. SCAN Scheduling
   * Move in one direction until we reach the "end" of the disk, then reverse directions.
4. C (Circular)-SCAN Scheduling
   * Exploits the fact that when the disk reaches one end, most requests are likely at the other end.
   * Instead of reversing directions and servicing requests on the way back, jump to the other end of disk and start from there.
   * If it takes $t$ to scan from the start to end of disk, the expected service interval for sectors at the edges is $2t$ with SCAN. With C-SCAN, the interval is reduced approximately to $t + s_{max}$, where $s_{max}$ is the maximum seek time.
5. LOOK Scheduling
   * Optimization on SCAN and C-SCAN.
   * Instead of going to the end of the disk each time, go to the final request and only then reverse direction or go back to the start.
   * Has circular variant C-LOOK.

## Additional Scheduling Details

Excluding FCFS, which is decidedly not the best choice, how do we decide between the other algorithms?

LOOK seems to prevent starvation, which is otherwise an issue with SSTF. We expect that with SCAN/C-SCAN all requests get serviced, but it could happen that a process is starved or has to wait an inordinately long time because the request at an end of the disk is constantly postponed as more requests come in.

To prevent this, we can modify SCAN so that we have two queues for requests; while one is being emptied, the other is being filled.

The discussed algorithms only consider seek time and not rotational latency, even though they can be about the same size. This is because it is very difficult for the OS to schedule for improved rotational latency, since the disk itself is responsible for the physical placement of logical blocks. To make this a bit easier, the disk controller takes on some of the scheduling options. The OS can provide the controller with a grouping of requests, and the controller figures out how they are scheduled to take into account rotational latency in addition to seek time.

If the speed of disk reads and writes were the only matter of concern, the OS would probably not worry about disk scheduling and it would just be the hard disk controller's job. But the OS may have certain goals that should take precedence over the highest performing option. For example, loading a page into memory might need to take priority over an application writing a file to disk. 