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
- first 3 bytes of boot sector contain jump instruction to boot code
- bytes 62(90)~509 contain boot code for FAT12/16(32)
- even non-bootable FAT FS can have boot code
- FAT boot code is called from boot code in MBR of disk, and locates/loads appropriate OS files

## 6. FAT32 fsstat output
``` sh 
$ fsstat –f fat fat-4.dd
FILE SYSTEM INFORMATION
--------------------------------------------
File System Type: FAT
OEM Name: MSDOS5.0
Volume ID: 0x4c194603
Volume Label (Boot Sector): NO NAME
Volume Label (Root Directory): FAT DISK
File System Type Label: FAT32
Backup Boot Sector Location: 6
FS Info Sector Location: 1
Next Free Sector (FS Info): 1778
Free Sector Count (FS Info): 203836
Sectors before file system: 100800
File System Layout (in sectors)
Total Range: 0 - 205631
* Reserved: 0 – 37
** Boot Sector: 0
** FS Info Sector: 1
** Backup Boot Sector: 6
* FAT 0: 38 - 834
* FAT 1: 835 - 1631
* Data Area: 1632 - 205631
** Cluster Area: 1632 - 205631
*** Root Directory: 1632 - 1635
CONTENT-DATA INFORMATION
--------------------------------------------
Sector Size: 512
Cluster Size: 1024
Total Cluster Range: 2 – 102001
[REMOVED]
```
- 38 reserved sectors until first FAT
- backup boot sector and FSINFO DS in reserved area

## 7. FAT FS analysis techniques
- purpose of analyzing FS category data is to determine FS layout and config details to specific analysis techniques can be applied
- to determine config of FAT FS, locate and process boot sector
- using info from boot sector calculate locations of reserved, FAT, and data areas
- FAT32 FSINFO DS can provide clues about recent activity

## 8. FAT FS analysis considerations
- OEM label and volume label may give clues
- areas not used by FS can contain data hidden by user:
	- between end of boot sector data and final signature 
	- unused reserved areas
	- FAT32 FSINFO DS unused areas
	- between end of FS and end of volume(volume slack)
- primary and backup sectors should be compared for inconsistencies

## 9. FAT FS analysis scenario for damaged partition table
- search for signature values 0x55 and 0xAA in final 2 bytes of boot sector:
``` sh
$ sigfind –o 510 55AA disk-9.dd
Block size: 512 Offset: 510
Block: 63 (-)
Block: 64 (+1)
Block: 65 (+1)
Block: 69 (+4)
Block: 70 (+1)
Block: 71 (+1)
Block: 75 (+4)
Block: 128504 (+128429)
Block: 293258 (+164754)
[REMOVED]
```
- read first sector found and apply boot sector DS:
	- backup boot sector in sector 6 of FS
	- FSINFO in sector 1 of FS
	- 20,482,812 sectors in FS
- sector 64 contains FSINFO which has same signature as boot sector
- sector 69 and 70 contain backups for boot sector and FSINFO
- sector 65 and 71 are all zeroes except for their signatures
- sector 128,504 is false positive
- can assume FAT FS from disk sector 63 to 20,482,874

``` sh
[REMOVED]
Block: 20112453 (+27031)
Block: 20482875 (+370422)
Block: 20482938 (+63)
Block: 20482939 (+1)
Block: 20482940 (+1)
Block: 20482944 (+4)
Block: 20482945 (+1)
Block: 20482946 (+1)
Block: 20482950 (+4)
Block: 20513168 (+30218)
```
- sector 20,482,875:
	- after final sector of previous FS
	- sequence of hits is different
	``` sh
	$ dd if=disk-9.dd bs=512 skip=20482875 count=1 | xxd
	0000000: 088c 039a 5f78 7694 8f45 bf49 e396 00c0 ...._xv..E.I....
	0000016: 889d ddc0 6d36 60df 485d adf7 46d1 3224 ....m6`.H]..F.2$
	0000032: 3829 95cd ad28 d2a2 dc89 f357 d921 cfde 8)...(.....W.!..
	0000048: df8e 1fd3 303e 8619 641e 9c2f 95b4 d836 ....0>..d../...6
	[REMOVED]
	0000416: 3607 e7be 1177 db5f 11c9 fba1 c913 1a3d 6....w._.......=
	0000432: da81 143d 00c7 7083 9d42 330c 0287 0001 ...=..p..B3.....
	0000448: c1ff 0bfe ffff 3f00 0000 fc8a 3801 0000 ......?.....8...
	0000464: c1ff 05fe ffff 3b8b 3801 7616 7102 0000 ......;.8.v.q...
	0000480: 0000 0000 0000 0000 0000 0000 0000 0000 ................
	0000496: 0000 0000 0000 0000 0000 0000 0000 55aa ..............U.
	```
	- contains an extended partition table starting at byte 446
	- entries show there is FAT32 partition from sector 20,482,938 to 40,965,749 and extended partition from sector 40,965,750 to 81,931,499
	- confirms *sigfind* output

![[FAT boot sector signature matches.png]]

## 10. FAT content category
- FAT uses term *cluster* for data units:
	- group of consecutive sectors
	- number of sectors must be power of 2
	- max cluster size is 32KB
	- address of first cluster is 2; not every logical volume address has a logical FS address
	- located in data area region of FS

## 11. finding first cluster
![[FAT first cluster.png]]
- cluster 2 in FAT12/16 FS starts after root directory
- cluster 2 in FAT32 FS starts with first sector of data area

## 12. cluster and sector addresses
- to convert between cluster and sector addresses get:
	- sector address of cluster 2
	-  \# of sectors in cluster
S = (C-2) * (# of sectors in cluster) + (sector address of cluster 2)
C = ((S - sector address of cluster 2) / (# of sectors in cluster)) + 2

## 13. cluster allocation status
- allocation status of a cluster is determined using FAT structure:
	- entry of 0 is unallocated
	- 0x0ff7 for FAT12, 0xfff7 for FAT16, 0x0fff fff7 for FAT32 is damaged
	- all other values mean allocated

## 14. allocation algorithms
- Windows 98 and XP seem to use next available algorithm 
- cluster contents are usually not wiped when unallocated

## 15. FAT content analysis techniques
- performed to locate specific data unit, determine alocation status 
- locating specific data unit in FAT is more complex that other FS
- data units prior to data area are not listed in FAT; do not have allocation state
- tools must be tested to see if they consider data prior to data area unallocated

## 16. FAT content analysis considerations 
- clusters marked as bad must be examined; bad sectors are usually handled at hardware level, abstracted away from OS
- there could be a few sectors at end of data area that are not part of a cluster
unused sectors = (total number of sectors - sector address of cluster 2)/(number of sectors per cluster)
- between end of last valid entry in FAT and start of backup copy or last entry in bakcup FAT and start of data area
(# of unused table entries) = (# of sectors allocated to FAT) * (sector size / entry size) - (# of clusters)
14 = 797 sectors * 128 entries per sector - 102,002 clusters
- logical volume search and logical FS search may have different results
- test tools to see if they search boot sector and FAT areas

## 17. FAT content analysis scenario for locating by cluster number
- process boot sector:
	- 6 reserved sectors
	- 2 FATs
	- 249 sectors per FAT
	- 32 sectors per cluster
	- 512 directory entries in root directory
- first FAT starts in sector 6 ends in sector 254
- second FAT starts in sector 255 ends in sector 503
- root directory needs 32 bytes for each entry; starts in sector 504 ends in sector 535
- data area begins in sector 536
- cluster 812 is 810th cluster; cluster 812 is 25,920 sectors from start of data area
- cluster 812 starts in sector 26,456 ends in sector 26,487

![[FAT content analysis scenario.png]]

## 18. FAT metadata category
- used to obtain additional information about a file or identify suspect files
- in FAT FS this is stored in a directory entry structure
- FAT structure is also used to store metadata information about layout of a file or directory

## 19. directory entries
- DS allocated for every file and directory
- 32 bytes in size
- contains:
	- file name using 8.3 naming convention (8 characters for name and 3 for extension)
	- file attributes
	- size
	- starting cluster
	- date/time
- plays a role in both metadata category and file name category
- can exist anywhere in data area
- list of directory entries make up a directory's contents
- not given unique numerical addresses; addressed with full name of file or directory that allocated it
- each directory entry has 3 times: created, last accessed, and last written
- allocation status of directory entry is determined by using first byte; first character in file name(allocated) and 0xe5(unallocated)

## 20. directory entry attributes field
- essential attributes:
	- *directory*:
		- used to identify the directory entries that are for other directories
	- *long file name*:
		- identifies special type of entry that has a different layout
	- *volume label*:
		- only one directory entry should have this attribute set
- non-essential attributes:
	- *read only*:
		- prevents a file from being written to
		- directories with this set can still have new files created in them (Windows XP and 98)
	- *hidden*:
		- may cause files and directories not to be listed
		- usually there is a setting for the OS to display them
	- *system*: 
		- should identify a file as system file
	- *archive*:
		- typically set when a file is created or written to

## 21. cluster chains
- non-zero entries in FAT contain address of the next cluster in the file, an *end of file(EOF)* marker, or value to show cluster has bad sectors

![[cluster chain.png]]
- the maximum size of a FAT FS is based on the size of FAT entries

## 22. directories
- when a new directory is created, a cluster is allocated and wiped with 0s
- size field should always be 0
- cluster chain should be used to determine the size
- the first 2 directory entries are . and .. directories:
	- written, accessed, and created times seem to reflect the time the directory was created

## 23. directory entry address
- the first letter of the name is deleted when directory entries are unallocated; potential unique naming conflict
- when a directory is deleted, there is no pointer to files and directories in the deleted directory; creating files with no addresses, *orphan files*

![[orphan files.png]]

- to find orphan files:
	- examine first 32-bytes of a sector; if it passes a sanity check, process rest of sector
	- examine first 32-bytes of a cluster for . and .. entries; only finds first clusters of directories
- using a different addressing method solves these problems:
	- root directory has address of 2
	- each following sector has a numerical address starting with 3

## 24. FAT example image
``` sh
$ istat -f fat fat-4.dd 4
Directory Entry: 4
Allocated
File Attributes: File, Archive
Size: 8689
Name: RESUME-1.RTF
Directory Entry Times:
Written: Wed Mar 24 06:26:20 2004
Accessed: Thu Apr 8 00:00:00 2004
Created: Tue Feb 10 15:49:40 2004
Sectors:
1646 1647 1648 1649 1650 1651 1652 1653
1654 1655 1656 1657 1658 1659 1660 1661
1662 1663
```

## 25. metadata category allocation algorithms
- directory entries need to be allocated for new files and directories:
	- Windows 98 uses first-available allocation
	- Windows XP uses next-available allocation
	- a new cluster is allocated for Windows 98 and XP when no unallocated directory entries are found
	- when a file is renamed, a new entry is made for the new name, unallocating the old one
	- for Windows XP, when a file is created from a Windows application, two entries are created; first entry without size and starting cluster which is unallocated for the second entry with both
	- when a file is deleted, the fist byte of the entry is set to 0xe5; allocated clusters are marked unallocated in the FAT
- temporal information must be updated:
	- time values are non-essential and can be false
	- created time is set when Windows allocates a new directory entry for a new file; no changes for renamed/moved files
	- copying files changes creation time for copy
	- written time is set when Windows writes new file content
	- write time is content-based and not directory entry-based; follows data when copied/moved
	- changing attributes or name of file does not change write time
	- last accessed date is updated when file is opened
	- moving to a different volume changes last accessed date; moving to same volume does not
	- moving/copying changes access date for both source and destination file
	- dates for directories are set when creating but not updated much after
- allocation behavior is OS-depedent

## 26. FAT metadata analysis techniques
- used to determine more details about a file/directory
- must locate specific directory entry, process contents, and determine where content is stored

## 27. FAT metadata analysis considerations
- times are stored without respect to timezone
- last accessed and created dates are optional by specification and may be 0
- written date being earlier than created dates are common for copied files
- next available allocation strategy for new directory entries in Windows XP allows for longer acces to names of deleted files; not necessarily the content
- DEFRAG utility in Windows compacts directories, removes unused directory entries, and moves file content to make it contiguous; analysis becomes more difficult
- unused sectors in the last cluster of a file allocated by Windows typically contain deleted data
- Windows doesn't show data after a directory entry with all zeroes; easy to hide data:
	- allocated size of directory should be compared to number of allocated files
	- logical file system search should find any data hidden
- volume label directory entry:
	- access and created times are often set to 0, but last written time is often file system creation time
	- with Windows XP can be used to hide data by manually starting cluster chain
	- with Windows XP it is possible to many volume label directory entries in any location

## 28. FAT metadata analysis scenarios
- file system creation date:
	- view volume label directory entry written time
- searching for deleted directories:
	- extract unallocated space 
	``` sh
	$ dls -f fat fat-10.dd > fat-10.dls
	```
	- search for signature for . directory
	``` sh
	$ sigfind –b 512 2e202020 fat-10.dls
	Block size: 512 Offset: 0
	Block: 180 (-)
	Block: 2004 (+1824)
	Block: 3092 (+1088)
	Block: 3188 (+96)
	Block: 19028 (+15840)
	[REMOVED]
	```
	- view sector 180
	``` sh
	$ dd if=fat-10.dls skip=180 count=1 | xxd
	0000000: 2e20 2020 2020 2020 2020 2010 0037 5daf . ..7].
	0000016: 3c23 3c23 0000 5daf 3c23 4f19 0000 0000 <#<#..].<#0.....
	0000032: 2e2e 2020 2020 2020 2020 2010 0037 5daf .. ..7].
	0000048: 3c23 3c23 0000 5daf 3c23 dc0d 0000 0000 <#<#..].<#......
	0000064: e549 4c45 312e 4441 5420 2020 0000 0000 .ILE1.DAT ....
	0000080: 7521 7521 0000 0000 7521 5619 00d0 0000 u!u!....u!V.....
	[REMOVED]
	```
	- . entry points to cluster 6,479(0x194f)
	- .. entry points to cluster 3,548(0x0ddc)
	- third entry is for file starting in cluster 6,486(0x1956) with a size of 53,248(0xd000) bytes

![[FAT find deleted files.png]]

## 29. FAT file name category
- typically allows mapping of file name to metadata, but not for FAT

## 30. long file name directory entry
- if a file name is longer than 8 character or has special name values, a *long file name(LFN)* type of directory entry is added
- LFN directory entries also have a *shoft file name(SFN)* entry; needed for storing time, size, and starting cluster information
- LFN entries use a special attribute value; remaining bytes are used to store 13 Unicode characters encoded in UTF-16; additional LFN entries are used if name need more than 13 characters
- all LFN entries precede SFN entry and contain checksum for correlation with SFN entry; LFN entries are in reverse order with earlier parts closer to SFN entry

![[directory entry with LFN.png]]
``` sh 
$ fls -f fat fat-2.dd
r/r 3: FAT DISK (Volume Label Entry)
r/r 4: RESUME-1.RTF
r/r 7: My Long File Name.rtf (MYLONG~1.RTF)
r/r * 8: _ile6.txt
```
- address gap between 4 and 7 due to LFN entries

## 31. FAT file name allocation algorithms
- allocation algorithms is same as for metadata category; except for LFN entries that must come before SFN entry
- deletion routine is same as metadata category; 0xe5 overwrites sequence number for LFN entries

## 32. FAT file name analysis techniques
- analysis used to find specific file name
- locate root directory:
	- FAT12/16: starts in sector following FAT area, size given in boot sector
	- FAT32: starting cluster address given in boot sector, FAT used to determine layout
- process contents of directory:
	- step through each 32-byte directory entry
- find corresponding metadata

## 33. FAT file name analysis considerations
- scandisk tool for FAT is not exhaustive; small amounts of data can be hidden
- attacker may hide data at end of directories
- use Unicode to search for file names
- file names may not be stored sequentially 

## 34. FAT file name analysis scenarios
- long file name searching:
	- each long name entries are broken up into chunks of 5, 6, and 2
	- search for longest string

## 35. the big picture
- file allocation example:
	1. read boot sector from sector 0 of volume 
	2. locate FAT structures, data area, and root directory
	3. process each directory entry to find directory for file, dir1
	4. search directory entries in dir1's starting cluster to find unallocated entry
	5. set allocation status by writing name, file1.txt, and write size and current time to appropriate fields
	6. search FAT structure to allocate cluster to file and set entry to EOF
	7. write address of first cluster to directory entry 
	8. if more clusters are needed, repeat 6 and update previous cluster entry to contain next cluster address

	![[file allocation summary.png]]
- file deletion example:
	1. read boot sector from sector 0 of volume
	2. locate FAT structures, data area, and root directory
	3. process each directory entry to find dir 1
	4. search directory entries in dir1's starting cluster for file1.txt
	5. use FAT to determine cluster chain
	6. set FAT entries in chain to 0
	7. unallocate directory entry for file by setting first byte of file name to 0xe5

	![[file deletion summary.png]]

## 36. other topics
- file recovery:
	- (a) blindly read clusters
	- (b) read only unallocated clusters

	![[file recovery options.png]]
	- both options will recover correctly for (A)
	- only (b) will recover correctly for (B)
	- neither will recover correctly for (C) 
	- multiple cluster directories are likely to be fragmented
- determining type:
	- calculate # of sectors in root directory (0 for FAT32)
	(root directory size in sectors) = ceil((# of entries) * 32) / (bytes per sector))
	- calculate # of sectors allocated to clusters
	(total cluster size in sectors) = (total # of sectors) - (reserved area size in sectors) - ((# of FAT) * (FAT size in sectors)) - (root directory size in sectors)
	- caluculate # of clusters
	(# of clusters) = (total cluster size in sectors) / (# of sectors in cluster)
	- FAT12: ~ 4,084
	- FAT16: 4,085 ~ 65,524
	- FAT32: 65,525 ~
- consistency check:
	- verify values are in appropriate range for boot sector and other DS in reserved area of FAT FS; also examine unused locations for non-zero values
	- compare backup and primary DS for inconsistencies
	- examine sectors marked as bad
	- space between FAT entry for last cluster and end of FAT sector should be examined
	- space between end of last cluster and end of file system
	- examine cluster chains to make sure allocated directory entry points to the starting cluster
	- examine allocated directory entries to make sure they point to allocated clusters
	- directory entry marked as volume label should not have a starting cluster
	- there should only be one directory entry marked as volume label
	- checksums for allocated LFN entries should be compared with allocated SFN entries
	- check for directory entries after a null entry

## 37. summary
- FAT FS have relatively few DS, but present challenges with respect to file recovery and temporal-based auditing
