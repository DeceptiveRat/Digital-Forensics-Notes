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
	``` plaintext
	# Flag Type Starting Sector Size
	1 0x00 0x07 0x0000003f (63) 0x001f6041 (2,056,257)
	2 0x80 0x83 0x001f6080 (2,056,320) 0x00032fcd (208,845)
	3 0x00 0x83 0x0022904d (2,265,165) 0x000fb040 (1,028,160)
	4 0x00 0x05 0x0032408d (3,293,325) 0x0496eb79 (76,999,545)
	```
	- partitions are NTFS, Linux, Linux, and extended partition
	- second partition is bootable

## 8. extended partition data structure
- starting address of secondary file system partition is relative to current partition table
- starting address of secondary extended partition is relative to primary extended partition
