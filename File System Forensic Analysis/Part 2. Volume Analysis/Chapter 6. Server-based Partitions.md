## 1. BSD partition (SKIP)

## 2. Sun Solaris Slices (SKIP)

## 3. GPT partition
- *GUID partition table*
- partition system used by *Extensible Firmware Interface(EFI)*
- supports 128 partitions and uses 64-bit LBA addresses
- contains 5 major areas:
	- protective MBR:
		- in first sector of disk
		- contains DOS partition table with 1 entry with type 0xEE
		- exists so legacy computers do not format disk
	- GPT header:
		- defines size and location of partition table
		- contains checksum of header and partition table for errors and modification detection
	- partition table:
		- each entry contains start/end address, type value, name, attribute flags, and GUID value
	- partition area:
		- contains sectors to be allocated to partitions
	- backup of GPT header and partition table:
		- GPT header is in last sector, partition table before it

## 4. GPT data structures
- DOS partition table
	``` sh
	$ mmls –t dos gpt-disk.dd
	DOS Partition Table
	Units are in 512-byte sectors
	Slot Start End Length Description
	00: ----- 0000000000 0000000000 0000000001 Primary Table (#0)
	01: 00:00 0000000001 0120103199 0120103199 GPT Safety Partition (0xEE)
	```
- GPT header

	| Byte Range | Description | Essential | 
	| --- --- | --- | --- | 
	| 0–7 | Signature value("EFI PART") | No | 
	| 8–11 | Version | Yes | 
	| 12–15 | Size of GPT header in bytes | Yes | 
	| 16–19 | CRC32 checksum of GPT header | No | 
	| 20–23 | Reserved | No | 
	| 24–31 | LBA of current GPT header structure | No | 
	| 32–39 | LBA of the other GPT header structure | No | 
	| 40–47 | LBA of start of partition area | Yes | 
	| 48–55 | LBA of end of partition area | No | 
	| 56–71 | Disk GUID | No | 
	| 72–79 | LBA of the start of the partition table | Yes | 
	| 80–83 | Number of entries in partition table | Yes | 
	| 84–87 | Size of each entry in partition table | Yes | 
	| 88–91 | CRC32 checksum of partition table | No | 
	| 92–End of Sector | Reserved | No | 

	``` sh
	$ dd if=gpt-disk.dd bs=512 skip=1 count=1 | xxd
	0000000: 4546 4920 5041 5254 0000 0100 5c00 0000 EFI PART....\...
	0000016: 8061 a3b0 0000 0000 0100 0000 0000 0000 .a..............
	0000032: 1fa1 2807 0000 0000 2200 0000 0000 0000 ..(.....".......
	0000048: fea0 2807 0000 0000 7e5e 4da1 1102 5049 ..(.....~^M...PI
	0000064: ab2a 79a6 3ea6 3859 0200 0000 0000 0000 .*y.>.8Y........
	0000080: 8000 0000 8000 0000 69a5 7180 0000 0000 ........i.q.....
	0000096: 0000 0000 0000 0000 0000 0000 0000 0000 ................
	[REMOVED]
	```
	- signature value in first 8 bytes
	- bytes 12~15 show size of header; 96 bytes
	- bytes 32~39 show location of backup header; 0x0728a1af
	- bytes 40~55 show start/end of partition area; 0x22, 0x0728a0fe
	- bytes 72~79 show start of partition table; 0x02
	- bytes 80~83 show number of partition table entries; 0x80
	- bytes 84~87 show size of each entry; 0x80
- partition table entry

	| Byte Range | Description | Essential |
	| --- | --- | --- |
	| 0–15 | Partition type GUID | No |
	| 16–31 | Unique partition GUID | No |
	| 32–39 | Starting LBA of partition | Yes |
	| 40–47 | Ending LBA of partition | Yes |
	| 48–55 | Partition attributes | No |
	| 56–127 | Partition name in Unicode | No |

	- 64-bit attribute field divided into 3 parts:
		- lowest bit set to 1 when system can't function without this partition
		- bits 1~47 undefined
		- bits 48~63 stores partition type specific data

	``` sh
	$ dd if=gpt-disk.dd bs=512 skip=34 | dd bs=128 skip=3 count=1 | xxd
	0000000: 16e3 c9e3 5c0b b84d 817d f92d f002 15ae ....\..M.}.-....
	0000016: 2640 69eb 2f99 1942 afc0 d673 7c0b 8ae4 &@i./..B...s|...
	0000032: 2200 0000 0000 0000 0080 3e00 0000 0000 ".........>.....
	0000048: 0000 0000 0000 0000 0000 ffff ffff ffff ................
	0000064: ffff ffff ffff ffff ffff ffff ffff ffff ................
	[REMOVED]
	```
	- bytes 32~39 show start of partition; 0x22
	- bytes 40~47 show end of partition; 0x003E8000

## 5. GPT partition types
- Intel

	| GUID Type Value | Description |
	| --- | --- |
	| 00000000-0000-0000-0000-000000000000 | Unallocated entry |
	| C12A7328-F81F-11D2-BA4B-00A0C93EC93B | EFI system partition |
	| 024DEE41-33E7-11d3-9D69-0008C781F39F | Partition with DOS partition table inside |

- Microsoft

	| GUID Type Value | Description |
	| --- | --- |
	| E3C9E316-0B5C-4DB8-817D-f92DF00215AE | Microsoft Reserved Partition(MRP) |
	| EBD0A0A2-B9E5-4433-87C0-68B6B72699C7 | Primary partition(basic disk) |
	| 5808C8AA-7E8F-42E0-85D2-E1E90434CFB3 | LDM metadata partition(dynamic disk) |
	| AF9B60A0-1431-4F62-BC68-3311714A69AD | LDM data partition(dynamic disk) |
	- reserved partition stores temporary files and data
	- primary partition stores file system
	- LDM partitions used for dynamic disk; which are used to merge multiple disks into one volume

## 6. GPT partition mmls output
``` sh
$ mmls –t gpt gpt-disk.dd
GUID Partition Table
Units are in 512-byte sectors
Slot Start End Length Description
00: ----- 0000000000 0000000000 0000000001 Safety Table
01: ----- 0000000001 0000000001 0000000001 GPT Header
02: ----- 0000000002 0000000033 0000000032 Partition Table
03: 00 0000000034 0004096000 0004095967
04: 01 0004096001 0012288000 0008192000
```
