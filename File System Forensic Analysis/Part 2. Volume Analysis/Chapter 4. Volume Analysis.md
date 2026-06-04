## 1. volume
- collection of addresssable sectors used for data storage
- concepts:
	- assemble multiple storage volumes into single volume
	- divide volume into multiple partitions

## 2. partition
- collection of consecutive sectors in volume
- used in:
	- some file systems have a max size smaller than hard disks
	- some laptops use special partition to store memory contents when asleep
	- UNIX systems use different partitions for different directories
	- IA-32 systems may require separate partitions for each OS

## 3. partition systems
- has tables that describe a partition

![[partition tables.png]]

- organizes layout of volume

## 4. volumes in UNIX
- subdirectories can be mounting points for new file systems and volumes
- partitions each disk into several volumes

## 5. analysis techniques
- locate partition tables to identify layout
- locate data structures that define which volumes are merged and how
- data not part of partition or merging process still may contain important information

## 6. extraction walkthrough
- view partition table
``` bash
$ mmls -t dos disk1.dd
```

![[mmls output.png]]

![[example partition layout.png]]

- extract partitions
	``` bash
	$ dd if=disk1.dd of=part1.dd bs=512 skip=63 count=1028097
	$ dd if=disk1.dd of=part2.dd bs=512 skip=2570400 count=1638630
	$ dd if=disk1.dd of=part3.dd bs=512 skip=4209030 count=2056320
	$ dd if=disk1.dd of=unalloc1.dd bs=512 skip=1028160 count=1542240
	```

## 7. partition recovery
- tools such as *gparted* can be used when partition structure is corrupt
	``` bash
	$ gpart -v disk2.dd
	* Warning: strange partition table magic 0x0000.
	[REMOVED]
	Begin scan...
	Possible partition(DOS FAT), size(800mb), offset(0mb)
		type: 006(0x06)(Primary 'big' DOS (> 32MB))
		size: 800mb #s(1638566) s(63-1638628)
		chs: (0/1/1)-(101/254/62)d (0/1/1)-(101/254/62)r
		hex: 00 01 01 00 06 FE 3E 65 3F 00 00 00 A6 00 19 00

	Possible partition(DOS FAT), size(917mb), offset(800mb)
		type: 006(0x06)(Primary 'big' DOS (> 32MB))
		size: 917mb #s(1879604) s(1638630-3518233)
		chs: (102/0/1)-(218/254/62)d (102/0/1)-(218/254/62)r
		hex: 00 00 01 66 06 FE 3E DA E6 00 19 00 34 AE 1C 00
		
	Possible partition(Linux ext2), size(502mb), offset(1874mb)
		type: 131(0x83)(Linux ext2 filesystem)
		size: 502mb #s(1028160) s(3839535-4867694)
		chs: (239/0/1)-(302/254/63)d (239/0/1)-(302/254/63)r
		hex: 00 00 01 EF 83 FE 7F 2E 2F 96 3A 00 40 B0 0F 00
	```
