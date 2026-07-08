# File Systems & Disk Scheduling

> **Tag: Theory + one numerical (disk scheduling trace)** — lighter-weight topic; know inodes, allocation methods, and the SCAN family.

## File Systems

**What the FS does:** maps names → data blocks, tracks free space, enforces permissions, survives crashes.

- **Inode (Unix):** per-file metadata block — size, permissions, timestamps, and pointers to data blocks (direct pointers + single/double/triple indirect for big files). **Filename lives in the directory entry, not the inode** — that's why hard links work (two names, one inode, link count).
- **Directory:** just a special file of (name → inode#) mappings.
- **Hard link vs symlink:** hard = another name for same inode (same file, survives original's deletion, same FS only). Symlink = a file containing a path (can dangle, cross filesystems).
- **Allocation methods:** contiguous (fast reads, external fragmentation — used for CDs), linked list/FAT (no external frag, terrible random access), **indexed/inode** (random access + no external frag — the winner).
- **Free space:** bitmaps or free lists.
- **Journaling (ext4, NTFS):** write intended changes to a log *first*, then apply; crash mid-operation → replay/discard the journal → no inconsistent metadata, no full-disk fsck. (Same WAL idea as databases — say this, it connects subjects.)
- **Page cache / buffer cache:** OS caches file blocks in RAM; writes are flushed lazily (`fsync` forces durability — this is what databases call on commit; ties ACID durability to the OS).

## Disk Scheduling (HDD)

Seek time dominates HDD access (~ms). Reorder pending requests to minimize head movement:

| Algorithm | Idea | Note |
|---|---|---|
| FCFS | In arrival order | Fair, awful movement |
| SSTF | Nearest request first | Good, starves far requests |
| **SCAN** (elevator) | Sweep to one end, reverse | Fairer; ends visited even if empty |
| **LOOK** | SCAN but reverse at last request | Practical SCAN |
| C-SCAN / C-LOOK | Sweep one direction only, jump back | Uniform wait times |

**Worked trace:** head at 50, queue [82, 170, 43, 140, 24, 16, 190], disk 0–199.
- **SSTF:** 50→43→24→16→82→140→170→190 = 7+19+8+66+58+30+20 = **208 cylinders**
- **SCAN (moving toward 0):** 50→43→24→16→**0**→82→140→170→190 = 50 + 190 = **240**
- **LOOK (toward 0):** 50→…→16→82→…→190 = 34 + 174 = **208** (no pointless trip to 0)

Method: sort requests, walk the head, sum |moves|.

**SSD note (modern reality check):** no moving head → scheduling algorithms above are irrelevant; SSD concerns are wear leveling, TRIM, write amplification, parallel channels. Saying this earns points.

## RAID (one-liner each)

RAID 0 striping (speed, no redundancy) · RAID 1 mirroring (redundancy, 2× cost) · RAID 5 striping + distributed parity (1-disk tolerance, cheap) · RAID 10 mirror+stripe (fast + safe, favorite for DBs).

## Most-Asked Interview Questions

1. **What is an inode? What's NOT in it?** Metadata + block pointers; the *name* is not (it's in the directory) — hard-link follow-up depends on this.
2. **Hard link vs soft link?** → above; test: delete original — hard link still works, symlink dangles.
3. **Trace SSTF/SCAN/LOOK for this queue.** → method above.
4. **What is journaling and why?** Metadata WAL → crash consistency without full scans; connect to DB WAL/ACID durability.
5. **What happens when you `rm` a file?** Directory entry removed, inode link count −−; data blocks freed only when count = 0 AND no process holds it open (why logs can fill disk after deletion).
6. **fsync / why do databases care?** OS buffers writes; fsync forces to stable storage; durability (the D in ACID) is literally this syscall.
7. **Why is sequential I/O so much faster than random on HDD?** No seeks; ~100 MB/s vs ~1 MB/s at 10ms/seek — the number that motivates LSM trees and WALs (cross-ref System Design indexing).
