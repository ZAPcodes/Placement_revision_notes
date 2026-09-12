# 08. Memory Management

> Memory management gives each process a protected address space while allocating finite physical memory efficiently.

## Address vocabulary

- **Source-level name:** variable/function identifier.
- **Logical/virtual address:** address generated in a process's address space.
- **Physical address:** location used to access RAM.
- **Address translation:** MMU transforms virtual to physical using OS-managed metadata.
- **Relocation:** allows a program to run at addresses different from those assumed earlier.

In modern systems, “logical” and “virtual” are often used similarly, though textbook contexts may distinguish them.

## Address binding

| Binding time | Consequence |
|---|---|
| Compile time | Absolute location assumed; relocation requires recompilation |
| Load time | Relocatable code fixed when loaded |
| Execution time | Hardware translates while running; process can move/remap dynamically |

Execution-time binding enables virtual memory and strong process isolation.

## Loading and linking

- **Static linking:** library code copied into executable; larger binary, fewer runtime library dependencies.
- **Dynamic linking:** executable references shared library resolved at load/run time; saves storage/memory but adds version/runtime dependency concerns.
- **Static loading:** whole program loaded before execution.
- **Dynamic loading:** routines loaded when first needed.

Dynamic linking and dynamic loading are not the same concept.

## Contiguous allocation

Each process occupies one continuous physical region.

- Base register identifies start.
- Limit register bounds valid offsets.
- Simple and fast, but placement and fragmentation become difficult.

### Placement policies

| Policy | Choice |
|---|---|
| First fit | First sufficiently large hole |
| Best fit | Smallest sufficient hole |
| Worst fit | Largest available hole |

No policy universally eliminates fragmentation.

## Fragmentation

| Internal fragmentation | External fragmentation |
|---|---|
| Wasted space inside allocated unit | Free space split into unusable holes |
| Fixed-size allocation commonly causes it | Variable-size contiguous allocation commonly causes it |
| Compaction does not recover it | Compaction may recover it if relocation is possible |

Paging mainly removes external fragmentation for physical allocation but still has internal waste, especially in a process's last page.

## Segmentation

Address is `(segment number, offset)`. Each segment has independent base, limit and permissions.

Benefits:

- Matches logical regions such as code, stack and data.
- Natural sharing/protection per region.

Costs:

- Variable-size segments create external fragmentation.
- Allocation and compaction are complex.

Some systems combine segmentation concepts with paging; modern flat address spaces often rely primarily on paging.

## Process memory regions

- **Stack:** call frames, parameters, local automatic variables and return information.
- **Heap:** dynamic lifetime allocations managed by an allocator.
- **Mapped regions:** shared libraries, file mappings and anonymous mappings.
- Stack/heap are virtual regions; they are not required to be physically contiguous.

## Allocation APIs and allocator behavior

- `malloc` obtains a block of at least requested size; contents are uninitialized.
- `calloc` allocates an array-sized block and initializes bytes to zero.
- `realloc` may resize in place or move/copy the block.
- `free` returns a block to the allocator, not necessarily immediately to the OS.

Allocators request larger regions from the OS and suballocate them. Consequently, every allocation does not require a system call, and freed process memory may remain in allocator arenas for reuse.

## Common memory bugs

| Bug | Meaning |
|---|---|
| Leak | Allocated memory becomes unreleased/unreachable |
| Dangling pointer | Pointer refers to an object whose lifetime ended |
| Use-after-free | Dereferences freed storage |
| Double free | Releases the same allocation twice |
| Buffer overflow | Accesses beyond object bounds |
| Stack overflow | Stack exceeds its available mapped/guarded region |

## Kernel allocator overview

- **Buddy allocator:** manages physical page blocks in power-of-two sizes; coalesces buddies.
- **Slab-style allocator:** caches frequently used kernel objects to reduce initialization and fragmentation overhead.

These are implementation concepts; do not confuse them with user-space `malloc` interfaces.

## Common traps

- Virtual contiguity does not require physical contiguity.
- `free()` does not guarantee immediate reduction in process RSS.
- Zero-initialized `calloc` memory is not the same as “no physical page allocated” in every situation.
- Heap and stack direction are conventions, not universal laws.
- Paging and segmentation solve different abstraction/allocation problems.
- A memory leak does not mean bytes disappeared from physical RAM permanently.

## Interview checks

1. Logical/virtual versus physical address?
2. Internal versus external fragmentation?
3. Why can compaction address external but not internal fragmentation?
4. Static linking versus dynamic linking?
5. Dynamic linking versus dynamic loading?
6. Why does `malloc()` not call the kernel for every allocation?
7. Why may RSS remain high after `free()`?
8. Stack overflow versus heap exhaustion?
9. Segmentation versus paging at a conceptual level?

## 60-second recall

- MMU translates per-process virtual addresses to protected physical memory.
- Binding can occur at compile, load or execution time.
- Fixed-size allocation creates internal waste; variable contiguous regions create external holes.
- Segmentation matches logical regions but fragments externally.
- User allocators suballocate OS-provided regions, so `malloc/free` are not one-to-one system calls.

## References

- [OSTEP: Address Spaces and Memory API](https://pages.cs.wisc.edu/~remzi/OSTEP/)

