## 1. *File Allocation Table(FAT)* file system
- supported by all Windows and most Unix OS
- frequently found in compact flash cards and USB drives
- versions include FAT12, FAT16, and FAT32
- each file/directory is allocated a directory entry that contains:
	- file name
	- size
	- starting address of content
- file/directory contents are stored in data units called clusters
- the FAT is used to identify:
	- the next cluster in a file
	- allocation status of clusters
	
![[directory entry FAT and cluster relationship.png]]

## 2. FAT layout
- first section is reserved area:
	- includes data in file system category
	- typically 1 sector in size, but defined in boot sector
- second section is FAT area:
	- contains primary and backup FAT structures
	- size calculated based on number and size of FAT structure
- third section is data area:
	- contains clusters to be allocated
	- beginning of area reserved for root directory for FAT12/16; size given in boot sector
	- root directory can be anywhere for FAT32 with location given in boot sector; size and location is flexible

## 3. FAT file system category

## 4. FAT boot sector
- located in first sector of volume
- contains data that belongs to all categories
- FAT32 boot sector also contains additional data:
	- sector address of backup copy of boot sector; should be sector 6
	- major/minor version number
	- FSINFO data structure that contains info about next available cluster and number of free clusters; not guaranteed to be accurate
- essential data:
	- contains size of reserved area; used to find start of FAT area
	- contains number and size of FAT structures; used to find start of data area
	- contains total number of sectors in file system; used to find end of data area
	- contains number of sectors per cluster
- non essential data:
	- 8 character string *OEM name*: corresponds to tool used to make file system
	- 4 byte volume serial number: used with removable media to determine when disk changed
	- 8 character string: FAT12/FAT16/FAT32/FAT
	- 11 character volume label: user can specify when creating file system

## 5. FAT boot code
- intertwined with file system DS
- 
