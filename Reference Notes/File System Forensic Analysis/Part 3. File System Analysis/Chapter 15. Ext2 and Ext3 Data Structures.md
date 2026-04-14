## 1. superblock
- fields:

	| Byte Range | Description | Essential | 
	| --- | --- | --- | 
	| 0–3 | Number of inodes in file system | Yes | 
	| 4–7 | Number of blocks in file system | Yes | 
	| 8–11 | Number of blocks reserved to prevent file system from filling up | No | 
	| 12–15 | Number of unallocated blocks | No | 
	| 16–19 | Number of unallocated inodes | No | 
	| 20–23 | Block where block group 0 starts | Yes | 
	| 24–27 | Block size(saved as the number of places to shift 1,024 to the left) | Yes | 
	| 28–31 | Fragment size(saved as the number of bits to shift 1,024 to the left) | Yes | 
	| 32–35 | Number of blocks in each block group | Yes | 
	| 36–39 | Number of fragments in each block group | Yes | 
	| 40–43 | Number of inodes in each block group | Yes | 
	| 44–47 | Last mount time | No | 
	| 48–51 | Last written time | No | 
	| 52–53 | Current mount count | No | 
	| 54–55 | Maximum mount count | No | 
	| 56–57 | Signature(0xef53) | No | 
	| 58–59 | File system state | No | 
	| 60–61 | Error handling method | No | 
	| 62–63 | Minor version | No | 
	| 64–67 | Last consistency check time | No | 
	| 68–71 | Interval between forced consistency checks | No | 
	| 72–75 | Creator OS | No | 
	| 76–79 | Major version | Yes | 
	| 80–81 | UID that can use reserved blocks | No | 
	| 82–83 | GID that can use reserved blocks | No | 
	| 84–87 | First non-reserved inode in file system | No | 
	| 88–89 | Size of each inode structure | Yes | 
	| 90–91 | Block group that this superblock is part of(if backup copy) | No | 
	| 92–95 | Compatible feature flags | No | 
	| 96–99 | Incompatible feature flags | Yes | 
	| 100–103 | Read only feature flags | No | 
	| 104–119 | File system ID | No | 
	| 120–135 | Volume name | No | 
	| 136–199 | Path where last mounted on | No | 
	| 200–203 | Algorithm usage bitmap | No | 
	| 204–204 | Number of blocks to preallocate for files | No | 
	| 205–205 | Number of blocks to preallocate for directories | No | 
	| 206–207 | Unused | No | 
	| 208–223 | Journal ID | No | 
	| 224–227 | Journal inode | No | 
	| 228–231 | Journal device | No | 
	| 232–235 | Head of orphan inode list | No | 
	| 236–1023 | Unused | No | 
	- flag values:

		| Flag Value | Description | Essential | 
		| --- | --- | --- | 
		| 0x0001 | File system is clean | No | 
		| 0x0002 | File system has errors | No | 
		| 0x0004 | Orphan inodes are being recovered | No | 
	- error handling field:
		
		| Value | Description | Essential | 
		| --- | --- | --- | 
		| 1 | Continue | No | 
		| 2 | Remount file system as read only | No | 
		| 3 | Panic | No | 
	- OS creator field:
		
		| Value | Description | Essential | 
		| --- | --- | --- | 
		| 0 | Linux | No | 
		| 1 | GNU Hurd | No | 
		| 2 | Masix | No | 
		| 3 | FreeBSD | No | 
		| 4 | Lites | No | 
	- major version field:

		| Value | Description | Essential | 
		| --- | --- | --- | 
		| 0 | Original version | Yes | 
		| 1 | "Dynamic" version | Yes | 
		- dynamic means each inode can be dynamic size
	- if major version is not dynamic, values from byte 84~ may not be accurate
	- compatible feature values:
		
		| Flag Value | Description | Essential | 
		| --- | --- | --- | 
		| 0x0001 | Preallocate directory blocks to reduce fragmentation | No | 
		| 0x0002 | AFS server inodes exist | No | 
		| 0x0004 | File system has a journal(Ext3) | No | 
		| 0x0008 | Inodes have extended attributes | No | 
		| 0x0010 | File system can resize itself for larger partitions | No | 
		| 0x0020 | Directories use hash index | No | 
	- incompatible feature values:

		| Flag Value | Description | Essential | 
		| --- | --- | --- | 
		| 0x0001 | Compression(not yet supported) | Yes | 
		| 0x0002 | Directory entries contain a file type field | Yes | 
		| 0x0004 | File system needs recovery | No | 
		| 0x0008 | File system uses a journal device | No | 
	- read only compatible feature values:

		| Flag Value | Description | Essential | 
		| --- | --- | --- | 
		| 0x0001 | Sparse superblocks and group descriptor tables | No | 
		| 0x0002 | File system contains a large file | No | 
		| 0x0004 | Directories use B-Trees(not implemented) | No | 

## 2. superblock example
- *dd* output:
	``` sh
	$ dd if=ext3.dd bs=1024 skip=1 count=1 | xxd
	0000000: c053 1d00 ff9d 3a00 4cee 0200 4708 0b00 .S....:.L...G...
	0000016: 6745 1d00 0000 0000 0200 0000 0200 0000 gE..............
	0000032: 0080 0000 0080 0000 a03f 0000 c9fd 1141 .........?.....A
	0000048: c9fd 1141 3601 2500 53ef 0100 0100 0000 ...A6.%.S.......
	0000064: da9d e83e 004e ed00 0000 0000 0100 0000 ...>.N..........
	0000080: 0000 0000 0b00 0000 8000 0000 0400 0000 ................
	0000096: 0600 0000 0300 0000 077a 06a5 1795 486e .........z....Hn
	0000112: 9485 ecc4 486f 63e4 0000 0000 0000 0000 ....Hoc.........
	0000128: 0000 0000 0000 0000 0000 0000 0000 0000 ................
	[REMOVED]
	0000224: 0800 0000 0000 0000 0000 0000 0000 0000 ................
	[REMOVED]
	0001008: 0000 0000 0000 0000 0000 0000 0000 0000 ................
	```
	- bytes 0~3: 1,921,984 inodes
	- bytes 4~7: 3,841,535 blocks
	- bytes 20~23: block 0 is where group 0 starts
	- bytes 24~27, 28~31: block and fragment size of 4,096 bytes
	- byte 32~35: 32,768 blocks per group
	- bytes 40~43: 16,288 inodes per group
	- bytes 76~79: dynamic version
	- bytes 88~91: size of inode 128
	- bytes 92~95: journal flag set; Ext3 FS
	- bytes 96~99: recovery should be done during next boot, special directory entries are being used
	- bytes 100~103: large files exist, not every group has backup of superblock
	- bytes 224~227: journal in inode 8

## 3. group descriptor tables
- fields:

	| Byte Range | Description | Essential |
	| --- | --- | --- |
	| 0–3 | Starting block address of block bitmap | Yes |
	| 4–7 | Starting block address of inode bitmap | Yes |
	| 8–11 | Starting block address of inode table | Yes |
	| 12–13 | Number of unallocated blocks in group | No |
	| 14–15 | Number of unallocated inodes in group | No |
	| 16–17 | Number of directories in group | No |
	| 18–31 | Unused | No |

## 4. group descriptor table example
- *dcat* output:
	``` sh
	$ dcat –f linux-ext3 ext3.dd 1 | xxd
	0000000: 0200 0000 0300 0000 0400 0000 d610 7b3f ..............{?
	0000016: 0a00 0000 0000 0000 0000 0000 0000 0000 ................
	0000032: 0280 0000 0380 0000 0480 0000 0000 8e3f ...............?
	0000048: 0100 0000 0000 0000 0000 0000 0000 0000 ................
	[REMOVED]
	```
	- bytes 0~3: block bitmap located in block 2
	- bytes 4~7: inode bitmap located in block 3
	- bytes 8~11: inode table starts in block 4

## 5. block bitmap example
- *dcat* output:
	``` sh
	$ dcat –f linux-ext3 ext3.dd 2 | xxd
	0000000: ffff ffff ffff ffff ffff ffff ffff ffff ................
	[REMOVED]
	0001168: ff01 fcff ffff 0ffe ffff ffff 03fe ffff ................
	```
	- bytes 1,169: 0x01
	- block 9,352 is allocated
	- blocks 9,353, 9359 are not

## 6. inode
- fields:

	| Byte Range | Description | Essential | 
	| --- | --- | --- |
	| 0–1 | File mode(type and permissions) | Yes | 
	| 2–3 | Lower 16 bits of user ID | No | 
	| 4–7 | Lower 32 bits of size in bytes | Yes | 
	| 8–11 | Access Time | No | 
	| 12–15 | Change Time | No | 
	| 16–19 | Modification time | No | 
	| 20–23 | Deletion time | No | 
	| 24–25 | Lower 16 bits of group ID | No | 
	| 26–27 | Link count | No | 
	| 28–31 | Sector count | No | 
	| 32–35 | Flags | No | 
	| 36–39 | Unused | No | 
	| 40–87 | 12 direct block pointers | Yes | 
	| 88–91 | 1 single indirect block pointer | Yes | 
	| 92–95 | 1 double indirect block pointer | Yes | 
	| 96–99 | 1 triple indirect block pointer | Yes | 
	| 100–103 | Generation number(NFS) | No | 
	| 104–107 | Extended attribute block(File ACL) | No | 
	| 108–111 | Upper 32 bits of size / Directory ACL | Yes / No | 
	| 112–115 | Block address of fragment | No | 
	| 116–116 | Fragment index in block | No | 
	| 117–117 | Fragment size | No | 
	| 118–119 | Unused | No | 
	| 120–121 | Upper 16 bits of user ID | No | 
	| 122–123 | Upper 16 bits of group ID | No | 
	| 124–127 | Unused | No | 
	- 16-bit file mode broken up into 3 sections; 4, 3, and 9 bits
	- lower 9 bits of file mode (permission flags):
	
		| Permission Flag | Description | 
		| --- | --- | 
		| 0x001 | Other—execute | 
		| 0x002 | Other—write | 
		| 0x004 | Other—read | 
		| 0x008 | Group—execute | 
		| 0x010 | Group—write | 
		| 0x020 | Group—read | 
		| 0x040 | User—execute | 
		| 0x080 | User—write | 
		| 0x100 | User—read | 
	- mid 3 bits of file mode (properties flags):

		| Flag Value | Description | 
		| --- | --- | 
		| 0x200 | Sticky bit | 
		| 0x400 | Set group ID | 
		| 0x800 | Set user ID | 
	- upper 4 bits of file mode (type of file):

		| Type Value | Description | 
		| --- | --- | 
		| 0x1000 | FIFO | 
		| 0x2000 | Character device | 
		| 0x4000 | Directory | 
		| 0x6000 | Block device | 
		| 0x8000 | Regular file | 
		| 0xA000 | Symbolic link | 
		| 0xC000 | Unix socket | 
	- time values stored as number of seconds since Jan 1st, 1970 UTC
	- flag field:
		
		| Flag Value | Description | 
		| --- | --- | 
		| 0x00000001 | Secure deletion(not used) | 
		| 0x00000002 | Keep a copy of data when deleted(not used) | 
		| 0x00000004 | File compression(not used) | 
		| 0x00000008 | Synchronous updates—new data is written immediately to disk | 
		| 0x00000010 | Immutable file—content cannot be changed | 
		| 0x00000020 | Append only | 
		| 0x00000040 | File is not included in 'dump' command | 
		| 0x00000080 | A-time is not updated | 
		| 0x00001000 | Hash indexed directory | 
		| 0x00002000 | File data is journaled with Ext3 | 

## 7. inode example
- *dd* output:
	``` sh
	$ dd if=ext3.dd bs=4096 skip=4 | dd bs=128 skip=15 count=1 | xxd
	0000000: a481 f401 0040 9c00 6d09 2a3f f607 2a3f .....@..m.*?..*?
	0000016: 8107 2a3f 0000 0000 f401 0100 404e 0000 ..*?........@N..
	0000032: 0000 0000 0000 0000 2c38 0000 2d38 0000 ........,8..-8..
	0000048: 2e38 0000 2f38 0000 3038 0000 3138 0000 .8../8..08..18..
	0000064: 3238 0000 3338 0000 3438 0000 3538 0000 28..38..48..58..
	0000080: 3638 0000 3738 0000 3838 0000 393c 0000 68..78..88..9<..
	0000096: 0000 0000 ba94 ea0b 0000 0000 0000 0000 ................
	0000112: 0000 0000 0000 0000 0000 0000 0000 0000 ................
	```
	- bytes 0~1: mode
	- everyone can read, group can read, user can write, user can read
	- regualr file
	- bytes 4~7: size of file 10,240,000 bytes
	- bytes 8~11: A-time August 1st, 2003 at 06:32:13 UTC
	- bytes 26~27: link count 1
	- bytes 32~35: no special flags or attributes
	- bytes 40~43: first direct pointer to 14,380
	- bytes 44~47: second direct pointer to 14,381
	- bytes 88~91: single indirect pointer to 14,392
		``` sh
		$ dcat –f linux-ext3 ext3.dd 14392 | xxd
		0000000: 3938 0000 3a38 0000 3b38 0000 3c38 0000 98..:8..;8..<8..
		0000016: 3d38 0000 3e38 0000 3f38 0000 4038 0000 =8..>8..?8..@8..
		0000032: 4138 0000 4238 0000 4338 0000 4438 0000 A8..B8..C8..D8..
		[REMOVED]
		```
	- bytes 92~95: double indirect pointer to 15,417
- inode allocation check:
	``` sh
	$ dcat –f linux-ext3 ext3.dd 3 | xxd
	0000000: fff7 fcff 1f00 0000 00c8 0000 0000 0000 ................
	```
	- (16-1)/8 = 1\*8 + 7
	- MSB in byte 1; allocated
- *istat* inode details:
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

## 8. extended attributes
- extended attribute block has 3 sections:
	- first 32 bytes used by header
	- second section follows header, contains list of attribute name entries
	- third section starts from end, contains values for each attribute pair; not necessarily in same order as name entries

![[extended attributes block layout.png]]
- header fields:

	| Byte Range | Description | Essential | 
	| --- | --- | --- | 
	| 0–3 | Signature(0xEA020000) | No | 
	| 4–7 | Reference count | No | 
	| 8–11 | Number of blocks | Yes | 
	| 12–15 | Hash | No | 
	| 16–31 | Reserved | No | 
	- reference count shows number of files using this block
	- hash value used to determine if two files have same attributes
- name entry DS:

	| Byte Range | Description | Essential | 
	| --- | --- | --- | 
	| 0–0 | Length of name | Yes | 
	| 1–1 | Attribute type | Yes | 
	| 2–3 | Offset to value | Yes | 
	| 4–7 | Block location of value | Yes | 
	| 8–11 | Size of value | Yes | 
	| 12–15 | Hash of value | No | 
	| 16+ | Name in ASCII | Yes | 
	- length of name determines length of entry, and next entry starts at next 4-byte boundary
	- offset is byte offset in block specified
	- size value is \# of bytes in value
	- entry type values:

		| Type Value | Description | 
		| --- | --- | 
		| 1 | User space attribute | 
		| 2 | POSIX ACL | 
		| 3 | POSIX ACL Default(directories only) | 
		| 4 | Trusted space attribute | 
		| 5 | LUSTRE(not currently used by Linux) | 
		| 6 | Security space attribute | 
		- if type is one of POSIX ACL types, value has own set of DS
- POSIX ACL header:

	| Byte Range | Description | Essential | 
	| --- | --- | --- | 
	| 0–3 | Version(1) | Yes | 
- POSIX ACL entries:

	| Byte Range | Description | Essential |
	| --- | --- | --- |
	| 0–1 | Type(tag) | Yes |
	| 2–3 | Permissions | Yes |
	| 4–7 | User / Group ID(not included for some types) | Yes |
	- type field:

		| Type | Description | 
		| --- | --- | 
		| 0x01 | User—specified in inode | 
		| 0x04 | Group—specified in inode | 
		| 0x20 | Other—all other users | 
		| 0x10 | Effective rights mask | 
		| 0x02 | User—specified in attribute | 
		| 0x08 | Group—specified in attribute | 
	- permissions field:

		| Permission Flag | Description | 
		| --- | --- | 
		| 0x01 | Execute | 
		| 0x02 | Write | 
		| 0x04 | Read | 

## 9. extended attribute example
- *dcat* output:
	``` sh
	$ dcat –f linux-ext3 ext3-2.dd 1238
	0000000: 0000 02ea 0100 0000 0100 0000 7447 05e8 ............tG..
	0000016: 0000 0000 0000 0000 0000 0000 0000 0000 ................
	0000032: 0601 c003 0000 0000 1900 0000 a8e9 5147 ..............QG
	0000048: 736f 7572 6365 0000 0002 dc03 0000 0000 source..........
	0000064: 2400 0000 2500 ad01 0000 0000 0000 0000 $...%...........
	[REMOVED]
	0000944: 0000 0000 0000 0000 0000 0000 0000 0000 ................
	0000960: 7777 772e 6469 6774 6974 616c 2d65 7669 www.digtital-evi
	0000976: 6465 6e63 652e 6f72 6700 0000 0100 0000 dence.org.......
	0000992: 0100 0600 0200 0400 4a00 0000 0200 0600 ........J.......
	0001008: f401 0000 0400 0400 1000 0600 2000 0400 ............ ...
	```
	- bytes 0~3: signature value
	- bytes 4~7: reference count 1
	- bytes 8~11: block count 1
	- byte 32: entry start, name length 6
	- byte 33: type user space attribute
	- bytes 34~35: value at byte offset 960
	- bytes 40~43: size of value is 25 bytes
	- bytes 48~53: start of name, source
	- bytes 960~984: attribute value, www[.]digtital-evidence[.]org
	- bytes 988~1023: second attribute
		- bytes 988~991: header
		- bytes 992~993: first entry with type 1
		- bytes 994~995: owner has read/write permissions
		- bytes 996~997: second entry, type 2 
		- bytes 998~1003: read permissions to user with ID 74
- *istat* output:
	``` sh
	$ istat –f linux-ext3 ext3-2.dd 70
	[REMOVED]
	Extended Attributes (Block: 1238)
	user.source=www.digtital-evidence.org
	POSIX Access Control List Entries:
	uid: 0: Read, Write
	uid: 74: Read
	uid: 500: Read, Write
	gid: 0: Read
	mask: Read, Write
	other: Read
	[REMOVED]
	```

## 10. directory entry
- incompatible features value identifies format of directory entry
- original format:

	| Byte Range | Description | Essential | 
	| --- | --- | --- | 
	| 0–3 | Inode value | Yes | 
	| 4–5 | Length of this entry | Yes | 
	| 6–7 | Name length | Yes | 
	| 8+ | Name in ASCII | Yes | 
	- DS aligned on 4-bytes boundaries for Linux
- second version:

	| Byte Range | Description | Essential |
	| --- | --- | --- |
	| 0–3 | Inode value | Yes |
	| 4–5 | Length of this entry | Yes |
	| 6–6 | Name length | Yes |
	| 7–7 | File type | No |
	| 8+ | Name in ASCII | Yes |
	- file type values:

		| value | Description |
		| --- | --- |
		| 0 | Unknown type |
		| 1 | Regular file |
		| 2 | Directory |
		| 3 | Character device |
		| 4 | Block device |
		| 5 | FIFO |
		| 6 | Unix Socket |
		| 7 | Symbolic link |

## 11. directory entry example
- *icat* output:
	``` sh
	$ icat –f linux-ext3 ext3.dd 69457 | xxd
	0000000: 510f 0100 0c00 0102 2e00 0000 00d0 0000 Q...............
	0000016: 0c00 0202 2e2e 0000 520f 0100 2800 0b01 ........R...(...
	0000032: 6162 6364 6566 672e 7478 7400 530f 0100 abcdefg.txt.S...
	0000048: 1400 0c01 6669 6c65 2074 776f 2e64 6174 ....file two.dat
	0000064: 540f 0100 1000 0702 7375 6264 6972 3100 T.......subdir1.
	0000080: 550f 0100 b003 0801 5253 5455 5657 5859 U.......RSTUVWXY
	0000096: 0000 0000 0000 0000 0000 0000 0000 0000 ................
	[REMOVED]
	```
	- bytes 0~3: inode corresponding to first entry is 64,457
	- bytes 4~5: directory entry 12 bytes
	- byte 6: name is 1 byte long
	- byte 7: entry for directory
	- byte 8: name .
	- bytes 12~23: second entry
	- bytes 23~63: third entry + deleted entry
- *fls* output:
	``` sh
	$ fls -f linux-ext3 –a ext3.dd 69457
	d/d 69457: .
	d/d 53248: ..
	r/r 69458: abcdefg.txt
	r/r * 69459: file two.dat
	d/d 69460: subdir1
	r/r 69461: RSTUVWXY
	```

## 12. symbolic link
- if destination path is less than 60 characters, it is stored in 60 bytes of inode for block pointers
- if path is more than 60 characters, a block is allocated
- *icat* output:
	``` sh
	$ icat -f linux-ext3 ext3-3.dd 26
	/dir1/dir2/dir3/dir4/dir5/dir6/dir7/dir8/dir9/dir10/dir11/dir12/dir13/dir14/dir15/
	file1.txt
	```
	- *icat* processes content as normal file

## 13. hash trees
- used to sort directory entries
- each block in directory corresponds to node in tree
- non-leaf nodes have *node descriptors* that point to next layer
- hash index descriptor header DS follows .. directory entry:

	| Byte Range | Description | Essential | 
	| --- | --- | --- | 
	| 0–3 | Unused | No | 
	| 4–4 | Hash version | Yes | 
	| 5–5 | Length of this structure | Yes | 
	| 6–6 | Levels of leaves | No | 
	| 7–7 | Unused | No | 
- node descriptor list follows header:

	| Byte Range | Description | Essential |
	| --- | --- | --- |
	| 0-3 | Minimum hash value in node | Yes |
	| 4–7 | Block address | Yes |
	- entry contains smallest hash value and directory block of node
	- first node descriptor minimum values are used for something else:

		| Byte Range | Description | Essential | 
		| --- | --- | --- | 
		| 0–1 | Maximum number of node descriptors | No | 
		| 2–3 | Current number of node descriptors | Yes | 
		| 4–7 | Block address of first node | Yes | 
- remainder of block after last node descriptor typically contains data from previous directory entries

## 14. hash tree examples
- *icat* output:
	``` sh
	$ icat -f linux-ext3 ext3-3.dd 16
	0000000: 1000 0000 0c00 0100 2e00 0000 0200 0000 ................
	0000016: f403 0200 2e2e 0000 0000 0000 0208 0000 ................
	0000032: 7c00 0400 0100 0000 3295 6541 0400 0000 |.......2.eA....
	0000048: 88d5 fa92 0200 0000 86e7 50be 0300 0000 ..........P.....
	0000064: 3738 3930 2e31 3233 3400 0000 1200 0000 7890.1234.......
	```
	- bytes 0~9: . directory entry
	- bytes 12~23: .. directory entry; record length is 1,012 bytes
	- byte 24~27: first bytes of hash index header
	- byte 28: hash version 2 being used
	- byte 29: structure is 8 bytes long
	- bytes 32~39: first node descriptor:
		- bytes 32~33: can have 124 descriptors in block
		- bytes 34~34: 4 descriptors are used
		- bytes 36~39: block 1 of directory contains first node
	- bytes 40~47: second node descriptor:
		- bytes 40~43: minimum hash value 0x4165 9632
		- bytes 44~47: block 4 contains second node
	- bytes 48~55: third node descriptor:
		- bytes 48~51: minimum hash value 0x92fa d588
		- bytes 52~55: block 2 contains third node
	- bytes 56~63: fourth node descriptor:
		- bytes 56~59: minimum hash value 0xbe50 e786
		- bytes 60~63: block 3 contains fourth node

## 15. journal data structures
- 4 DS in Ext3 journal:
	- journal superblock
	- descriptor block
	- commit block
	- revoke block
- journal superblock located in first block of journal
- journal data is written in big-endian 
- common header:

	| Byte Range | Description | Essential |
	| --- | --- | --- |
	| 0–3 | Signature(0xC03B3998) | Yes |
	| 4–7 | Block type | Yes |
	| 8–11 | Sequence Number | Yes |
	- block types:

		| Value | Description |
		| --- | --- |
		| 1 | Descriptor block |
		| 2 | Commit block |
		| 3 | Superblock version 1 |
		| 4 | Superblock version 2 |
		| 5 | Revoke block |
- superblock header:

	| Byte Range | Description | Essential |
	| --- | --- | --- |
	| 0–11 | Standard header | Yes |
	| 12–15 | Journal block size | Yes |
	| 16–19 | Number of journal blocks | Yes |
	| 20–23 | Journal block where the journal actually starts | Yes |
	| 24–27 | Sequence number of first transaction | Yes |
	| 28–31 | Journal block of first transaction | Yes |
	| 32–35 | Error number | No |

- superblock header for version 2:

	| Byte Range | Description | Essential |
	| --- | --- | --- |
	| 36–39 | Compatible Features | No |
	| 40–43 | Incompatible Features | No |
	| 44–47 | Read only compatible Features | No |
	| 48–63 | Journal UUID | No |
	| 64–67 | Number of file systems using journal | No |
	| 68–71 | Location of superblock copy | No |
	| 72–75 | Max journal blocks per transaction | No |
	| 76–79 | Max file system blocks per transaction | No |
	| 80–255 | Unused | No |
	| 256–1023 | 16-byte IDs of file systems using the journal | No |

- descriptor block contains standard header then entries
- descriptor entries:

	| Byte Range | Description | Essential |
	| --- | --- | --- |
	| 0–3 | File system block | Yes |
	| 4–7 | Entry flags | Yes |
	| 8–23 | UUID(does not exist if the SAME_UUID flag is set) | No |
	- entries identify which file system block a journal block corresponds to; e.g. first journal block after descriptor block corresponds to FS block in first descriptor entry
	- flags:

		| Flag Values | Description |
		| --- | --- |
		| 0x01 | Journal block has been escaped |
		| 0x02 | Entry has the same UUID as the previous(SAME_UUID) |
		| 0x04 | Block was deleted by this transaction(currently not used) |
		| 0x08 | Last entry in descriptor block |
		- escape flag used when FS block has same 4 bytes as signature value in header; the 4 bytes are cleared when written to journal

- commit block has only standard header
- revoke block has standard header and list of FS blocks that were revoked:

	| Byte Range | Description | Essential |
	| --- | --- | --- |
	| 0–11 | Standard header | Yes |
	| 12–15 | Size in bytes of revoke data | Yes |
	| 16–SIZE | List of 4-byte file system block addresses | Yes |
	- revoke applies to transactions whose seq num is equal to or less than seq num of record

## 16. journal DS example
- *icat* output:
	``` sh
	$ icat –f linux-ext3 /dev/hdb2 8 | xxd
	0000000: c03b 3998 0000 0004 0000 0000 0000 0400 .;9.............
	0000016: 0000 0400 0000 0001 0000 0126 0000 0000 ...........&....
	0000032: 0000 0000 0000 0000 0000 0000 0000 0000 ................
	0000048: a34c 4be5 c222 460b b76f d45b 518b 083c .LK.."F..o.[Q..<
	0000064: 0000 0001 0000 0000 0000 0000 0000 0000 ................
	0000080: 0000 0000 0000 0000 0000 0000 0000 0000 ................
	```
	- bytes 0~3: signature
	- bytes 4~7: block type version 2 superblock
	- bytes 8~11: seq 0
	- bytes 12~15: journal block size 1,024
	- bytes 16~19: 1,024 blocks in journal
	- bytes 20~23: journal entries start in block 1
	- bytes 24~27: transaction 1 seq num 294
	- bytes 28~31: transaction 1 in journal block 0; FS was cleanly unmounted, all transaction complete
- *icat* output after creating file in root:
	``` sh
	$ icat –f linux-ext3 /dev/hdb2 8 | xxd
	0000000: c03b 3998 0000 0004 0000 0000 0000 0400 .;9.............
	0000016: 0000 0400 0000 0001 0000 0127 0000 0001 ..........."....
	```
	- bytes 24~27: transaction 1 seq num 295
	- bytes 28~31: transaction 1 in journal block 1; valid transaction in journal
- *jcat* of journal block 1:
	``` sh
	$ jcat –f linux-ext3 /dev/hdb2 1 | xxd
	0000000: c03b 3998 0000 0001 0000 0127 0000 0004 .;9........"....
	0000016: 0000 0000 0000 0000 0000 0000 0000 0000 ................
	0000032: 0000 0000 0000 0002 0000 0002 0000 000e ................
	0000048: 0000 0002 0000 0005 0000 0002 0000 00a3 ................
	0000064: 0000 0002 0000 0003 0000 000a 0000 0000 ................
	```
	- bytes 4~7: descriptor block type
	- bytes 8~11: seq num 295
	- bytes 12~35: first descriptor entry:
		- bytes 12~15: for FS block 4
		- bytes 16~19: no entry flags
		- bytes 20~35: UUID
		- first journal block after descriptor block corresponds to FS block 4, inode bitmap
	- bytes 36~43: second descriptor entry:
		- bytes 36~39: for FS block 2
		- bytes 40~43: entry flag 2; same UUID
		- group descriptor table changed
	- bytes 44~51: third descriptor entry:
		- bytes 44~47: for FS block 14
		- bytes 48~51: entry flag 2; same UUID
		- inode table containing inode for new file changed
	- bytes 52~59: fourth descriptor entry:
		- bytes 52~55: for FS block 5
		- bytes 55~59; entry flag 2; same UUID
		- inode for root directory changed
	- bytes 60~67: fifth descriptor entry:
		- bytes 60~63: for FS block 163
		- bytes 64~67: entry flag 2; same UUID
		- root directory's directory entries changed
	- bytes 68~75: sixth descriptor entry:
		- bytes 68~71: for FS block 3
		- bytes 71~75: entry flag 10; same UUID, last entry
		- block bitmap changed
- *jcat* of journal block 8 for commit block:
	``` sh
	$ jcat –f linux-ext3 /dev/hdb2 8 | xxd
	0000000: c03b 3998 0000 0002 0000 0127 0000 0000 .;9........'....
	0000016: 0000 0000 0000 0000 0000 0000 0000 0000 ................
	```
	- bytes 4~7: type commit block
	- bytes 8~11: seq num 295
- *jcat* of journal block 6 for root directory:
	``` sh
	$ jcat –f linux-ext3 /dev/hdb2 6 | xxd
	0000000: 0200 0000 0c00 0100 2e00 0000 0200 0000 ................
	0000016: 0c00 0200 2e2e 0000 0b00 0000 e803 0c00 ................
	0000032: 6e65 772d 6669 6c65 2e74 7874 0c00 0000 new-file.txt....
	[REMOVED]
	```
	- can determine which files were recently created/deleted
- *jls* output:
	``` sh
	$ jls –f linux-ext3 /dev/hdb2
	JBlk Descriptrion
	0: Superblock (seq: 0)
	1: Allocated Descriptor Block (seq: 295)
	2: Allocated FS Block 4
	3: Allocated FS Block 2
	4: Allocated FS Block 14
	5: Allocated FS Block 5
	6: Allocated FS Block 163
	7: Allocated FS Block 3
	8: Allocated Commit Block (seq: 295)
	9: Unallocated FS Block Unknown
	[REMOVED]
	```
