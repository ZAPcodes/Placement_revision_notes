# 05. Interprocess Communication and Signals

> IPC lets isolated processes exchange data and coordinate without giving up all protection.

## IPC design questions

When choosing a mechanism, ask:

- Same machine or network?
- Related parent/child processes or unrelated processes?
- Stream or message boundaries?
- Small control messages or high-volume data?
- One-to-one, one-to-many or many-to-many?
- Blocking, non-blocking or asynchronous behavior?
- Who provides synchronization and backpressure?
- Must data survive process termination?

## Mechanism comparison

| Mechanism | Data model | Strength | Responsibility/trade-off |
|---|---|---|---|
| Shared memory | Common mapped bytes | Fast after setup; avoids repeated kernel copying | Application must synchronize and define layout |
| Anonymous pipe | Unidirectional byte stream | Simple parent-child pipelines | No message boundaries; limited scope/lifetime |
| Named pipe/FIFO | Named byte stream | Unrelated local processes can connect | Local and stream-oriented |
| Message queue | Discrete messages | Preserves boundaries; kernel-managed coordination | Copying, limits and queue backpressure |
| Unix-domain socket | Local stream/datagrams | Bidirectional; familiar socket API | More protocol overhead than raw shared memory |
| Network socket | Network stream/datagrams | Cross-machine communication | Serialization, failure and protocol design |
| Memory-mapped file | File pages mapped into address spaces | Sharing plus file-backed persistence | Synchronization and consistency remain important |
| Signal | Small asynchronous notification | Lightweight control notification | Carries little information; restricted handler actions |

## Shared memory

Typical sequence:

1. Create/select a shared-memory object.
2. Map it into each process's virtual address space.
3. Processes access the same physical pages through different virtual addresses.
4. Use synchronization for shared mutable structures.
5. Unmap and remove according to lifetime rules.

Shared memory is often the fastest bulk IPC mechanism because ordinary loads/stores move data after mapping. It is not automatically safe, ordered or durable.

## Pipes

- A pipe is usually a kernel buffer with read and write endpoints.
- It is a **byte stream**; one write need not equal one read.
- Reads may block while the pipe is empty and writers still exist.
- A reader observes EOF only after all write endpoints are closed.
- Writing when no reader exists can fail and generate `SIGPIPE` on Unix-like systems.
- Shell pipelines connect one process's output descriptor to another's input descriptor.

### Classic pipe trap

A reader waits forever for EOF because a process accidentally retained an inherited write descriptor. Close every unused endpoint in parent and child.

## Message queues

- Preserve discrete message boundaries.
- Can add priorities/types, depending on API.
- Kernel or broker controls buffering and wakeups.
- Bounded queues create backpressure; unbounded queues risk memory growth.
- Delivery does not automatically mean processing succeeded.

## Sockets

- Unix-domain sockets communicate locally and may support credential passing.
- Network sockets cross machines.
- Stream sockets deliver an ordered byte stream, not application messages; framing is still required.
- Datagram sockets preserve message units but may not provide reliability/order.

## Memory-mapped files

- Map file contents into an address range.
- Page faults load pages on demand.
- Multiple processes can map the same file/pages.
- Updates may reach storage later through write-back; mapping does not equal immediate durability.
- Private mappings use copy-on-write for modifications; shared mappings can propagate changes.

## Signals

Signals are asynchronous notifications delivered to a process/thread.

- A signal has a disposition: default, ignored or handled.
- Signals may be blocked and delivered later when unblocked.
- Standard signals may coalesce; do not assume one queued delivery per occurrence.
- A handler interrupts ordinary execution, so only async-signal-safe operations should be used.
- Synchronous signals arise from the current execution, such as invalid memory access; asynchronous signals come from external events/processes.
- Signal disposition and delivery target details depend on whether the state is process-wide or thread-specific.

## Blocking, non-blocking, synchronous, asynchronous

These are different axes:

- **Blocking:** call may suspend the calling thread until progress/result.
- **Non-blocking:** call returns immediately if it cannot proceed.
- **Synchronous:** completion/result is tied to the call's control flow.
- **Asynchronous:** operation completes later and reports through event, callback, signal or completion queue.

A non-blocking program can still repeatedly poll synchronously. An asynchronous operation can be awaited in a way that blocks a thread.

## Backpressure

If producers outpace consumers, a finite system must eventually:

- Block/throttle producers
- Reject/drop work
- Spill to durable storage
- Scale consumers
- Degrade service

IPC design is incomplete without a queue-capacity and overload policy.

## Common traps

- Shared memory is fast because it avoids repeated data copying, not because it avoids synchronization.
- Pipes and TCP-style streams do not preserve application message boundaries.
- Closing one duplicate write descriptor is insufficient if others remain open.
- Signals are poor for transferring structured data.
- `mmap` does not guarantee that updates are already durable on disk.
- Local IPC can still fail due to process exit, permission, capacity or protocol errors.

## Interview checks

1. Shared memory versus pipe: when would you choose each?
2. Why does shared memory require locks?
3. Why might a pipe reader never receive EOF?
4. Pipe versus socket versus message queue?
5. How would you frame messages over a stream socket?
6. Blocking versus synchronous?
7. What may go wrong if a signal handler calls a non-safe library function?
8. How do you implement backpressure between processes?

## 60-second recall

- Shared memory maximizes throughput but delegates synchronization.
- Pipes/sockets are byte streams unless the API explicitly supplies message boundaries.
- Message queues preserve messages and provide buffering, but queues require overload policies.
- Signals notify; they are not general data transport.
- Blocking/non-blocking and synchronous/asynchronous are separate dimensions.

## References

- [OSTEP: Process and Memory APIs](https://pages.cs.wisc.edu/~remzi/OSTEP/)
- [MIT 6.1810: Unix utilities and system calls](https://pdos.csail.mit.edu/6.S081/2026/schedule.html)

