It’s definitely one of those "wait, why would they do it like that?" moments in low-level programming. The short answer is **legacy compatibility**.

When Microsoft introduced Long File Names (LFN) in Windows 95, they had a major problem: they needed to store long names on a file system (FAT12/FAT16) that was strictly designed for "8.3" filenames (8 characters for the name, 3 for the extension).

### The "Frankenstein" Structure

To keep old computers from crashing when they saw a long name, Microsoft decided to hide the long names inside "dummy" directory entries. They used a specific combination of attributes (Read-Only, Hidden, System, and Volume Label) that would cause older DOS systems to simply ignore them.

Each of these dummy entries is 32 bytes total. After accounting for the attributes and checksums, they were left with **26 bytes** for the actual name characters. Since FAT uses **UTF-16** (where each character is 2 bytes), those 26 bytes can hold exactly **13 characters**.

### The Breakdown: 5, 6, and 2

The 13 characters are split into three non-contiguous chunks because they had to "overlay" the long name on top of the existing 8.3 entry fields without touching the bytes that identified the entry type.

Here is how a single LFN entry is mapped:

| Name Chunk | Characters | Offset in Entry | Original 8.3 Field it Overwrites |
| --- | --- | --- | --- |
| **Part 1** | 1–5 | `0x01` | Filename (part 1) |
| **Part 2** | 6–11 | `0x0E` | Creation Date/Time & Cluster info |
| **Part 3** | 12–13 | `0x1C` | Last Access Date & remaining cluster info |

### Why this specific split?

The gaps between these chunks aren't random; they are "protected" bytes that the system needs to maintain the illusion that these aren't real files:

* **Between 5 and 6:** This is where the **Attributes** byte (offset `0x0B`) sits. It must stay set to `0x0F` to tell the OS "ignore me, I’m a special LFN entry."
* **Between 11 and 12:** This is where the **First Cluster Low** (offset `0x1A`) resides. For LFN entries, this must always be `0x0000` to prevent disk repair tools from thinking these entries are taking up space on the drive.

### Summary

Every single LFN entry follows this **5-6-2** pattern. If your filename is longer than 13 characters, the system just stacks multiple 32-byte entries on top of each other until the full name is spelled out—each one repeating that same fragmented 5-6-2 layout.

---

Would you like me to show you how a specific filename (like "SuperLongFileName.txt") would be encoded into these chunks?