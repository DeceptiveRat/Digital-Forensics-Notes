# Q. do journals not store data changes?

# A.
It’s a common point of confusion, but the short answer is: **it depends on how your file system is configured.**

By default, most modern journaling file systems (like NTFS for Windows or ext4 for Linux) only journal **metadata**. Here is the breakdown of why that is and how "actual data" fits into the picture.

---

## 1. Metadata vs. Actual Data

To understand the journal, we have to distinguish between the two types of information written to a disk:

* **Metadata:** This is the "data about the data." It includes file permissions, timestamps, file names, and—most importantly—the **pointers** (addresses) that tell the OS where the actual content is stored on the drive.
* **Actual Data (Payload):** This is the content inside your file (the text in a Word doc, the pixels in a photo).

## 2. Why focus on Metadata?

The primary goal of a journal is **file system integrity**, not necessarily "data integrity."

If the power cuts while you are saving a file:

* **Without a journal:** The file system might point to a block of data that doesn't exist yet, leading to a "corrupted file system" error that requires a lengthy `chkdsk` or `fsck` scan to fix the entire drive structure.
* **With metadata journaling:** The journal ensures the "map" of the hard drive remains consistent. You might lose the last 5 seconds of work inside the file, but the hard drive itself remains healthy and readable.

---

## 3. Does it work for "Actual Data"?

It can, but it usually doesn't for performance reasons. There are three main "modes" of journaling:

| Mode | What is Journaled? | Impact |
| --- | --- | --- |
| **Writeback** | Metadata only. | Fast, but data inside files can be corrupted. |
| **Ordered** | Metadata is journaled, but data is written to disk *first*. | **The Standard.** Prevents old data from appearing in new files. |
| **Data Journaling** | Both Metadata **and** Actual Data. | Very slow (everything is written twice), but provides maximum safety. |

> **The Verdict:** While "Data Journaling" exists, it is rarely used because it essentially doubles the work the hard drive has to do. Most systems use **Ordered Mode**, where the system ensures the data is safely on the disk before it marks the metadata change as "complete" in the journal.

---

## Why this matters for "Crash Recovery"

If your computer crashes, the OS looks at the journal. If it sees a "started" entry for a metadata change but no "finished" entry, it simply rolls back that specific change. This takes seconds, whereas checking the entire disk for errors could take hours.

Would you like me to explain how "copy-on-write" file systems like ZFS or Btrfs handle this differently without using a traditional journal?