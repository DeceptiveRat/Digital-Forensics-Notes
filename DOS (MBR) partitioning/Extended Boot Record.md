## *Extended Boot Record* (EBR)
- aka secondary extended partition
- each contain a partition table
- first EBR is located on first sector of extended partition(primary extended partition)
- multiple EBRs form linked list
- contains a logical partition(secondary file system partition)

## EBR structure
- same structure as MBR
- only first 2 partitions in partition table are used
- magic number 0xAA55 at end of sector

## EBR partition table values
- first entry:
	- starting sector = relative offset between this EBR sector and first sector of logical partition
	- number of sectors = total count of sectors for this logical partition
	![[EBR partition table first entry.png]]
	
- second entry:
	- partition type = 0x05 for CHS, 0x0F for LBA addressing
	- starting sector = relative address of next EBR within extended partition
	- number of sectors = total count of sectors for next logical partition, count starting from next EBR sector
	![[EBR partition table second entry.png]]
	
reference:
- [Wikipedia] "Extended boot record" - https://en.wikipedia.org/w/index.php?title=Extended_boot_record&oldid=1223591571, Jan 18 2026
- [Brian Carrier, 2005] "File System Forensic Analysis"