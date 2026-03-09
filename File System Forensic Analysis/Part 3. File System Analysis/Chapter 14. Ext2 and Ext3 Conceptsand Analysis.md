## 1. ExtX FS
- Ext2 and Ext3 
- default FS for many Linux distributions
- based on *UNIX File System(UFS)*
- copies of important DS are stored throughout FS 
- all data associated with a file are localized
- layout of FS starts with optional reserved area; remainder of FS divided into sections, called *block groups*
- basic layout information stored in superblock DS, at beginning of FS

## 2. blocks and block groups
- file content is stored in *blocks*, groups of consecutive sectors
- all block groups except the last contain the same number of blocks
- blocks are used to store file names, metadata, and file content

![[block group layout example.png]]
	- block bitmap:
		- manages allocation status of blocks in group
		- starting block address given in group descriptor
		- \# of blocks per group = \# of bits per block; block bitmap requires exactly 1 block
	- inode bitmap:
		- manages allocation status of inodes in group
		- starting block address given in group descriptor
		- size in bytes = \# of inodes per group / 8
	- inode table:
		- starting block address given in group descriptor
		- size = \# inodes per group * size of each inode(128)


## 3. inode
- metadata for each file/directory is stored in DS called *inode*
- inodes have a fixed size and are located in an inode table
- there is one inode table in each block group

## 4. directory entry DS
- simple DS that contain name of file and pointer to file's inode entry
- located in blocks allocated to the file's parent directory

![[directory entry inode block relationship.png]]

## 5. feature categories
- ExtX has optional features organized into categories based on what OS should do with FS with a feature it doesn't support
- *compatible features*:
	- can still mount
	- examples:
		- allocation methods, existence of journal, etc
- *incompatible features*: 
	- OS should not mount FS
	- examples:
		- compression
- *read only compatible features*:
	- FS should be mounted as read only
	- examples:
		- large file support
		- B-trees to sort directories

## 6. FS category
- there are 2 DS that store data in FS category:
	- *superblock*
	- *group descriptor*

## 7. superblock
- located 1,024 bytes from start of FS
- 1,024 bytes in size; most aren't used
- similar to boot sector DS in NTFS/FAT
- backup copies typically stored in first block of each block group, unless sparse superblock feature is enabled
- contains basic size and config info:
	- block size
	- total \# of blocks
	- \# of blocks per block group
	- \# of reserved blocks before first block group
	- total number of inodes
	- \# of inodes per block group
- non-essential data:
	- volume name
	- last write time
	- last mount time
	- path where FS was last mounted
	- value to identify if FS is clean
	- total \# of free inodes/blocks
- defines what features FS has enabled
- with Linux, volume label can be used to identify FS

![[ExtX layout example.png]]

## 8. group descriptor table
- contains group descriptors for every block group in FS
- group descriptor describes layout of block group
- each group descriptor contains:
	- starting block address of block bitmap
	- starting block address of inode bitmap
	- starting block address of inode table
	- number of free blocks and inodes
- located in block after superblock
- backup copies exist in each block group, unless sparse superblock feature is enabled

## 9. boot code
- ExtX FS containing OS kernel may have boot code
- located 1,024 bytes before superblock; first 2 sectors
- executed after control is passed from boot code in *Master Boot Record(MBR)*
- typically knows which blocks have been allocated to kernel and will load it into memory
- many Linux systems use *boot loader* in MBR to load kernel without boot code

## 10. FS category example image
- *fsstat* output:
	``` sh
	$ fsstat –f linux-ext3 ext3.dd
	FILE SYSTEM INFORMATION
	--------------------------------------------
	File System Type: Ext3
	Volume Name:
	Volume ID: e4636f48c4ec85946e489517a5067a07
	Last Written at: Wed Aug 4 09:41:13 2004
	Last Checked at: Thu Jun 12 10:35:54 2003
	Last Mounted at: Wed Aug 4 09:41:13 2004
	Unmounted properly
	Last mounted on:
	Source OS: Linux
	Dynamic Structure
	Compat Features: Journal,
	InCompat Features: Filetype, Needs Recovery,
	Read Only Compat Features: Sparse Super, Has Large Files,
	Journal ID: 00
	Journal Inode: 8
	METADATA INFORMATION
	--------------------------------------------
	Inode Range: 1 - 1921984
	Root Directory: 2
	Free Inodes: 1917115
	CONTENT INFORMATION
	--------------------------------------------
	Block Range: 0 - 3841534
	Block Size: 4096
	Free Blocks: 663631
	BLOCK GROUP INFORMATION
	--------------------------------------------
	Number of Block Groups: 118
	Inodes per group: 16288
	Blocks per group: 32768
	Group: 0:
	Inode Range: 1 - 16288
	Block Range: 0 - 32767
	Layout:
	Super Block: 0 - 0
	Group Descriptor Table: 1 - 1
	Data bitmap: 2 - 2
	Inode bitmap: 3 - 3
	Inode Table: 4 - 512
	Data Blocks: 513 - 32767
	Free Inodes: 16245 (99%)
	Free Blocks: 0 (0%)
	Total Directories: 11
	Group: 1:
	Inode Range: 16289 - 32576
	Block Range: 32768 – 65535
	[REMOVED]
	```
	- shows general data from superblock; total \# of inodes/blocks, block sizes
	- layout details for each block group are also given
	- feature flags and temporal information are also shown

## 11. FS category analysis technique
1. locate superblock
	- always located at offset 1,024 from start of FS
2. process superblock
	- essential to investigation
	- use backup copies if corrupt
	- feature flags identify if FS has any unique features
3. calculate size of FS
	- use block size and number of blocks in FS
	- compare with volume size to check for volume slack
4. process group descriptor table
	- used to loate DS that will determine allocation status of blocks/inodes
	- together with superblock, locates all DS used by files/directories

## 12. FS category analysis considerations
- usually doesn't contain evidence of event
- lots of unused space in superblock to hide data
- sectors for boot code may not be used and can contain hidden data
- compare FS size with volume size for volume slack
- unused space after group descriptor table can be used to hide data; backup copies should be checked as well
- superblock signature 0xef53 in bytes 56~56 can be used to find backup superblocks
- block group boundaries can be guessed using expected block size and the fact that size of block group is frequently based on \# bits in one block

## 13. FS category analysis scenario
- tips:
	- valid superblocks signatures should have a series of equally spaced hits due to backups
	- block groups are typically based on size of block; distance between superblocks should be 8,192, 16,384, 32,768, etc
- *sigfind* output for boot partition:
	``` sh
	$ sigfind –o 56 –l ef53 disk-8.dd
	Block size: 512 Offset: 56
	Block: 298661 (-)
	Block: 315677 (+17016)
	Block: 353313 (+37636)
	Block: 377550 (+24237)
	```
	- distances are not powers of 2
	- not consistently spaced
	``` sh
	[REMOVED]
	Block: 2056322 (+274327)
	Block: 2072706 (+16384)
	Block: 2105474 (+32768)
	Block: 2138242 (+32768)
	Block: 2171010 (+32768)
	Block: 2203778 (+32768)
	```
	- last 4 entries are same distance apart
	- first superblock seems to be in sector 2,056,322
	- probably 16,384 sectors per group with sparse superblock feature enabled
	- test theory:
		- extract and process 1,024 bytes from sector 2,056,322
		- block size 1,024 bytes
		- sparse superblock enabled
		- total of 104,422 blocks in FS
		- 8,192 blocks per group; space between backups should be 16,384
		- superblock block group identification field is 0; indicating primary superblock

- *sigfind* output for after boot partition:
	``` sh
	Block: 2265167 (+61389)
	Block: 2265733 (+566)
	Block: 2265985 (+252)
	Block: 2266183 (+198)
	Block: 2266357 (+174)
	Block: 2266457 (+100)
	[REMOVED]
	```
	- hits are not evenly spaced
	``` sh
	[REMOVED]
	Block: 2278273 (+2800)
	Block: 2281551 (+3278)
	Block: 2282617 (+1066)
	Block: 2314319 (+31702)
	Block: 2347087 (+32768)
	Block: 2379855 (+32768)
	Block: 2412623 (+32768)
	```
	- examine sector 2,314,319:
		- Ext3
		- 1,024 block size
		- 8,192 blocks per group
		- reports to be from block group 3; block group 0 started 49,152 sectors ago

![[FS category analysis scenario layout.png]]

## 14. content category

## 15. blocks
- can be 1,024, 2048, or 4,096 bytes
- size given in superblock
- all blocks are given an address, starting with 0
- all blocks belong to a block group, except where superblock has defined a reserved area at beginning of FS; group 0 follows after reserved area
group # = (block - FIRST_DATA_BLOCK) / BLOCKS_PER_GROUP

## 16. allocation status
- block allocation status determined by group block bitmap
- block group sizes are created so the block bitmap needs only 1 block; can be manually overridden
- not every allocated block is allocated to a file:
	- superblocks
	- group descriptor tables
	- bitmaps
	- inode tables
- *dstat* tool can be used to show allocation status and block group of block:
	``` sh
	$ dstat -f linux-ext3 ext3.dd 14380
	Block: 14380
	Allocated
	Group: 0
	```

## 17. content category allocation algorithms
- on Linux, allocation of blocks is done on group basis
- when block is allocated to inode, Linux will use a first-available strategy
- allocating a block in the same group as the inode makes reading quicker for disks
- when existing file is being made larger, next-availble strategy is used
- ExtX typically prevents user from allocating every block in FS; superblock defines how many blocks are reserved for specific user
- when Linux writes data to a block, it zeroes unused bytes; should not be any data in slack space of last file block

## 18. content category analysis techniques
1. locate block
	- use block size given in superblock
2. determine allocation status of block
	1. determine block group
	2. locate group entry in group descriptor header to find block bitmap
	3. process bitmap

## 19. content category analysis considerations
- sometimes searches can be restricted to group, not entire FS
- group location can be determined using *fsstat*
- groups with high write activity have higher chances of overwriting deleted data

## 20. content category analysis scenario
objective: read contents of last block in FS
1. read superblock
	- block size in bytes 24~27; 2
	- 0x0400<<2 = 0x1000; 4,096 bytes in block
	- block \# in bytes 4~7; 512,064
2. determine last block
	- 512,063 blocks * 8 sectors = sector 4,096,504 

## 21. metadata category
- file's primary metadata is stored in inode
- additional metadata stored in extended attributes

## 22. inode
- all same size; defined in superblock
- one inode allocated to every file/directory
- each inode has address starting with 1
- a set of inodes as a table is assigned to each group
group = (inode - 1) / INODES_PER_GROUP
- inodes 1~10 are typically reserved and should be allocated:
	- inode 2 used for root directory
	- inode 1 keeps track of bad blokcs; does not have any special status in Linux kernel
	- inode 8 used by journal
- superblock has value of first non-reserved inode
- first user file typically allocated inode 11; frequently used for lost+found directory
- each inode has a static number of fields; additional information may be stored in extended attributes and indirect block pointers
- inode contains:
	- file size:
		- 64 bits in newer versions
		- 32 bits in older versions; max 4GB
	- ownership:
		- stored using user ID and group ID
	- temporal info:
		- times for last access(A), modification(M), change(C), and deletion(D)
		- time stored as number of seconds since Jan 1st, 1970 UTC
		- C-time corresponds to when file metadata was last changed
		- M- and A-time can be set easily using *touch*
	- mode field:
		- type of file
		- permission value
		- special properties of file/directory
	- link count equal to number of files that point to it:
		- inode becomes unallocated if this is 0 and no process has file open
		- if 0 with process that has file open, becomes orphan inode; deleted when process closes it
	- generation ID:
		- used by *Network File System(NFS)* to determine when new file has been allocated
		- similar to sequence number in NTFS

## 23. lost+found directory
- any inode that is allocated but does not have a file name pointing to it is added to lost+found with a new name

## 24. different file types
- hardware devices:
	- assigned one or more file names
	- file type of *block device* or *character device*
	- block device:
		- devices that operate on block sized chunks of data
		- typically also has character device created for it
		- e.g. hard disk
	- character device:
		- a.k.a *raw device*
		- do not operate in blocks
		- e.g. keyboard
	- inode space that stores info on what blocks have been allocated to a file are used to store device identifier information
- FIFO:
	- a.k.a *named pipe*
	- process can open file and receive data written to it by another process
	- data stored in kernel memory and not disk
- *Unix socket*:
	- used for two-way communication between processes
	- data is not written to disk
- symbolic link:
	- soft link
	- provides shortcut to another file/directory

## 25. special file/directory properties
- *sticky bit*:
	- if set on executable file, some process data will remain in memory after process ends
	- if set on directory, only owner of a file can delete the directory contents
- *Set User ID(SUID)*, *Set Group ID(SGID)*:
	- when set for executable, process takes on user/group ID from file instead of user that started process
	- allows non-privileged user to run specific programs as privileged user
	- when SGID is set on directory, all files in directory are assigned same group ID as directory

## 26. block pointers
- *direct pointer*:
	- 12 exist in inode
	- points to file block
- *indirect block pointer*:
	- for larger files
	- 1 exists in inode
	- points to block allocated to storing remaining addresses
	- addresses in block are all 4 bytes; \# of addresses is based on block size
- *double indirect block*:
	- for even larger files
	- 1 exists in inode
	- inode points to block that contains list of indirect block pointers
- *triple indirect block*:
	- 1 exists in inode
	- contains addresses of double indirect blocks

![[direct and indirect block pointers.png]]

## 27. sparse blocks
- exists when original block was undefined or all zeros
- undefined blocks exist because program forced file to be specific size, but never wrote data to some parts
- address 0 is placed in block pointer for blocks of all zeros

## 28. inode attributes
- defined in flags field
- may not be supported by OS
- supported attributes:
	- does not update A-time of files/directories
	- identifies files that should be written to disk as soon as changes occur, instead of being cached
	- 2 attributes for security
	- makes files append only; data can't be removed
	- makes files immutable; no modification
	- dump attribute identifies files that should not be saved when *dump* is used to backup files
- attributes not generally supported:
	- secure deletion: 
		- wipes blocks associated with file
	- compression
	- undelete:
		- makes backup copy of file for recovery
- extended attributes:
	- with Linux, only allowed when FS has been mounted with user_xattr option and kernel has support
	- list of name and value pairs
	- a user can create any pair using *setfattr*
	- stored in block that has been allocated to the file
	- if more than one file has the same extended attributes, they will share the block
	- used by OS for POSIX *Access Control Lists(ACL)*

## 29. metadata category example image
- *istat* output
	``` sh
	$ istat –f linux-ext3 ext3.dd 16
	inode: 16
	Allocated
	Group: 0
	Generation Id: 199922874
	uid / gid: 500 / 500
	mode: -rw-r--r--
	size: 10240000
	num of links: 1
	Inode Times:
	Accessed: Fri Aug 1 06:32:13 2003
	File Modified: Fri Aug 1 06:24:01 2003
	Inode Modified: Fri Aug 1 06:25:58 2003
	Direct Blocks:
	14380 14381 14382 14383 14384 14385 14386 14387
	14388 14389 14390 14391 14393 14394 14395 14396
	[REMOVED]
	16880 16881 16882 16883
	Indirect Blocks:
	14392 15417 15418 16443
	```
	- file is 10MB
	- required 4 indirect block pointers
	- shows permission and file types
	- shows MAC times

## 30. metadata category allocation algorithms
- different OS or kernel could use different algorithms
- inode allocation:
	- non-directory inode: 
		- allocated to same block group as parent directory
		- if parent directory has no free inode/block select random group with quadratic hash
	- directory inode:
		- place in group that has not been used much:
			- calculate average number of free inodes and blocks per group
			- first available search for group with free inode and block \# smaller than average
	- when an inode is allocated, contents are cleared:
		- MAC set to current time
		- D-time set to 0
		- link count set to 1 for file, 2 to directory
		- generation field set with current counter value
	- file deletion:
		- Ext3 sets file size to 0 and wipes block pointers in inode and indirect blocks
		- Ext2 does not wipe block pointers
		- deletion from moving to different volume sets A-time to when content was read; helps distinguish move and delete
- time value updating:
	- A-time of file:
		- process reads contents
		- file is copied
		- file is moved to new volume; not same volume
	- A-time of directory:
		- directory listing done on contents
		- file or subdirectory opened
	- M-time of file:
		- file content change
		- file moved; not copied
	- M-time of directory
		- file is created/deleted inside
	- C-time:
		- file permissions or ownership changes
		- file/directory content changes
		- file moved
	- D-time:
		- file deleted

	| change type | file M-time | file A-time | file C-time | file D-time | parent M-time | parent A-time | parent C-time |
	| --- | --- | --- | --- | --- | --- | --- | --- |
	| file creation | update | update | update | 0 | update | X | update |
	| copy | X | update | X | X | X | update | X |
	| move | X | X | X | X | update | update | update |
	- copying creates new file at destination

## 31. metadata category analysis techniques
1. determine block group of inode
2. process group descriptor of group
	- locate inode table
3. process inode entry in table
4. read pointers to see file

## 32. metadata category analysis considerations
- examine unallocated inode entries to view temporal data
- in Ext3 FS deleted file names point to their inode entries; file names may be overwritten before inode data is
- extended attributes could contain important data associated with file
- unallocated inode entries may exist longer in groups with little activity
- M- and A- times can be set easily; not to be trusted
- time values are stored with respect to UTC; should be converted to local time
- Linux will write 0s to unused bytes of a block; deleted files exist only in unallocated blocks
- if you encounter ExtX with a deleted file whose size and block pointers were not wiped, indirect block pointers may point to reallocated blocks
- orphane inode list in superblock should be examined during investigation; can be found with *fsstat*

## 33. metadata category analysis scenario
- timeline of system created in TSK
	``` sh
	[REMOVED]
	Wed Nov 08 2000 08:51:56
	..c -/-rwxr-xr-x 1010 users /usr/man/.Ci/ /Anap
	..c -/-rwxr-xr-x 1010 users /usr/man/.Ci/bx
	Wed Nov 08 2000 08:52:09
	m.c l/lrwxrwxrwx root root /.bash_history -> /dev/null
	m.c l/lrwxrwxrwx root root /root/.bash_history -> /dev/null
	[REMOVED]
	```
	- 2 files in /usr/man/.Ci have changed time of 08:51:56, with User ID 1010

objective: determine user account that created files in suspicious directory

1. some User IDs in inodes are not in /etc/passwd:
	1. user account was deleted after file creation
	2. files came from an archive file and saved User ID from computer it was created
2. search unallocated blocks of FS containing /etc/ for deleted copy of passwd
	- search for string from current passwd
	- unsuccessful
3. search system for tar files containing files in question
	1. tar file found
	2. User ID of file can be changed easily; permissions of parent directory should be examined
	3. parent directory /usr/local/ is owned by root, and only it has write permissions
	4. attacker likely had access to root account

## 34. file name category

## 35. directory entries
- simple DS
- dynamic length; has field value identifying name length and where next directory can be found
- length of entry is rounded up to a multiple of 4
- contains:
	- file name
	- inode address where metadata can be found
- directories allocate blocks that will contain a list of directory entry DS
- size of directory corresponds to number of blocks it has allocated; irrelevant to how many files exist
- every directory starts with entries for . and .., current and parent directory
- file/directory deleted by increasing record length of previous directory entry
	![[directory entries example.png]]
- when new entry is created, OS compares record length with name length for each record:
	- 8 bytes of static fields exist after name
	- minimum record length is 8 + name length rounded up to multiple of 4
	- if record length is larger than necessary, entry is placed in unused space
- *fls* output
	``` sh
	$ fls -f linux-ext3 –a ext3.dd 69457
	d/d 69457: .
	d/d 53248: ..
	r/r 69458: abcdefg.txt
	r/r * 69459: file two.dat
	d/d 69460: subdir1
	r/r 69461: RSTUVWXY
	```
	- file two.dat has been deleted
	- types for deleted file is same; inode may not have been reallocated yet

## 36. links and mount points
- hard link:
	- additional name for file/directory in same FS
	- can't tell if it's original name or link
	- OS allocates new directory entry and points it to original inode and increments link count by 1
	- . and .. directories are hard links
- soft link:
	- second name for file/directory 
	- can span different FS
	- created using symbolic link, a special type of file
	- full path of destination file/directory is stored in either blocks allocated to file or in inode if path is less than 60 characters long

![[hard link soft link comparison.png]]

- mount points:
	- in Unix, directories can be used to mount volumes
	- when a volume is mounted, files in directory are not listed and files from mounted FS is shown
	- several FS may need to be referenced before finding file

![[volume mount in directory.png]]

## 37. hash trees
- when a FS is created, user can choose to use hash tree to organize files
- if a FS uses hash trees, a compatible feature flag will be set in superblock
- hash trees use directory entries, but they are sorted 
- similar to B-Trees in NTFS
- directory is using a hash tree:
	- each block will be a node in the tree
	- each node contains files whose hash value is in a given range
	- root node contains . and ..
	- root node also contains node descriptors, which contain a hash value and a block address
	- node descriptors are used to determine which block to jump to given a hash

![[hash tree with 2 leaves.png]]

## 38. file name category allocation algorithms
- when a new file name is created, Linux uses first-availble strategy
- new blocks are added as necessary and wiped before use
- entry can't cross a block boundary
- Linux will clear inode pointer in Ext2 FS when entry is deleted, but not Ext3

## 39. file name category analysis techniques
1. locate root directory
2. process directory content as sequence of directory entry DS
	- ignore reported size of entry to view unallocated file names
	- after 4 byte reaching boundaries, examine 4 bytes at a time for next directory entry

## 40. file name category analysis considerations
- it may be possible to infer order of file deletion using pointer values
	![[file deletion order.png]]
	- only B and D are the same
- Ext3 inode number is not cleared on delete; it is possible to obtain temporal information about deletion
- entry with short name might not have entry overwritten quickly 
- *fsck* can be used to remove unused space in directories
- inode may have been reallocated for file name found; type value can help determine this sometimes
- space between last directory entry and end of block is unused and can contain hidden data; especially true for first block of hash trees

## 41. file name category analysis scenario
- source of moved file:
	``` sh
	$ fls -f linux-ext3 ext3-8.dd 1840555
	r/r 1840556: log-001.dat
	r/r 1840560: log-002.dat
	r/r 1840566: log-003.dat
	r/r 1840569: log-004.dat
	r/r 32579: snifferlog-1.dat
	r/r 1840579: log-005.dat
	r/r 1840585: log-006.dat
	```
	1. to find executable that created file, search for full path of file
		- unsuccessful
	2. snifferlog-1.dat has a significantly different address
		- block group was full when file was created or it was moved
		- *fsstat* output:
			``` sh
			Group: 113:
			Inode Range: 1840545 - 1856832
			Block Range: 3702784 - 3735551
			Free Inodes: 16271 (99%)
			Free Blocks: 15728 (48%)
			```
		- *fsstat* output 2:
			``` sh
			Group: 2:
			Inode Range: 32577 - 48864
			Block Range: 65536 - 98303
			Free Inodes: 16268 (99%)
			Free Blocks: 0 (0%)
			```
	3. look for directories in block group 2 that could have been parent of sniffer file
		- *ils* output:
			``` sh
			$ ils -f linux-ext3 -m -a ext3-8.dd 32577-48864 | grep "|d"
			<ext3-8.dd-alive-32577>|0|32577|16893|drwxrwxr-x|4|500|500|0|4096|
			<ext3-8.dd-alive-32655>|0|32655|16893|drwxrwxr-x|2|500|500|0|4096|
			<ext3-8.dd-alive-32660>|0|32660|16877|drwxr-xr-x|2|500|500|0|4096|
			```
	4. view directory with *fls*
		- *fls* output for directory 1:
			``` sh
			$ fls -f linux-ext3 ext3-8.dd 32577
			r/r 32578: only_live_twice.mp3
			r/r 32582: goldfinger.mp3
			r/r 32580: lic_to_kill.mp3
			r/r 32581: diamonds_forever.mp3
			```
			- inode address of sniffer log is 32,579; could have been allocated between only_... and lic_...
			- gold... has larger inode but is located between them
			- gold... has less characters than sniffer file
	5. examine files in suspicious directory
		- only_... is an executable
		- other files have same format as sniffer file
		- packets in only_... have timestamps before sniffer file
		- packets in lic_... have timestamps after sniffer file
	6. construct approximate timeline
		1. only_... created
		2. sniffer file created
		3. lic_... created
		4. diamonds_... created
		5. sniffer file moved
		6. gold... created; overwriting directory entry of moved file

	![[file name category analysis scenario 1.png]]
- file deletion order
	1. find suspicious directory
	2. view contents
		- 8 deleted file names
		- all corresponding inodes have been reallocated to new files
	3. parse directory entries

		| Byte Location | Name | Record Length | Needed Length | 
		| --- | --- | --- | --- | 
		| 12 | .. | 1,012 | 12 | 
		| 24 | config.dat | 20 | 20 | 
		| 44 | readme.txt | 104 | 20 | 
		| 64 | allfiles.tar | 20 | 20 | 
		| 84 | random.dat | 64 | 20 | 
		| 104 | mytools.zip | 44 | 20 | 
		| 124 | delete-files.sh | 24 | 24 | 
		| 148 | sniffer | 876 | 16 | 
		| 164 | passwords.txt | 860 | 860 | 
		- if record length of unallocated directory entry is actual size that it needs, next directory entry was deleted after it was
		- config.dat deleted before readme.txt
		- allfiles.tar was deleted before random.dat
		- delete-files.sh was deleted before sniffer
		- mytools.zip was deleted after delete-files.sh but before sniffer
		- passwords.txt was deleted before sniffer 
		- random.dat was deleted after mytools.zip
		- readme.txt was deleted before sniffer
		- can infer files have been deleted in alphabetical order
