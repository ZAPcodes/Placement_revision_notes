# 11. File Systems

> A filesystem maps human-visible names and file operations onto persistent blocks while preserving metadata and crash consistency.

## File abstraction

A file typically has:

- Data/content
- Type
- Size
- Owner/group
- Permissions
- Timestamps
- Link/reference metadata
- Internal identifier such as an inode number

Common operations: create, open, close, read, write, seek, truncate, rename, link, unlink and query metadata.

## Path resolution

```mermaid
flowchart LR
    P["Path"] --> D["Directory entries"]
    D --> I["File metadata / inode"]
    I --> B["Data blocks or extents"]
```

- **Absolute path:** starts from filesystem root.
- **Relative path:** starts from process's current working directory.
- Directories map names to internal file identifiers.
- Mounting attaches a filesystem at a directory in the namespace.

## Descriptor, open-file state and inode

| Object | Scope | Stores |
|---|---|---|
| File descriptor | Per process | Small integer indexing an open reference |
| Open-file description/table entry | Kernel-wide/shared | Current offset, access mode/status flags, reference count |
| Inode/file metadata object | Filesystem/kernel cache | Owner, permissions, size, block mapping, timestamps |
| Directory entry | Directory namespace | Name -> file identifier mapping |

Multiple descriptors can refer to the same open-file description, and multiple open-file descriptions can refer to the same inode.

After `fork()`, inherited descriptors generally reference the same underlying open-file description, so they can share an offset.

## Hard links vs symbolic links

| Hard link | Symbolic link |
|---|---|
| Another directory entry for the same inode/file | Separate file containing a target path |
| Survives deletion/rename of another name | Can become dangling |
| Normally cannot cross filesystems | Can cross filesystems |
| Usually restricted for directories | Can refer to directories |
| Indistinguishable from other hard-linked names | Has its own metadata and identity |

Unlinking removes a name. File storage is reclaimed only when no directory links and no open references remain, under Unix-like semantics.

## Access methods

- **Sequential:** process bytes/records in order.
- **Random/direct:** seek to arbitrary offset.
- **Indexed:** use a higher-level index to locate records.

The filesystem supplies byte/block access; database indexes are separate structures built for query semantics.

## Allocation strategies

| Strategy | Strength | Weakness |
|---|---|---|
| Contiguous | Fast sequential/random access | Growth and external fragmentation |
| Linked | Easy growth; no external holes | Poor random access; pointer overhead/reliability |
| Indexed | Direct access through index block(s) | Metadata/index overhead |
| Extents | Store ranges of contiguous blocks | Extent management/fragmentation over time |

Modern filesystems commonly use extents and tree structures rather than textbook-pure schemes.

## Free-space management

- Bitmap/bit vector
- Free list
- Grouping/counting ranges
- Space trees and allocation groups in advanced filesystems

Allocation aims to reduce fragmentation and keep related data near each other.

## Caching and buffering

- **Page cache:** keeps file data pages in memory.
- **Buffering:** absorbs differences in transfer size/timing.
- **Read-ahead:** predicts sequential access.
- **Write-back:** marks pages dirty and persists them later.

A successful user-space write may have copied data only into kernel cache. Durability may require an explicit synchronization operation and correct storage/filesystem guarantees.

## Virtual File System (VFS)

VFS provides a common kernel interface for different filesystem implementations. Generic operations dispatch to filesystem-specific code. It separates pathname/file APIs from on-disk format details.

## Crash consistency

A logical operation may require several physical updates: data block, inode, allocation bitmap and directory entry. A crash between writes can leave inconsistent metadata.

### `fsck`

- Scans filesystem structures after failure.
- Repairs inconsistencies using structural rules.
- Can be slow because it may examine large parts of the filesystem.

### Journaling / write-ahead logging

- Record intended metadata/data updates in a log before applying them to home locations.
- Commit record identifies a complete transaction.
- Recovery replays committed operations and ignores incomplete ones.
- Metadata-only journaling protects structural consistency, not necessarily latest user data.

### Log-structured filesystem

Treats storage as an append-oriented log and cleans old segments later. It differs from merely adding a small recovery journal to a conventional filesystem.

## Atomicity and ordering caveats

- Rename is often designed as an atomic namespace operation within one filesystem, but exact guarantees matter.
- Atomic metadata operation does not imply data contents are durable.
- Application crash consistency requires write ordering, synchronization and often a replace-by-rename protocol.
- Storage caches and controllers also affect durability.

## Common traps

- File descriptor is not an inode.
- Filename is stored in a directory entry, not typically in the inode itself.
- Deleting an open file may remove its name while allowing existing descriptors to continue using it.
- Hard links share the same file identity; symlinks store paths.
- Page cache and disk contents can differ after a successful `write()`.
- Journaling is not a backup and may journal only metadata.
- File size and allocated disk blocks can differ for sparse files.

## Interview checks

1. Explain descriptor -> open-file entry -> inode -> blocks.
2. What happens when an open file is deleted?
3. Hard link versus symbolic link?
4. Why can parent and child share a file offset after `fork()`?
5. What does a directory store?
6. Why is journaling needed?
7. Journaling versus log-structured filesystem?
8. Does successful `write()` guarantee durability?
9. Why may a sparse file's logical size exceed allocated storage?
10. How could you atomically replace a configuration file conceptually?

## Resume connection: append-only recovery

Prepare precise answers for:

- Record framing and partial final records
- Checksums/versioning
- Flush and durability policy
- Replay idempotency
- Corruption handling
- Log compaction/snapshotting
- Whether acknowledged SET operations survive power loss or only process restart

## 60-second recall

- Directory maps name to file identity; inode stores metadata/block mapping.
- Descriptor indexes per-process state; open-file description holds offset/flags.
- Unlink removes a name; storage survives while links/open references remain.
- Writes often land in page cache before durable storage.
- Journaling protects multi-block update consistency through write-ahead recovery.

## References

- [OSTEP: Files, Directories and File-System Implementation](https://pages.cs.wisc.edu/~remzi/OSTEP/)
- [MIT 6.1810: File Systems and Crash Recovery](https://pdos.csail.mit.edu/6.S081/2026/schedule.html)

