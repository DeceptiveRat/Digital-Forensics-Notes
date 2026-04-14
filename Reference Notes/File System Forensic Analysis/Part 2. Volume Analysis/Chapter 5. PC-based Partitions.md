## 1. DOS partitions
- disks with DOS partitions are called *Master Boot Record* (MBR) disks
- used with Microsoft DOS, Microsoft Windows, Linux, IA32-based FreeBSD and OpenBSD systems

## 2. MBR disk concepts
- has MBR in first sector of disk
- MBR contains:
	- boot code
	- partition table
	- signature value
- *primary file system partition*: contains file system or other structured data
- *primary extended partition*: contains additional partitions

## 3. DOS partition table
- contains 4 entries, which can describe a DOS partition each
- each entry has:
	- starting CHS address
	- ending CHS address
	- starting LBA address
	- number of sectors in partition
	- type of partition:
		- identifies type of data stored
		- Linux ignores it
		- Windows relies on it => can be used to hide partitions
	- flags:
		- identifies bootable partition

## 4. primary extended partition
- precede each parition with size of partition and location of next partition => linked list of partitions
- *secondary file system partition* or *logical partition*: partitions located within primary extended partitions, contains file system or other structured data
- *secondary extended partition*:
	- contains partition table and secondary file system partition
	- describes location of secondary file system partition and next secondary extended partition
	- should only contain 1 secondary file system partition => having more file system partitions can cause errors when analyzing
	- has special types for partition table entries
- multiple types of extended partitions exist: DOS, Windows 95, Linux etc
	
![[six file system partition layout.png]]

## 5. DOS boot code
- exists in first 446 bytes
- standard MS boot code processes partition table and executes code in first sector of identified boot partition

## 6. MBR data structure

![[DOS partition table data structure.png]]

- partition table:
	- extended partitions do not need boot code => data can be hidden

![[DOS partition table entry data structure.png]]

- partition table entry:
	- CHS addresses are not essential on newer systems
	- bootable flag is not always necessary

## 7. MBR extraction walkthrough
- system: dual boot Windows and Linux, 8 file system partitions
- read first sector
	``` bash
	$ dd if=disk3.dd bs=512 skip=0 count=1 | xxd
	0000000: eb48 9010 8ed0 bc00 b0b8 0000 8ed8 8ec0 .H..............
	[REMOVED]
	0000384: 0048 6172 6420 4469 736b 0052 6561 6400 .Hard Disk.Read.
	0000400: 2045 7272 6f72 00bb 0100 b40e cd10 ac3c Error.........<
	0000416: 0075 f4c3 0000 0000 0000 0000 0000 0000 .u..............
	0000432: 0000 0000 0000 0000 0000 0000 0000 0001 ................
	0000448: 0100 07fe 3f7f 3f00 0000 4160 1f00 8000 ....?.?...A`....
	0000464: 0180 83fe 3f8c 8060 1f00 cd2f 0300 0000 ....?..`.../....
	0000480: 018d 83fe 3fcc 4d90 2200 40b0 0f00 0000 ....?.M.".@.....
	0000496: 01cd 05fe ffff 8d40 3200 79eb 9604 55aa .......@2.y...U.
	```
	- 0XAA55 signature found at end of sector
	- partition table starts at offset 446 with 0x0001

- view partition table

	| # | Flag | Type | Starting Sector | Size |
	| --- | --- | --- | --- | --- |
	| 1 | 0x00 | 0x07 | 0x0000003f(63) | 0x001f6041(2,056,257) |
	| 2 | 0x80 | 0x83 | 0x001f6080(2,056,320) | 0x00032fcd(208,845) |
	| 3 | 0x00 | 0x83 | 0x0022904d(2,265,165) | 0x000fb040(1,028,160) |
	| 4 | 0x00 | 0x05 | 0x0032408d(3,293,325) | 0x0496eb79(76,999,545) |

	- partitions are NTFS, Linux, Linux, and extended partition
	- second partition is bootable

## 8. extended partition data structure
- starting address of secondary file system partition is relative to current partition table
- starting address of secondary extended partition is relative to primary extended partition

![[extended partition table entry.png]]

## 9. extended partition extraction walkthrough
- read first sector of primary extended partition
	``` sh
	$ dd if=disk3.dd bs=512 skip=3293325 count=1 | xxd
	[REMOVED]
	0000432: 0000 0000 0000 0000 0000 0000 0000 0001 ................
	0000448: 01cd 83fe 7fcb 3f00 0000 0082 3e00 0000 ......?.....>...
	0000464: 41cc 05fe bf0b 3f82 3e00 40b0 0f00 0000 A.....?.>.@.....
	0000480: 0000 0000 0000 0000 0000 0000 0000 0000 ................
	0000496: 0000 0000 0000 0000 0000 0000 0000 55aa ..............U.
	```
- parsed partition table

	| # | Flag | Type | Starting Sector | Size |
	| --- | --- | --- | --- | --- |
	| 5 | 0x00 | 0x83 | 0x0000003f(63) | 0x003e8200(4,096,572) |
	| 6 | 0x00 | 0x05 | 0x003e823f(4,096,575) | 0x000fb040(1,028,160) |

	- entry 5 is a secondary file system partition starting at 3,293,388
	- entry 6 is DOS extended partition starting at 7,389,900

## 10. partition structure analysis tool
- fdisk output
	``` sh
	$ fdisk –lu disk3.dd
	Disk disk3.dd: 255 heads, 63 sectors, 0 cylinders
	Units = sectors of 1 * 512 bytes
	Device Boot Start End Blocks Id System
	disk3.dd1 63 2056319 1028128+ 7 HPFS/NTFS
	disk3.dd2 * 2056320 2265164 104422+ 83 Linux
	disk3.dd3 2265165 3293324 514080 83 Linux
	disk3.dd4 3293325 80292869 38499772+ 5 Extended
	disk3.dd5 3293388 7389899 2048256 83 Linux
	disk3.dd6 7389963 8418059 514048+ 82 Linux swap
	disk3.dd7 8418123 9446219 514048+ 83 Linux
	disk3.dd8 9446283 17639369 4096543+ 7 HPFS/NTFS
	disk3.dd9 17639433 48371714 15366141 83 Linux
	```
	- secondary extended partitions are not displayed
- mmls output
	``` sh
	$ mmls –t dos disk3.dd
	Units are in 512-byte sectors
	Slot Start End Length Description
	00: ----- 0000000000 0000000000 0000000001 Table #0
	01: ----- 0000000001 0000000062 0000000062 Unallocated
	02: 00:00 0000000063 0002056319 0002056257 NTFS (0x07)
	03: 00:01 0002056320 0002265164 0000208845 Linux (0x83)
	04: 00:02 0002265165 0003293324 0001028160 Linux (0x83)
	05: 00:03 0003293325 0080292869 0076999545 DOS Extended (0x05)
	06: ----- 0003293325 0003293325 0000000001 Table #1
	07: ----- 0003293326 0003293387 0000000062 Unallocated
	08: 01:00 0003293388 0007389899 0004096512 Linux (0x83)
	09: 01:01 0007389900 0008418059 0001028160 DOS Extended (0x05)
	10: ----- 0007389900 0007389900 0000000001 Table #2
	11: ----- 0007389901 0007389962 0000000062 Unallocated
	12: 02:00 0007389963 0008418059 0001028097 Linux Swap (0x82)
	13: 02:01 0008418060 0009446219 0001028160 DOS Extended (0x05)
	14: ----- 0008418060 0008418060 0000000001 Table #3
	15: ----- 0008418061 0008418122 0000000062 Unallocated
	16: 03:00 0008418123 0009446219 0001028097 Linux (0x83)
	17: 03:01 0009446220 0017639369 0008193150 DOS Extended (0x05)
	18: ----- 0009446220 0009446220 0000000001 Table #4
	19: ----- 0009446221 0009446282 0000000062 Unallocated
	20: 04:00 0009446283 0017639369 0008193087 NTFS (0x07)
	21: 04:01 0017639370 0048371714 0030732345 DOS Extended (0x05)
	22: ----- 0017639370 0017639370 0000000001 Table #5
	23: ----- 0017639371 0017639432 0000000062 Unallocated
	24: 05:00 0017639433 0048371714 0030732282 Linux (0x83)
	```
	- output is sorted by starting sector of partition
	- second column shows partition table and entry number

## 11. DOS analysis considerations
- 63 sectors are allocated for MBR and extended partitions => data can be hidden
- extended partitions should only have 2 entries
- partition types are not always enforced
- some versions of Windows do not create 3 primary partitions before extended partitions
- searching for 0xAA55 in the last 2 bytes of a sector can be used to find partition table
	- same signature exists in first sector of NTFS and FAT file systems => examine to determine
	- if it is boot sector of file system, partition table may be found 63 sectors prior

## 12. Apple partitions
- can describe any number of partitions
- data structures are in consecutive sectors of the disk
- Mac OS X is based on BSD kernel but uses Apple partition map

## 13. partition map
- describes partitions
- does not contain bootcode
- each entry describes:
	- starting sector of partition
	- size
	- type
	- volume name
- first entry is entry for itself; shows max size of partition map
- many partitions contain non-file system content such as hardware drivers

## 14. partition map data structure
- Apple partition entry data structure

	| Byte Range | Description | Essential |
	| --- | --- | --- |
	| 0–1 | Signature value(0x504D) | No |
	| 2–3 | Reserved | No |
	| 4–7 | Total Number of partitions | Yes |
	| 8–11 | Starting sector of partition | Yes |
	| 12–15 | Size of partition in sectors | Yes |
	| 16–47 | Name of partition in ASCII | No |
	| 48–79 | Type of partition in ASCII | No |
	| 80–83 | Starting sector of data area in partition | No |
	| 84–87 | Size of data area in sectors | No |
	| 88–91 | Status of partition | No |
	| 92–95 | Starting sector of boot code | No |
	| 96–99 | Size of boot code in sectors | No |
	| 100–103 | Address of boot loader code | No |
	| 104–107 | Reserved | No |
	| 108–111 | Boot code entry point | No |
	| 112–115 | Reserved | No |
	| 116–119 | Boot code checksum | No |
	| 120–135 | Processor type | No |
	| 136–511 | Reserved | No |
	- data area field used for file systems with data areas that don't start at beginning of disk

- status values

	| Type | Description |
	| --- | --- |
	| 0x00000001 | Entry is valid(A/UX only) |
	| 0x00000002 | Entry is allocated(A/UX only) |
	| 0x00000004 | Entry in use(A/UX only) |
	| 0x00000008 | Entry contains boot information(A/UX only) |
	| 0x00000010 | Partition is readable(A/UX only) |
	| 0x00000020 | Partition is writable(Macintosh & A/UX) |
	| 0x00000040 | Boot code is position independent(A/UX only) |
	| 0x00000100 | Partition contains chain-compatible driver(Macintosh only) |
	| 0x00000200 | Partition contains a real driver(Macintosh only) |
	| 0x00000400 | Partition contains a chain driver(Macintosh only) |
	| 0x40000000 | Automatically mount at startup(Macintosh only) |
	| 0x80000000 | The startup partition(Macintosh only) |

## 15. partition map parsing
- first entry
	``` sh
	$ dd if=mac-disk.dd bs=512 skip=1 | xxd
	0000000: 504d 0000 0000 000a 0000 0001 0000 003f PM.............?
	0000016: 4170 706c 6500 0000 0000 0000 0000 0000 Apple...........
	0000032: 0000 0000 0000 0000 0000 0000 0000 0000 ................
	0000048: 4170 706c 655f 7061 7274 6974 696f 6e5f Apple_partition_
	0000064: 6d61 7000 0000 0000 0000 0000 0000 0000 map.............
	0000080: 0000 0000 0000 003f 0000 0000 0000 0000 .......?........
	0000096: 0000 0000 0000 0000 0000 0000 0000 0000 ................
	[removed]
	```
	- data is stored in big-endian
	- signature value 0x504d in bytes 0~1
	- number of partitions in bytes 4~7; 10
	- bytes 8~11 show first sector of disk is starting sector of this partition
	- bytes 12~15 show size of partition is 63 sectors
	- name and type of partition can be seen; Apple, Apple_partition_map
- mmls on 20GB iBook laptop
	``` sh
	$ mmls -t mac mac-disk.dd
	MAC Partition Map
	Units are in 512-byte sectors
	Slot Start End Length Description
	00: ----- 0000000000 0000000000 0000000001 Unallocated
	01: 00 0000000001 0000000063 0000000063 Apple_partition_map
	02: ----- 0000000001 0000000010 0000000010 Table
	03: ----- 0000000011 0000000063 0000000053 Unallocated
	04: 01 0000000064 0000000117 0000000054 Apple_Driver43
	05: 02 0000000118 0000000191 0000000074 Apple_Driver43
	06: 03 0000000192 0000000245 0000000054 Apple_Driver_ATA
	07: 04 0000000246 0000000319 0000000074 Apple_Driver_ATA
	08: 05 0000000320 0000000519 0000000200 Apple_FWDriver
	09: 06 0000000520 0000001031 0000000512 Apple_Driver_IOKit
	10: 07 0000001032 0000001543 0000000512 Apple_Patches
	11: 08 0000001544 0039070059 0039068516 Apple_HFS
	12: 09 0039070060 0039070079 0000000020 Apple_Free
	```
	- entry 12 shows unallocated sector
	- entries 0, 2, and 3 were added by mmls to show what space partition map is using

## 16. Apple disk image files
- archive that can save several individual files
- can contain partition map
- can contain single partition with file system or file system with no partitions
- layout example
	``` sh
	$ mmls -t mac test.dmg
	MAC Partition Map
	Units are in 512-byte sectors
	Slot Start End Length Description
	00: ----- 0000000000 0000000000 0000000001 Unallocated
	01: 00 0000000001 0000000063 0000000063 Apple_partition_map
	02: ----- 0000000001 0000000003 0000000003 Table
	03: ----- 0000000004 0000000063 0000000060 Unallocated
	04: 01 0000000064 0000020467 0000020404 Apple_HFS
	05: 02 0000020468 0000020479 0000000012 Apple_Free
	```

## 17. Apple analysis considerations
- there are several unused fields in data structures
- data can be hidden between last entry and end of space allocated to partition map

## 18. removable media
- each floppy disk formatted in FAT12 is treated as a single partition
- partition table on a ZIP disk will be Apple partition or DOS partition
- flash cards typically have a FAT file system
- CD-ROMS have multiple variations:
	- ISO 9660 format
	- Joliet format
	- Apple HFS+ format
- CD-Rs should be examined with specialized analysis tools
