## 1. check file system type
- *fdisk*
```
$ fdisk -l image.raw 
Disk toy_story.raw: 60 GiB, 64424509440 bytes, 125829120 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 29CC463E-111B-4F2F-8A56-3F524B3FCFE8

Device             Start       End   Sectors  Size Type
toy_story.raw1      2048    206847    204800  100M EFI System
toy_story.raw2    206848    239615     32768   16M Microsoft reserved
toy_story.raw3    239616 124705181 124465566 59.3G Microsoft basic data
toy_story.raw4 124706816 125825023   1118208  546M Windows recovery environment
```

- *mmls*
```
$ mmls image.raw  
GUID Partition Table (EFI)  
Offset Sector: 0  
Units are in 512-byte sectors  
  
     Slot      Start        End          Length       Description  
000:  Meta      0000000000   0000000000   0000000001   Safety Table  
001:  -------   0000000000   0000002047   0000002048   Unallocated  
002:  Meta      0000000001   0000000001   0000000001   GPT Header  
003:  Meta      0000000002   0000000033   0000000032   Partition Table  
004:  000       0000002048   0000206847   0000204800   EFI system partition  
005:  001       0000206848   0000239615   0000032768   Microsoft reserved partition  
006:  002       0000239616   0124705181   0124465566   Basic data partition  
007:  -------   0124705182   0124706815   0000001634   Unallocated  
008:  003       0124706816   0125825023   0001118208      
009:  -------   0125825024   0125829119   0000004096   Unallocated
```

- *fsstat* on correct partition
```
$ fsstat toy_story.raw -o 239616  
FILE SYSTEM INFORMATION  
--------------------------------------------  
File System Type: NTFS  
Volume Serial Number: 464ED8AC4ED895D1  
OEM Name: NTFS       
Version: Windows XP  
  
METADATA INFORMATION  
--------------------------------------------  
First Cluster of MFT: 786432  
First Cluster of MFT Mirror: 2  
Size of MFT Entries: 1024 bytes  
Size of Index Records: 4096 bytes  
Range: 0 - 458752  
Root Directory: 5  
  
CONTENT INFORMATION  
--------------------------------------------  
Sector Size: 512  
Cluster Size: 4096  
Total Cluster Range: 0 - 15558194  
Total Sector Range: 0 - 124465564  
  
$AttrDef Attribute Values:  
$STANDARD_INFORMATION (16)   Size: 48-72   Flags: Resident  
$ATTRIBUTE_LIST (32)   Size: No Limit   Flags: Non-resident  
$FILE_NAME (48)   Size: 68-578   Flags: Resident,Index  
$OBJECT_ID (64)   Size: 0-256   Flags: Resident  
$SECURITY_DESCRIPTOR (80)   Size: No Limit   Flags: Non-resident  
$VOLUME_NAME (96)   Size: 2-256   Flags: Resident  
$VOLUME_INFORMATION (112)   Size: 12-12   Flags: Resident  
$DATA (128)   Size: No Limit   Flags:    
$INDEX_ROOT (144)   Size: No Limit   Flags: Resident  
$INDEX_ALLOCATION (160)   Size: No Limit   Flags: Non-resident  
$BITMAP (176)   Size: No Limit   Flags: Non-resident  
$REPARSE_POINT (192)   Size: 0-16384   Flags: Non-resident  
$EA_INFORMATION (208)   Size: 8-8   Flags: Resident  
$EA (224)   Size: 0-65536   Flags:    
$LOGGED_UTILITY_STREAM (256)   Size: 0-65536   Flags: Non-resident
```

## 2. extract MFT
- verify MFT entry 
``` data
$ istat toy_story.raw -o 239616 0 | grep -e "Name"  
Name: $MFT  
Type: $STANDARD_INFORMATION (16-0)   Name: N/A   Resident   size: 72  
Type: $FILE_NAME (48-3)   Name: N/A   Resident   size: 74  
Type: $DATA (128-6)   Name: N/A   Non-Resident   size: 469762048  init_size: 469762048  
Type: $BITMAP (176-5)   Name: N/A   Non-Resident   size: 57352  init_size: 57352
```

- extract MFT entry
``` data
$ icat toy_story.raw -o 239616 0 > "mft_file"
```

- make MFT into CSV
``` data
$ MFTECmd -f mft_file --csv .
```

- view all columns (optional)
``` data
$ csvtool head 1 *MFTECmd*.csv | awk -F, '{for(i=1;i<=NF;i++) print i ": " $i}'  
1: ﻿EntryNumber  
2: SequenceNumber  
3: InUse  
4: ParentEntryNumber  
5: ParentSequenceNumber  
6: ParentPath  
7: FileName  
8: Extension  
9: FileSize  
10: ReferenceCount  
11: ReparseTarget  
12: IsDirectory  
13: HasAds  
14: IsAds  
15: SI<FN  
16: uSecZeros  
17: Copied  
18: SiFlags  
19: NameType  
20: Created0x10  
21: Created0x30  
22: LastModified0x10  
23: LastModified0x30  
24: LastRecordChange0x10  
25: LastRecordChange0x30  
26: LastAccess0x10  
27: LastAccess0x30  
28: UpdateSequenceNumber  
29: LogfileSequenceNumber  
30: SecurityId  
31: ObjectIdFileDroid  
32: LoggedUtilStream  
33: ZoneIdContents  
34: SourceFile  
35: ResidentDataBase64  
36: ResidentDataHex  
37: ResidentDataASCII
```

## 3. search for key artifacts
### 3.1. prefetch files
- extract prefetch files:
``` sh
$ python3 extract_artifacts.py -i "toy_story.raw" -o 239616 -p prefetch_dir -a prefetch
```

- 

### 3.2. LNK files
### 3.3. browser history
### 3.4. SRUM
### 3.5. Windows event logs
### 3.6. registry artifacts
- Amcache
- Registry
- Shimcache
- Userassist
### 3.7. downloads directory
### 3.8. temp directory
### 3.9. *.exe* files
### 3.10. change journal (*$UsnJrnl*)
### 3.11. *$LogFile*
### 3.12. ADS
