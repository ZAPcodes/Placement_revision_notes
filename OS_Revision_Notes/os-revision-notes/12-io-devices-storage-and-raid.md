# 12. I/O Devices, Storage and RAID

> I/O systems bridge very fast CPUs/memory with slower, diverse devices while limiting CPU overhead and preserving correctness.

## Device path

```mermaid
flowchart LR
    A["Application"] --> B["System call"]
    B --> C["Generic OS I/O layer"]
    C --> D["Device driver"]
    D --> E["Controller / device"]
    E -->|"interrupt / completion"| D
```

- **Device controller:** hardware interface managing a device.
- **Driver:** kernel software translating generic requests into device commands.
- **Device registers:** command, status and data interfaces.
- **Memory-mapped I/O:** device registers appear in an address range.
- **Port-mapped I/O:** special I/O instructions/address space, on supporting architectures.

## Polling, interrupts and DMA

| Method | CPU behavior | Good fit | Main cost |
|---|---|---|---|
| Polling | Repeatedly checks status | Very fast/expected completion or simple hardware | Wasted CPU cycles |
| Interrupt | Device notifies CPU | Infrequent/unpredictable events | Entry/handler/context overhead |
| DMA | Controller transfers bulk data between device and memory | Large transfers | Setup, mapping, cache/coherency and completion handling |

DMA does not remove CPU involvement: CPU/driver configures descriptors, handles completion/errors and manages buffers.

High-rate devices may combine interrupts with batching and polling to avoid interrupt storms.

## Blocking and asynchronous I/O

- **Blocking:** caller may sleep until operation can make progress/completes.
- **Non-blocking:** returns immediately when it would otherwise wait.
- **Synchronous:** caller observes completion through the operation's flow.
- **Asynchronous:** submission and completion are separated.

These terms are not interchangeable. Readiness notification (`select`/`poll`/`epoll`) says an operation is likely to make progress; completion APIs report completed operations.

## Buffering, caching and spooling

| Technique | Purpose |
|---|---|
| Buffering | Smooth size/speed mismatch and batch transfers |
| Caching | Retain copies of likely reused data |
| Spooling | Queue complete jobs for a serially shared device, such as a printer |

The same memory region may serve more than one role, but the concepts differ.

## Character vs block devices

- **Character device:** stream-oriented access, often no random block addressing.
- **Block device:** fixed-size addressable blocks, usually supports random access and caching/filesystems.

## HDD characteristics

Access time includes:

- Seek: move head to track.
- Rotational delay: wait for sector.
- Transfer: move bytes.

Schedulers historically reorder requests to reduce mechanical movement.

## Disk scheduling

| Algorithm | Rule | Main concern |
|---|---|---|
| FCFS | Arrival order | Fair/simple but excessive movement |
| SSTF | Closest request | Good average seek; starvation possible |
| SCAN | Sweep both directions | More predictable, elevator-like |
| C-SCAN | Service in one direction, jump back | More uniform waiting |
| LOOK | SCAN but reverses at last pending request | Avoids travel to physical end |
| C-LOOK | Circular LOOK | Uniform circular service without end travel |

For SSDs, mechanical seek optimization matters far less, but queueing, parallelism, fairness and write behavior still matter.

## SSD concepts

- Reads/writes operate on pages; erase occurs in larger blocks.
- Updating data often writes elsewhere and invalidates old data.
- Flash Translation Layer maps logical to physical locations.
- Garbage collection reclaims blocks.
- Wear levelling spreads erasures.
- TRIM/discard informs device which logical blocks are no longer needed.
- Write amplification means physical writes can exceed logical writes.

## RAID summary

| Level | Layout | Minimum disks | Capacity concept | Tolerates |
|---|---|---:|---|---|
| RAID 0 | Striping | 2 | Sum of disks | No disk failure |
| RAID 1 | Mirroring | 2 | One copy per mirror set | Typically one disk per mirror set |
| RAID 5 | Block striping + distributed single parity | 3 | N-1 disks | One disk |
| RAID 6 | Distributed double parity | 4 | N-2 disks | Two disks |
| RAID 10 | Stripe across mirrored pairs | 4 | About half raw capacity | Depends which disks fail; one per affected pair |

Assume equal-sized disks for simple capacity reasoning; usable capacity is constrained by the smallest member.

### RAID trade-offs

- Striping increases parallel throughput but not redundancy.
- Mirroring gives simple recovery/read options at capacity cost.
- Parity saves capacity but makes small writes more complex.
- Rebuilds stress remaining disks and increase risk/exposure.
- RAID protects availability from certain device failures; it does not protect against deletion, corruption, ransomware or site loss.

## I/O scheduling layers

Reordering may happen in application, OS, filesystem, block layer, controller and device firmware. Therefore textbook head-scheduling algorithms are conceptual models, not the complete modern storage stack.

## Common traps

- DMA still requires setup and completion handling.
- Interrupts are not always better than polling.
- Non-blocking and asynchronous are not synonyms.
- SSDs have no seek head but do have erase, garbage-collection and parallelism concerns.
- RAID 0 is not redundant despite the name RAID.
- RAID is not backup.
- RAID 10 can survive some two-disk failures but not every two-disk combination.
- Buffering, caching and spooling solve different problems.

## Interview checks

1. Polling versus interrupts versus DMA?
2. Why might a network driver switch from interrupts to polling under load?
3. Blocking versus non-blocking versus asynchronous I/O?
4. Character versus block device?
5. Why can SSTF starve a request?
6. SCAN versus LOOK? C-SCAN versus C-LOOK?
7. Why is HDD scheduling less relevant for SSDs?
8. Explain SSD write amplification.
9. Compare RAID 0, 1, 5, 6 and 10.
10. Why is RAID not a backup?

## 60-second recall

- Driver controls hardware; interrupts report events; DMA moves bulk data.
- Polling trades CPU time for lower/simple completion handling.
- Blocking and sync are separate from non-blocking and async.
- HDD cost includes seek/rotation; SSD cost includes erase/GC/write amplification.
- RAID trades capacity, performance and failure tolerance; it never replaces backup.

## References

- [OSTEP: I/O Devices, Disks, RAID and SSDs](https://pages.cs.wisc.edu/~remzi/OSTEP/)
- [MIT 6.1810: Device Drivers](https://pdos.csail.mit.edu/6.S081/2026/schedule.html)

