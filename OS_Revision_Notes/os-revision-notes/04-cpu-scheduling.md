# 04. CPU Scheduling

> Scheduling chooses which runnable thread receives a CPU and for how long.

## Core terms

| Metric | Meaning |
|---|---|
| CPU utilization | Fraction of time the CPU performs useful work |
| Throughput | Completed jobs per unit time |
| Turnaround time | Completion time - arrival time |
| Waiting time | Total time spent in ready queue |
| Response time | First run time - arrival time |
| Fairness | Reasonable access across competing work |

Turnaround is not response time. An interactive request may start quickly but finish much later.

## Scheduling points

The scheduler may run when a thread:

- Becomes blocked
- Exits
- Is preempted by timer
- Wakes another/higher-priority thread
- Changes priority or CPU affinity

**Non-preemptive:** running task keeps CPU until it blocks/exits voluntarily.  
**Preemptive:** kernel can interrupt and replace it.

## Algorithm comparison

| Algorithm | Core rule | Strength | Main weakness |
|---|---|---|---|
| FCFS | Earliest arrival first | Simple, low overhead | Convoy effect; poor response |
| SJF | Shortest next CPU burst | Minimum average waiting if burst known | Burst unknown; long-job starvation |
| SRTF | Preemptive SJF | Strong mean response/turnaround | More preemption; starvation |
| Round Robin | Fixed time quantum | Fair interactive response | Quantum trade-off and overhead |
| Priority | Highest priority first | Expresses importance | Starvation; priority inversion |
| Multilevel Queue | Fixed queues/classes | Simple class separation | Inflexible; lower queues starve |
| MLFQ | Feedback moves tasks between queues | Approximates short jobs interactively | Complex tuning; gaming/history effects |

## FCFS and convoy effect

A long CPU-bound job at the front forces many short/I/O-bound jobs to wait. Devices may become idle while short jobs wait behind the long job, reducing responsiveness and utilization balance.

## SJF and SRTF

- SJF chooses the smallest predicted next CPU burst.
- SRTF preempts when a job with less remaining time becomes ready.
- SJF minimizes average waiting time for a fixed known workload, but exact future bursts are normally unavailable.
- Systems estimate bursts from history, often weighting recent behavior.

## Round Robin

- Each runnable task receives at most one quantum before moving to the back.
- Very large quantum approaches FCFS.
- Very small quantum improves apparent responsiveness but increases switching overhead.
- A good quantum is much larger than switch cost while still meeting response goals.

## Priority scheduling

- Priority can be static or dynamically adjusted.
- **Starvation:** low-priority work waits indefinitely.
- **Aging:** gradually increases priority of waiting work.
- **Priority inversion:** a high-priority thread waits for a lock held by a low-priority thread while medium-priority work prevents the low-priority holder from running.
- **Priority inheritance:** temporarily boosts the lock holder.

## Multilevel Feedback Queue intuition

Typical behavior:

- New jobs begin at high priority.
- Jobs consuming full quanta move downward.
- Jobs that block quickly tend to remain interactive/high priority.
- Periodic boosts prevent permanent starvation.

MLFQ learns from observed behavior rather than knowing job length.

## Multiprocessor scheduling

- Per-CPU run queues reduce contention.
- Load balancing moves runnable tasks between CPUs.
- CPU affinity preserves cache locality.
- A globally balanced schedule may still perform poorly if migration destroys locality.
- Scheduling domains may reflect cores, shared caches and NUMA nodes.

## Real-time overview

| Hard real-time | Soft real-time |
|---|---|
| Missing deadline can be system failure | Occasional miss degrades quality |
| Requires bounded worst-case reasoning | Optimizes typical/tail response |

- **Rate Monotonic:** fixed priorities; shorter period gets higher priority.
- **Earliest Deadline First:** dynamically chooses closest deadline.
- General-purpose scheduler fairness does not imply deadline guarantees.

## Common traps

- Scheduling is usually performed for threads, though textbooks often say processes.
- Waiting time counts ready-queue time, not I/O-blocked time.
- Preemptive scheduling needs a mechanism such as timer interrupts.
- Fairness can conflict with throughput or latency.
- SJF's optimality assumes known burst lengths and a specific objective.
- Round Robin does not guarantee equal completion times.
- Priority inversion requires a dependency; it is not merely a low-priority task waiting.

## Interview checks

1. Why does SJF minimize average waiting time?
2. What happens as Round Robin quantum becomes extremely large or small?
3. Explain convoy effect with CPU-bound and I/O-bound jobs.
4. Starvation versus priority inversion?
5. How does MLFQ approximate SJF?
6. Why might the scheduler keep a task on a busy CPU instead of migrating it?
7. Does a fair scheduler guarantee low latency?
8. Why cannot a general-purpose scheduler guarantee hard deadlines?

## 60-second recall

- Response measures first service; turnaround measures completion.
- FCFS is simple but suffers convoy effect.
- SJF is mean-wait optimal with known bursts; SRTF is its preemptive form.
- RR trades response against switch overhead through its quantum.
- Priority needs aging; dependency inversion may need priority inheritance.
- Multicore scheduling balances work against locality.

## References

- [OSTEP: CPU Scheduling, MLFQ and Multi-CPU Scheduling](https://pages.cs.wisc.edu/~remzi/OSTEP/)

