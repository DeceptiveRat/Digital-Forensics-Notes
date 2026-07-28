## allocation group
![[XFS allocation group.png]]

- similar to block groups in ext FS
- superblock structure:
	- located at first sector of each allocation group
- free block info:
	- 1 block
- Inode B-tree info
- internal free list:
	- reserved in case FS grows
	- similar to block group descriptor growth area in ext FS
- free space B+tree:
	- 2 trees using different keys
	- block number key: ensure content is close to other parts of the file on disk
	- block count key: store file without fragmentation
	- merely ordering of entries is different
- Inode B+tree:
	- information on currently allocated inodes in FS
	- inodes are allocated 64 inodes at a time
	- space is not reserved in advance; inodes and data share available space

## inodes
- generally 512 bytes in size
- divided into 3 areas:
	- inode core:
		- 176 bytes in size
		- certain information is written during inode allocation process; signature *IN*, version, next unlinked attribute, CRC-32C checksum, inode number, and FS UUID
	- data fork:
		- initialized to 0
		- can store data in inode or in external blocks through extents
		- occur immediately after inode core
	- attribute fork:
		- initialized to 0
		- can store data in inode or in external blocks through extents
		- found in any location in inode structure
- records similar information as to ext:
	- file type
	- permissions
	- user/group ID of file owner
	- timestamp metadata (MACB time); 4 byte Unix time value + 4 byte nanosecond component

## anti forensics & data hiding
- anti forensics categories:
	- data hiding:
		- make data diffcult to detect
		- easy to retrieve for those who know it
		- e.g. steganography
	- artifact wiping:
		- permanent removal of particular information
		- e.g. file wiping, secure deletion, disk cleaning
	- trail obfuscation:
		- disorient and divert a digital investigation
		- e.g. metadata manipulation, log file alteration, IP spoofing
	- attacks against forensic process:
		- target tools and techniques used in forensic process
		- e.g. anti-reverse engineering techniques, program packers, attacks against investigator integrity
	- indications of anti-forensics:
		- e.g. logs of actions showing use of anti-forensics, presence of anti-forensic tools

## data hiding techniques
![[data hiding methods.png]]
- 3 methods of hiding information in FS:
	- slack space:
		- file slack: large storage
	- reserved space
	- misuse of FS structures

## evaluation of data hiding techniques
- based on dimensions: 
	- capacity
	- detection rating
	- stability
- ext4 timestamp data hiding:
	- indistinguishability: entropy of nanosecond time components
	- correctness: hidden data is stable
	- robustness: data is not altered
	- available capacity

## data hiding method 1 - superblock slackspace
- superblock structure uses 272 bytes of first sector; leaves 240 bytes for data hiding
- checksum value must be updated to hide data; ensure FS is useable after attack
- capacity depends on number of allocation groups:
	- determined by size of allocation group defined by *mkfs.xfs*
	- generally 4 allocation groups; 960 bytes of capacity

## data hiding method 2 - inode slack space
- capacity of inode slack space:
$512 - (176 + (16 \times x))$
*x* is the number of data forks present in node
- high capability

## data hiding method 3 - free list area
- capacity is 4 blocks per allocation group
- with 4 allocation groups, $4096 \times 4 \times 4 = 65636$ bytes available

## data hiding method 4 - free inodes
- there can exist allocated, but unused inodes 
- initialization involves:
	- setting certain values in inode core 
	- remaining space in inode is left blank
- examining allocation map structure allows finding of unused inodes
- update alloation map to mark inodes with hidden data as used
- using entire inode structure (512 bytes) is easily detectable; only slack space is used
- 63 out of 64 allocated inodes allows for $336 \times 63 = 21168$ bytes of storage
- can be manually extended through creating benign files to allocate new inode chunks; increases risk of detection

## data hiding method 5 - nanosecond timestamps
- nanosecond timestamp can be overwritten to hide information
- allows 16 bytes per inode for storage space
- only 3 bytes of each timestamp should be used to hide data to avoid detection

## evaluation
- check for:
	- stability: hidden data still present
	- detection difficulty: no information has been leaked
- steps taken:
	1. mount fs, check for errors
	2. read contents of files in FS
	3. modify existing file in FS
	4. write new file to FS
	5. delete file from FS
	6. fill FS with files
- tests performed:
	- user level:
		- mount/unmount: outcome checked through dmesg
	- system administrator level:
		- *xfs_repair* used after each action to detect errors
	- forensic analyst level:
		- TSK fork providing XFS support; *fsstat*, *fls*, *istat*, *icat* to determine what can be found

![[evaluation summary.png]]
- test results:
	- inode slack space techniques had best results; only overwritten if a contiguous file becomes fragmented or extended attributes are added to a file

![[evaluation summary 2.png]]
