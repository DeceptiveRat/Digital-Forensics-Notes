## 1. FS category
- NTFS stores these data in FS metadata files
- FS metadata files have date/time stamps associated with them; seem to be set to time FS was created

## 2. $MFT file overview
- contains MFT
- MFT layout determined by processing entry 0 of MFT
- $DATA contains clusters used by MFT
- $BITMAP manages allocation status of MFT entries
- has standard $FILE_NAME and $STANDARD_INFORMATION
- starts small and grows as more files/directories are created
- can become fragmented
- *istat* output
	``` sh
	$ istat –f ntfs ntfs1.dd 0
	[REMOVED]
	$STANDARD_INFORMATION Attribute Values:
	Flags: Hidden, System
	Owner ID: 0 Security ID: 256
	Created: Thu Jun 26 10:17:57 2003
	File Modified: Thu Jun 26 10:17:57 2003
	MFT Modified: Thu Jun 26 10:17:57 2003
	Accessed: Thu Jun 26 10:17:57 2003
	[REMOVED]
	Attributes:
	Type: $STANDARD_INFORMATION (16-0) Name: N/A Resident size: 72
	Type: $FILE_NAME (48-3) Name: N/A Resident size: 74
	Type: $DATA (128-1) Name: $Data Non-Resident size: 8634368
	342709 342710 342711 342712 342713 342714 342715 342716
	342717 342718 342719 342720 342721 342722 342723 342724
	[REMOVED]
	443956 443957 443958 443959 443960 443961 443962 443963
	Type: $BITMAP (176-5) Name: N/A Non-Resident size: 1056
	342708 414477 414478 414479
	```
	- temporal data is typically date FS was created 
	- $STANDARD_INFORMATION and $FILE_NAME exist
	- 8MB $DATA
	- $BITMAP

## 3. $MFTMirr file overview
- MFT entry 1
- has non-resident attribute that contains backup copy of first MFT entries
- $DATA allocates clusters in middle of FS and saves copies of at least first 4 MFT entries:
	- $MFT
	- $MFTMirr
	- $LogFile
	- $Volume
- 4 backup entries allow recovery tool to determine MFT layout and size, location of $LogFile for FS recovery, and version and status information from $Volume attributes
- *istat* output
	``` sh
	$ istat –f ntfs ntfs1.dd 1
	[REMOVED]
	Attributes:
	Type: $STANDARD_INFORMATION (16-0) Name: N/A Resident size: 72
	Type: $FILE_NAME (48-2) Name: N/A Resident size: 82
	Type: $DATA (128-1) Name: $Data Non-Resident size: 4096
	514064 514065 514066 514067
	```
	- 1,028,128 clusters
	- $DATA starts in middle cluster
	- temporal data had same values as $MFT

## 4. $Boot file overview
- FS metadata file located in MFT entry 7
- contains boot sector of FS
- only FS metadata file with static location
- $DATA is always located in first sector of FS
- MS typically allocates 16 sectors of the FS, but only first half has non-zero value
- boot sector:
	- similar to FAT boot sector, shares many fields
	- gives basic information:
		- size of each cluster
		- number of sectors in FS
		- starting cluster address of MFT
		- size of each MFT entry
	- gives serial number for FS
- $DATA contains boot sector and then boot code
- backup copy of boot sector exists in either last sector or middle of volume
- total number of sectors in FS is less than in volume; backup copy not necessarily alloated to specific file
- *istat* output:
	``` sh
	$ istat –f ntfs ntfs1.dd 7
	[REMOVED]
	Attributes:
	Type: $STANDARD_INFORMATION (16-0) Name: N/A Resident size: 48
	Type: $FILE_NAME (48-2) Name: N/A Resident size: 76
	Type: $SECURITY_DESCRIPTOR (80-3) Name: N/A Resident size: 104
	Type: $DATA (128-1) Name: $Data Non-Resident size: 8192
	0 1 2 3 4 5 6 7
	```
	- has 4 attributes
	- $DATA is 8KB in size
	- $DATA allocated clusters 0~7
	- temporal data same as $MFT and $MFTMirr
	
## 5. $Volume file overview
- located in MFT entry 3
- contains volume label and other version information
- has 2 unique attributes:
	- $VOLUME_NAME contains unicode name of volume
	- $VOLUME_INFORMATION contains NTFS version and dirty status
- typically has $DATA, but found to be 0 bytes
- *istat* output:
	``` sh
	$ istat -f ntfs ntfs1.dd 3
	[REMOVED]
	Attributes:
	Type: $STANDARD_INFORMATION (16-0) Name: N/A Resident size: 48
	Type: $FILE_NAME (48-1) Name: N/A Resident size: 80
	Type: $OBJECT_ID (64-6) Name: N/A Resident size: 16
	Type: $SECURITY_DESCRIPTOR (80-2) Name: N/A Resident size: 104
	Type: $VOLUME_NAME (96-4) Name: N/A Resident size: 22
	Type: $VOLUME_INFORMATION (112-5) Name: N/A Resident size: 12
	Type: $DATA (128-3) Name: $Data Resident size: 0
	```
	- $DATA has size of 0
	- temporal data same as $MFT

## 6. $AttrDef file overview
- MFT entry 4
- $DATA defines names and type identifiers for each type of attribute
- circular logic issue; reading $DATA of $AddrDef to learn type identifier of $DATA
- allows:
	- each FS to have unique attributes for its files
	- each FS to redefine identifiers for standard attributes
- *istat* output:
	``` sh
	$ istat -f ntfs ntfs1.dd 4
	[REMOVED]
	Attributes:
	Type: $STANDARD_INFORMATION (16-0) Name: N/A Resident size: 48
	Type: $FILE_NAME (48-2) Name: N/A Resident size: 82
	Type: $SECURITY_DESCRIPTOR (80-3) Name: N/A Resident size: 104
	Type: $DATA (128-4) Name: $Data Non-Resident size: 2560
	342701 342702 342703
	```
	- $DATA is over 2KB
	- temporal data same as $MFT

## 7. FS category example image
``` sh
$ fsstat –f ntfs ntfs1.dd
FILE SYSTEM INFORMATION
--------------------------------------------
File System Type: NTFS
Volume Serial Number: 0450228450227C94
OEM Name: NTFS
Volume Name: NTFS Disk 2
Version: Windows XP
META-DATA INFORMATION
--------------------------------------------
First Cluster of MFT: 342709
First Cluster of MFT Mirror: 514064
Size of MFT Entries: 1024 bytes
Size of Index Records: 4096 bytes
Range: 0 - 8431
Root Directory: 5
CONTENT-DATA INFORMATION
--------------------------------------------
Sector Size: 512
Cluster Size: 1024
Total Cluster Range: 0 - 1028127
Total Sector Range: 0 - 2056255
$AttrDef Attribute Values:
$STANDARD_INFORMATION (16) Size: 48-72 Flags: Resident
$ATTRIBUTE_LIST (32) Size: No Limit Flags: Non-resident
$FILE_NAME (48) Size: 68-578 Flags: Resident,Index
$OBJECT_ID (64) Size: 0-256 Flags: Resident
$SECURITY_DESCRIPTOR (80) Size: No Limit Flags: Non-resident
$VOLUME_NAME (96) Size: 2-256 Flags: Resident
$VOLUME_INFORMATION (112) Size: 12-12 Flags: Resident
$DATA (128) Size: No Limit Flags:
$INDEX_ROOT (144) Size: No Limit Flags: Resident
$INDEX_ALLOCATION (160) Size: No Limit Flags: Non-resident
$BITMAP (176) Size: No Limit Flags: Non-resident
$REPARSE_POINT (192) Size: 0-16384 Flags: Non-resident
$EA_INFORMATION (208) Size: 8-8 Flags: Resident
$EA (224) Size: 0-65536 Flags:
$LOGGED_UTILITY_STREAM (256) Size: 0-65536 Flags: Non-resident
```

## 8. FS category analysis techniques
1. process boot sector
2. identify starting location of MFT, size of MFT entry
3. process first entry in MFT
	- use $MFTMirr stored in middle of FS for backup if corrupt
4. identify MFT location
5. locate and process $Volume and $AttrDef

## 9. FS category analysis considerations
- MS states some values not used in boot sector must be 0; XP will not mount if non-zero
- small amounts of data can be hidden in boot sector
- more space available at end of $Boot file after boot code
- number of sectors should be compared with size of volume to determine volume slack of unused space after FS
- temporal data typically correspond to when FS was created
- $MFT file can be any size and will not decrease in size from within Windows; could be used to hide data
- possible to allocate additional attributes to files to hide data

## 10. FS category analysis scenario
1. search for signature value in boot sector
	- hope to find first sector of NTFS FS or sector after end of FS
	``` sh 
	$ sigfind –o 510 -1 AA55 disk5.dd
	Block size: 512 Offset: 510
	Block: 210809 (-)
	Block: 210810 (+1)
	Block: 210811 (+1)
	Block: 210812 (+1)
	Block: 210813 (+1)
	Block: 210814 (+1)
	Block: 210815 (+1)
	Block: 210816 (+1)
	Block: 318170 (+107354)
	Block: 339533 (+21363)
	Block: 718513 (+378980)
	[REMOVED]
	```
	- not expected from boot sector or partition tables
2. look at copy to check for false hit
	``` sh
	$ dd if=disk5.dd skip=210809 count=1 | xxd
	0000000: eb3c 904d 5357 494e 342e 3100 0208 0100 .<.MSWIN4.1.....
	0000016: 0200 0203 51f8 0800 1100 0400 0100 0000 ....Q...........
	0000032: 0000 0000 8000 2900 0000 004e 4f20 4e41 ......)....NO NA
	0000048: 4d45 2020 2020 4641 5431 3220 2020 33c9 ME FAT12 3.
	[REMOVED]
	```
	- looks like boot sector from FAT12 FS
3. check next sector
	``` sh
	$ dd if=disk5.dd skip=210810 count=1 | xxd
	0000000: eb58 904d 5357 494e 342e 3100 0202 0800 .X.MSWIN4.1.....
	0000016: 0100 0400 00f8 0000 1100 0400 0100 0000 ................
	0000032: 0000 2000 e01f 0000 0000 0000 0000 0000 .. .............
	0000048: 0100 0600 0000 0000 0000 0000 0000 0000 ................
	0000064: 8000 2900 0000 004e 4f20 4e41 4d45 2020 ..)....NO NAME
	0000080: 2020 4641 5433 3220 2020 33c9 8ed1 bcf4 FAT32 3.....
	[REMOVED]
	```
	- basic template for FAT32 FS boot sector
4. examine sector 210,811
	- hit at 210,811 is basic template for FSINFO FAT with no values
5. search for string "NTFS" starting in byte 3 of sector
	``` sh
	$ sigfind -o 3 4e544653 disk5.dd
	```
	- sectors 1,075,545; 1,075,561; 2,056,319 occur in both results
6. examine sectors 1,075,545 and 1,075,561
	- value for total number of sectors is 0
	- not likely to be from real FS
7. examine sector 2,056,319
	- more non-zero values
	- cluster size of 1,024 bytes
	- \# of sectors 2,056,256
	- starting MFT cluster 342,709
	- scenarios:
		1. located in first sector of FS
		2. located following end of FS
	- reported # of sectors in FS is 63 smaller than address of boot sector; could be backup boot sector for FS starting at 63
8. look for MFT
	- cluster 342,709 is sector 685,481 of disk
	``` sh
	$ dd if=disk5.dd skip=685481 count=1 | xxd
	0000000: 4649 4c45 3000 0300 4ba7 6401 0000 0000 FILE0...K.d.....
	[REMOVED]
	```
9. move back 2 sectors to check if MFT entry
	``` sh
	$ dd if=disk5.dd skip=685479 count=1 | xxd
	0000000: ffff 00ff ffff ffff ffff ffff ffff ffff ................
	[REMOVED]
	```
	- doesn't start with FILE
10. copy boot sector backup to sector 63
11. create basic partition table in sector 0 with partition from 63~2,056,318 or extract 2,056,256 sectors of FS

## 11. content category
- cluster 0 starts with first sector of file
Sector = Cluster * sectors_per_cluster
- no strict layout requirements; any cluster can be allocated to any file

## 12. $Bitmap file overview
- MFT entry 6
- determines allocation status of clusters
- $DATA has one bit for every cluster in FS; 1 is allocated and 0 is not
- *istat* output:
	``` sh
	$ istat –f ntfs ntfs1.dd 6
	[REMOVED]
	Attributes:
	Type: $STANDARD_INFORMATION (16-0) Name: N/A Resident size: 72
	Type: $FILE_NAME (48-2) Name: N/A Resident size: 80
	Type: $DATA (128-1) Name: $Data Non-Resident size: 128520
	514113 514114 514115 514116 514117 514118 514119 514120
	514121 514122 514123 514124 514125 514126 514127 514128
	[REMOVED]
	```
	- temporal data same as $MFT

## 13. $BadClus file overview
- MFT entry 8
- damaged clusters in FS are allocated to $DATA
- $DATA is named $Bad and is sparse
- $Bad is reported to have size equal to total size of FS
- *istat* output:
	``` sh
	$ istat –f ntfs ntfs1.dd 8
	[REMOVED]
	Attributes:
	Type: $STANDARD_INFORMATION (16-0) Name: N/A Resident size: 72
	Type: $FILE_NAME (48-3) Name: N/A Resident size: 82
	Type: $DATA (128-2) Name: $Data Resident size: 0
	Type: $DATA (128-1) Name: $Bad Non-Resident size: 1052803072
	```
	- has 2 $DATA
	- $Bad shows no cluster addresses because FS has no bad sectors

## 14. allocation algorithms
- XP uses best-fit algorithm; data placed in location that will most effectively use available space

![[bestfit cluster allocation.png]]

## 15. FS layout
- *MFT zone* is a collection of consecutive clusters reserved for MFT unless disk is full; 12.5% of FS by default
- first clusters are allocated to $Boot 

![[Windows XP and 2000 FS layout.png]]

## 16. content category analysis techniques
1. locate specific cluster
2. determine allocation status by processing $Bitmap $DATA
3. process content

## 17. content category analysis considerations
- only 1 addressing scheme is needed
- should not be sectors at end of FS without cluster address; sectors after FS and before end of volume may have hidden data
- no difference between logical volume search and logical FS search
- check clusters marked as bad by FS

## 18. content category analysis scenario
objective: locate cluster 9,900,009
1. determine cluster size
	- process boot sector
	- each sector is 512 bytes
	- each cluster is 8 sectors
2. get cluster location
	- cluster 9,900,009 is 79,200,072 sectors
	- 79,200,072 sectors is 40,550,436,864 bytes
- much more simple that locating sector in FAT

## 19. metadata category
![[NTFS basic file with attributes.png]]

## 20. $STANDARD_INFORMATION
- exists for all files and directories
- may not exist for non-base MFT entries
- contains core metadata:
	- primary set of time/date stamps
	- ownership info
	- security info
	- quota info
- nothing inside is essential for storing files
- default type ID 16
- static size 72 bytes for Windows XP and 2000
- NTFS date/time stamps: 64-bit values that represent number of 100ns since Jan 1st, 1601 UTC
- contains 4 date/time stamps: 
	- creation time
	- modified time: content of $DATA or $INDEX modification time
	- MFT modified time: metadata of file modification time
	- accessed time
- contains flags for general properties of file; read-only, system or archive, compressed, sparse, encrypted
- in 3.0+ versions of NTFS (Windows 2000 and XP), contains 4 additional values for security information and application-level features:
	- owner identity:
		- corresponds to owner of file
		- index into quota data that stores how much data a user is using
	- how many bytes are added to user quota due to this file
	- SID value:
		- used as index for $Secure file 
		- determines access control rules that apply to file
- contains *Update Sequence Number(USN)* for last record that was created for this file if change journaling is used

## 21. $FILE_NAME
- every file/directory has at least one:
	- in MFT entry
	- in index of parent directory
- type ID 48
- variable length; 66 bytes + name length
- contains name of file in UTF-16 unicode
- name must be in specific name space; 8.3 DOS format, Win32 format, or POSIX
- some files have $FILE_NAME for both real file name and DOS version
- contains file reference for parent directory; one of the most useful values when located in MFT entry
- contains a set of 4 temporal values, same as $STANDARD_INFORMATION; frequently correspond to when created, moved, or renamed
- has fields for actual and allocated size of file; 0 for user files/directories
- contains flag field to identify:
	- directory
	- read-only
	- system file
	- compressed
	- encrypted
	- flags used in $STANDARD_INFORMATION

## 22. $DATA
- used to store any form of data
- type ID 128
- can be any size, including 0
- at least one allocated for each file; doesn't have name
- additional $DATA must have names
- file with 2 $DATA that are encrypted
	![[file with 2 data encrypted.png]]
- stores summary information in Windows
- some anti-virus or backup software create $DATA on files it processed
- directories can have $DATA in addition to index attributes
- can be encrypted or compressed; sets corresponding flag in attribute header
- encryption creates $LOGGED_UTITLITY_STREAM

## 23. *Alternate Data Streams(ADS)*
- additional $DATA can be used to hide data
- not shown when contents of directory are listed
- creating $DATA named foo on file.txt
``` cmd
C:\> echo "Hello There" > file.txt:foo
```

## 24. $ATTRIBUTE_LIST
- used when file/directory needs more than one MFT entry to store all attributes
- common when non-resident attribute becomes fragmented and runlist becomes long
- type ID 32
- always in base MFT entry
- contains list of all file's attributes, beside itself
- each entry contains attribute type and MFT entry address
- attribute header of non-base MFT entry $DATA has field to show starting VCN of run

![[MFT entry with attribute list.png]]

## 25. $SECURITY_DESCRIPTOR
- primarily found in FS from Windows NT; NTFS version 3.0+ have this for backward compatibility only
- *security descriptors* are used to desribe access control policy that should be applied to file/directory
- type ID 80
- newer versions store in single file instead of attribute; saves space for files with same security descriptor

## 26. $Secure file
- MFT entry 9
- $STANDARD_INFORMATION of every file/directory contains identifier, SID, used as index to $Secure to find appropriate descriptor; these 32-bit SIDs are different from Windows SIDs assigned to users
- contains 2 indexes, $SDH and $SII, and one $DATA, $SDS:
	- $DATA contains actual security descriptors
	- indexes are used to reference the descriptors
	- $SII is sorted by SID; used to locate security descriptor for a file through SID
	- $SDH is sorted by hash of security descriptor; used when new security descriptor is applied to a file/directory to create a new descriptor and SID if it doesn't already exist
- *istat* output:
	``` sh
	$ istat –f ntfs ntfs2.dd 9
	[REMOVED]
	Attributes:
	Type: $STANDARD_INFORMATION (16-0) Name: N/A Resident size: 72
	Type: $FILE_NAME (48-7) Name: N/A Resident size: 80
	Type: $DATA (128-8) Name: $SDS Non-Resident size: 266188
	10016 10017 10018 10019 10020 10021 10022 10023
	10024 10025 10026 10027 10028 10029 10030 10031
	10032 10033 10034 10035 10036 10037 10038 10039
	[REMOVED]
	Type: $INDEX_ROOT (144-11) Name: $SDH Resident size: 56
	Type: $INDEX_ROOT (144-14) Name: $SII Resident size: 56
	Type: $INDEX_ALLOCATION (160-9) Name: $SDH Non-Resident size: 4096
	8185 8186 8187 8188 8189 8190 8191 8192
	Type: $INDEX_ALLOCATION (160-12) Name: $SII Non-Resident size: 4096
	8196 8197 8198 8199 8200 8201 8202 8203
	Type: $BITMAP (176-10) Name: $SDH Resident size: 8
	Type: $BITMAP (176-13) Name: $SII Resident size: 8
	```
	- 200KB $DATA with copies of security descriptors
	- 2 indexes named $SDH and $SII

## 27. metadata category example image
- *istat* output part 1:
	``` sh
	$ istat –f ntfs ntfs1.dd 3234
	MFT Entry Header Values:
	Entry: 3234 Sequence: 1
	$LogFile Sequence Number: 103605752
	Allocated File
	Links: 1
	```
	- shows data in MFT entry header
	- sequence # is 1; first file to allocate entry
	- LSN corresponds to when file was last changed; FS journal log uses this value
- *istat* output part 2:
	``` sh
	$STANDARD_INFORMATION Attribute Values:
	Flags: Hidden, System, Not Content Indexed
	Owner ID: 256 Security ID: 271
	Quota Charged: 1024
	Last User Journal Update Sequence Number: 4372416
	Created: Thu Jun 26 10:22:46 2003
	File Modified: Thu Jun 26 15:31:43 2003
	MFT Modified: Tue Nov 18 11:03:53 2003
	Accessed: Sat Jul 24 04:53:36 2004
	```
	- data from $STANDARD_INFORMATION
	- SID is given; possible to look up security descriptor
	- Last User Journal Update Sequence Number is changed; used by change journal
- *istat* output part 3:
	``` sh
	$FILE_NAME Attribute Values:
	Flags: Archive
	Name: boot.ini
	Parent MFT Entry: 5 Sequence: 5
	Allocated Size: 0 Actual Size: 0
	Created: Thu Jun 26 10:22:46 2003
	File Modified: Thu Jun 26 10:22:46 2003
	MFT Modified: Thu Jun 26 10:22:46 2003
	Accessed: Thu Jun 26 10:22:46 2003
	```
	- data from $FILE_NAME
	- time value different from $STANDARD_INFORMATION
	- size values are 0
	- parent directory is MFT entry 5, root directory
- *istat* output part 4:
	``` sh
	Attributes:
	Type: $STANDARD_INFORMATION (16-0) Name: N/A Resident size: 72
	Type: $FILE_NAME (48-2) Name: N/A Resident size: 82234
	Type: $DATA (128-3) Name: $Data Resident size: 194
	```
	- can see all file attributes
	- has resident $DATA of 194 bytes
	- type ID and attribute ID are given after type name

## 28. metadata category allocation algorithms
- MFT entry allocation:
	- Windows allocates MFT entries on first-available basis starting with entry 24
	- entries 0~15 are reserved and set to allocated 
	- entries 16~23 are typically not allocated
	- when an entry is no longer being used, only flag identifying usage is changed; temporal and runlist information can be recovered
	- when an entry is allocated, it is wiped; there is no slack data from previous file in MFT entry
- attribute allocation:
	- MS sorts entries based on attribute type
	- if $DATA at end of entry is resident and size decreases, old contents can be found at end of entry (minus end marker 0xffff ffff)
	- when attribute grows from resident to non-resident, content in MFT entry exist until overwrite
- time value updating:
	- creation time is set for **new** file; create, copy
	- last modified time is set when value of $DATA, $INDEX_ROOT, or $INDEX_ALLOCATION are modified; move/copy does not change it
	- MFT modified time is set when any of attributes are changed; rename/move updates value since it changes $FILE_NAME but doesn't seem to change when moving to new volume
	- last accessed time is set when metadata or content is viewed, but not written immediately
- deleting files doesn't seem to update times
- copying a directory updates all 4 time values
- moving a directory within volume updates MFT modified and access times
- movig a directory to a new volume updates all 4 time values

## 29. metadata category analysis techniques
![[MFT entry for processing.png]]
1. locate MFT entry using starting address in boot sector
2. process MFT entry header to find first attribute
3. process first attribute:
	1. read header
	2. determine type
	3. process content appropriately
4. locate second attribute from length of first attribute
5. process attributes until all attributes have been processed

- core metadata can be found in $STANDARD_INFORMATION
- $DATA has file content
- $FILE_NAME is useful to verify creation date and determine parent directory
- resident attributes have content offset and size value in header
- non-resident attributes have runlist in header that describes starting cluster and number of clusters in each run
- unallocated MFT entries should be processed as well
- search for clusters that start with FILE signature if $MFT and $MFTMirr are not located

## 30. metadata category analysis considerations
- extra attributes can be used to hide data
- unused part in MFT entry can be used to hide data
- ways $DATA can be stored in entry:
	![[data in MFT entry.png]]
	- entry 90: $DATA can be recovered before reallocation
	- entry 91: $DATA can be recovered if cluster isn't reallocated
	- entry 92, 93: if either entry is reallocated $DATA can't be recovered
- it is harder to recover files with smaller MFT addresses
- Windows wipes unused bytes of last sector, but not unused sectors; deleted data can be found at end of file in slack space
- time values are not updated on delete

## 31. metadata category analysis scenario
objective: recover files from previous FS in a newly formatted disk
1. extract unallocated space of FS
	``` sh
	$ dls –f ntfs ntfs9.dd > ntfs9.dls
	```
2. search for old MFT entries
	``` sh
	$ sigfind –o 0 46494c45 ntfs9.dls
	```
3. analyze hits
	``` sh
	$ dd if=ntfs9.dls skip=412 count=2 | xxd
	0000000: 4649 4c45 3000 0300 b6e5 1000 0000 0000 FILE0...........
	0000016: 0100 0100 3800 0100 0003 0000 0004 0000 ....8...........
	0000032: 0000 0000 0000 0000 0400 0000 4800 0000 ............H...
	0000048: 0500 6661 0000 0000 1000 0000 6000 0000 ..fa........`...
	0000064: 0000 0000 0000 0000 4800 0000 1800 0000 ........H.......
	[REMOVED]
	0000144: b049 0000 0000 0000 3000 0000 7000 0000 .I......0...p...
	0000160: 0000 0000 0000 0200 5800 0000 1800 0100 ........X.......
	0000176: 0500 0000 0000 0500 d064 f7b9 eb69 c401 .........d...i..
	[REMOVED]
	0000240: 0b03 6c00 6500 7400 7400 6500 7200 3100 ..l.e.t.t.e.r.1.
	0000256: 2e00 7400 7800 7400 4000 0000 2800 0000 ..t.x.t.@...(...
	0000272: 0000 0000 0000 0300 1000 0000 1800 0000 ................
	0000288: 6dd4 12e6 d9d5 d811 a5c7 00b0 d01d e93f m..............?
	[REMOVED]
	0000304: 8000 0000 c801 0000 0000 1800 0000 0100 ................
	0000320: ae01 0000 1800 0000 4865 6c6c 6f20 4d72 ........Hello Mr
	0000336: 204a 6f6e 6573 2c0a 5765 2073 6861 6c6c Jones,.We shall
	[REMOVED]
	```
	- bytes 20~21: first attribute starts at offset 56
	- bytes 22~23: allocated before format
	- bytes 56~59: type $STANDARD_INFORMATION
	- bytes 60~63: length is 96 bytes
	- bytes 152~155: type $FILE_NAME
	- bytes 156~159: length 112 bytes
	- bytes 176~181: parent directory MFT entry 5, root
	- bytes 240~263: name of file: letter1.txt
	- bytes 264~267: type $OBJECT_ID
	- bytes 268~271: length 40 bytes
	- bytes 304~307: type $DATA
	- bytes 308~311: length 456 bytes
	- bytes 328~: content of $DATA

![[unallocated MFT entry processing.png]]

## 32. file name category

## 33. directory indexes
- NTFS directories have a normal MFT entry with special flag in header, $STANDARD_INFORMATION and $FILE_NAME attributes
- index entries in directory index contain a file reference and $FILE_NAME
- if Windows is configured to require a DOS space name, multiple $FILE_NAME for same file exist in index
- $INDEX_ROOT, $INDEX_ALLOCATION, $BITMAP are all assigned name $I30
- basic directory tree layout:
![[basic directory tree with 2 levels.png]]
	- entries in index record 0 have names less than hhh.txt
	- entries in index record 1 have names greater than hhh.txt
	- eeeeeeeeeeee.txt is not in DOS name space; there is second entry for it
- each node in tree has a header value that identifies where last allocated entry is

## 34. root directory
- always located in MFT entry 5
- named .
- has standard $INDEX_ROOT, $INDEX_ALLOCATION, and $BITMAP attributes
- all FS metadata files are located in this directory; but hidden from most users

## 35. links to files and directories
- *hard link*:
	- allows files to have more than one name
	- doesn't look different from original file name
	- allocated an entry in its parent directory index that points to same MFT entry as original name
	- when created, increment link count in MFT entry header; entry not unallocated until link count is zero
	- MFT entry has one $FILE_NAME for each hard link names
	- hard links can only be created within same volume
- *reparse points*:
	- used to link files, directories, and volumes
	- special file/directory that contains information about what it links to
	- can be used to mount volume on directory instead of at drive letter such as E:\
	- *symbolic link* is reparse point that links 2 files
	- *junction* is reparse point that links 2 directories
	- *mount point* is reparse point that links directory with volume
	- Windows Remote Storage Server feature uses reparse points to describe server location of a file or directory
	- have flag set in $STANDARD_INFORMATION and $FILE_NAME
	- $REPARSE_POINT contains information about where target file/directory is
	- kept track of using an index in \$Extend\$Reparse FS metadata file; sorted by file reference of reparse point
- NTFS keeps track of mount points in $DATA in root directory, called $MountMgrRemoteDatabase; contains list of target volumes pointed to by mount points

## 36. object identifier
- another method for addressing file/directory
- application or OS can assign unique 128-bit object identifier to file
- used to refer to file even after name change or move to different volume
- file/directory with object ID assigned has $OBJECT_ID that contains object ID and information about original domain and volume
- refer to \$Extend\$ObjId index to find file based on object ID; contains every assigned object ID with file reference ID

## 37. file name category allocation algorithms
- small directory has one node allocated to $INDEX_ROOT
- OS moves entries to an index record in $INDEX_ALLOCATION when entries no longer fit in $INDEX_ROOT
- when first index record fills up, a second is allocated and $INDEX_ROOT is used as root node
- tempora and size values in $FILE_NAME are updated at same rate as values in $STANDARD_INFORMATION of MFT entry

## 38. file name category analysis techniques
1. locate root directory
2. examine contents of $INDEX_ROOT and $INDEX_ALLOCATION to process directory
	- contains index record that correspond to index nodes
	- index record may also contain unallocated index entries; allocation status determined via $BITMAP
- file name may correspond to reparse point

## 39. file name category analysis considerations
- to determine if a file has truly been deleted, all index entries must be examined 
- file reference contains sequence number; easier to tell if file name matches MFT entry
- $FILE_NAME exists in index; basic information exists even after MFT entry is overwritten
- when trying to determine which deleted files exist in directory, check:
	- unallocated areas of each node in directory index tree
	- unallocated MFT entries

## 40. file name category analysis scenario
![[NTFS file name analysis scenario.png]]
- aaa.txt is shown as deleted:
	- aaa.txt has an unallocated entry in index
	- also has allocated entry in index; not deleted
- wrong time stamp is show for mmm.txt:
	- mmm.txt has entry 31 and seq 3
	- current entry 31 has seq 4; time stamp here is for new file
	- correct time stamp for mmm.txt can be found at $FILE_NAME in index entry
- www.txt is not shown as deleted:
	- deleted file name can be overwritten in index
	- orphan files should be searched for in MFT entries

## 41. application category
- NTFS supports many application-level features
- not essential with respect to FS
- in this section, data are essential if they are required for application-level goals, not for storing data

## 42. disk quota
- limits amount of space each user allocates
- can be set up by admin
- part of quota information is stored as FS data and other data is stored in application-level files such as registry
- $Quota exists in \$Extend directory and can be in any MFT entry
- $Quota uses 2 indexes to manage quota information:
	- $O: correlates SID with owner ID
	- $Q: correlates owner ID with how many bytes is being used by user and how much is allowed

## 43. disk quota analysis considerations
- considered non-essential; not needed when using FS
- quota could be useful when determining which users stored large amounts of data
- quota system is not turned on be default

## 44. logging (journaling)
![[LogFile data attribute layout.png]]
- record information about metadata updates before and after; allows quick recovery
- log journal file is located in MFT entry 2, named $LogFile
- $LogFile has no special attributes and log data is stored in $DATA
- log file has 2 major sections:
	- restart area:
	- logging area:
- example FS transactions:
	- create file/directory
	- change content of file/directory
	- rename file/directory
	- change data stored in MFT entry of file/directory
- log file does not contain non-resident data

## 45. restart area
- contains 2 copies of a DS that help OS determine what transactions need to be examined during cleanup
- contains pointer to logging area for last transaction that was known to be successful

## 46. logging area
- contains series of record
- each record has a *Logical Sequence Number(LSN)*, a unique 64-bit value
- has a finite size; restarts from beginning after reaching end
- LSN is assigned based on record creation date, not location
- update record:
	- used to describe FS transaction before/after it occurs
	- 2 major fields created before transaction:
		- redo field: contains information about what operation is going to do
		- undo field: shows how to undo operation
	- commit record: update record created after transaction to show completion
- checkpoint record:
	- identifies where in log file OS should start from to verify FS
	- created every 5 seconds
	- LSN value stored in restart area of log file

## 47. logging example
![[transaction logging example.png]]
1. scan from last checkpoint record
2. transaction 1 has a commit record
	- use redo field to ensure disk is in correct state
3. transaction 2 does not have a commit record
	- use undo field to ensure no partial changes exist in disk

## 48. logging analysis considerations
- unknown how log file is organized
- evidence, even if found, can be difficult to explain
- LSN value given in MFT entry header of file could be used to reconstruct order in which files were edited

## 49. change journal
- record when changes are made to file/directory
- can be used by application to determine which files have changed in a certain time span
- any appliation can turn journal feature on/off; off by default
- journal has 64-bit number assigned to it, which is changed every time journal is enabled/disabled; used to determine if journal may have missed changes while disabled
- when journal is disabled, Windows 2000 and XP purges file
- stored in \$Extend\$UsrJrnl file
- not typically allocated one of the reserved MFT entries
- has 2 $DATA:
	- $Max: contains basic information about journal
	- $J: contains journal as list of various size records
		- each record contains file name, time of change, and type of change
		- length of a record is based on length of file name
		- each record has a 64-bit *Update Sequence Number(USN)* used to index record in journal
		- USN stored in $STANDARD_INFORMATION of modified file
		- USN corresponds to byte offset in journal; easy to find record given USN
		- record does not include which data changed
- max size exists for journal:
	 - once reached, turns into sparse file
	 - file looks like it's getting bigger, but occupies same space on disk

## 50. change journal analysis considerations
- if enabled and trust worthy, could be useful in reconstructing recent events 

## 51. file allocation big picture
scenario: create file \dir1\file1.dat and assume dir1 already exists in root. File size is 4,000 bytes and cluster size is 2,048 bytes
1. read and process first sector
	1.  determine cluster size, MFT starting address, size of MFT entry
2. read and process first entry from MFT
	1. determine layout of MFT from $DATA
3. process $BITMAP of $MFT to find unused entry
	1. first free entry 304 is allocated to new file
	2. bit in $BITMAP set to 1
4. clear and initialize MFT entry 304
	1. $STANDARD_INFORMATION and $FILE_NAME are created
	2. times are set to current time
	3. in-use flag set in entry header
5. allocate clusters for content
	1. find and allocate clusters with $DATA of $Bitmap in entry 6
	2. 2 consecutive clusters, 692 and 693 are found and allocated
	3. corresponding bits are set to 1
	4. file content is written to clusters
	5. $DATA of entry 304 is updated with cluster addresses
	6. entry 304 file modified times are updated
6. add file name entry
	1. locate dir1 in root directory
	2. read $INDEX_ROOT and $INDEX_ALLOCATION and find dir1 index entry and MFT entry address
	3. update last access time of root directory
7. process dir1 MFT entry
	1. process $INDEX_ROOT to find where file.dat should go
	2. create new index entry and resort tree
	3. new index entry has MFT entry 304 in file reference address and time and flags are set appropriately
	4. last written, modified, and accessed times are updated for dir1
8. in each previous step, entries could have been made to:
	- FS journal in $LogFile
	- change journal in \$Extend\$UsrJrnl
	- user quota in \$Extend\$Quota

![[NTFS big picture file creation.png]]

## 52. file deletion big picture
scenario: delete \dir1\file1.dat
1. read and process first sector
	1.  determine cluster size, MFT starting address, size of MFT entry
2. read and process first entry from MFT
	1. determine layout of MFT from $DATA
3. find dir1
	1. process root directory 
	2. find dir1 index entry and get MFT entry address
	3. last accessed time of root directory is updated
4. search for file1.dat entry
	1. process $INDEX_ROOT of dir1
	2. find file1.dat index entry and get MFT entry address
5. remove file1.dat index entry
	1. remove entry from index
	2. last written, modified, and accessed time is updated for dir1
6. unallocate MFT entry 304
	1. clean in-use flag in header
	2. process $DATA of $Bitmap and set bitmap to 0 for this entry
7. process non-resident atttributes for MFT entry 304 
	1. corresponding clusters are set to unallocated in \$Bitmap (clusters 692 and 693)
8. in each previous step, entries could have been made to:
	- FS journal in $LogFile
	- change journal in \$Extend\$UsrJrnl
	- user quota in \$Extend\$Quota

![[NTFS big picture file deletion.png]]
- link between MFT entry and clusters still exists
- link between file name and MFT entry would exist if not overwritten

## 53. file recovery
- easier than other FS
- if directory index is resorted, file name information can be lost; but unallocated entries can be easily found in MFT
- MFT entries have $FILE_NAME attribute with file reference address of parent directory; easy to determine full path
- additional $DATA should be recovered if they exist
- to recover all deleted files, MFT should be examined for unallocated entries
- if file had more than 1 MFT entry, they may be required for recovery
- FS log or change log can be useful for recent deletions

## 54. consistency check
- used to identify corrupt images or tampering
- NTFS boot sectors enforces some values should be 0
- clusters marked as bad should be examined
- metadata structures reserved and unused are easy places to hide data
- each allocated cluster must be part of cluster run of a file
- each allocated MFT entry must have:
	- inuse flag and $BITMAP bit set
	- directory index entry for each file name
- valid value combinations are unknown for flags and options
