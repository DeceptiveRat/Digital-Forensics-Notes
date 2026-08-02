## extents
- resident data is found in inode data fork:
	- very small content is saved in data fork
- similar to runlists for NTFS
- records:
	- number of blocks in extent
	- absolute block address of starting block in extent
	- logical file block offset represented by extent
	- flag specifying whether extent was pre-allocated
- 16 bytes in size
- structure:
![[XFS extent structure.png]]
	- 21 LSB: number of blocks in current extent
	- subsequent 52 bits: starting block in extent
	- subsequent 54 bits: logical block address inside file
	- single MSB: pre-allocation flag bit
- stored in extent list format
- list is stored in inode data fork
- in XFS V5 21+ extents are stored in B+tree instead of list

## reference
[1] (Fergus Toolan, 2025/02/07) File System Forensics, 2026/08/01, -