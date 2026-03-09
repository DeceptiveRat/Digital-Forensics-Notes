## 1. fixup values
![[fixup value example.png]]
- used for DS over 1 sector in length
- last 2 bytes of each sector are replaced with a signature value when written to disk
- signature is used to verify integrity of data; all sectors should have same signature
- used only in DS, not file content
- DS with fixup values have header fields that identify 16-bit signature and array for original values
- signature value increases by 1 for each write to disk
- used to detect damaged sectors and corrupt DS

## 2. MFT entries
| Byte Range | Description | Essential | 
| --- | --- | --- | 
| 0–3 | Signature("FILE") | No | 
| 4–5 | Offset to fixup array | Yes | 
| 6–7 | Number of entries in fixup array | Yes | 
| 8–15 | $LogFile Sequence Number(LSN) | No | 
| 16–17 | Sequence value | No | 
| 18–19 | Link count | No | 
| 20–21 | Offset to first attribute | Yes | 
| 22–23 | Flags(in-use and directory) | Yes | 
| 24–27 | Used size of MFT entry | Yes | 
| 28–31 | Allocated size of MFT entry | Yes | 
| 32–39 | File reference to base record | No | 
| 40–41 | Next attribute id | No | 
| 42–1023 | Attributes and fixup values | Yes | 
- some entries have BAAD as signature; if *chkdsk* finds error
- LSN is used for FS log
- sequence value increases when entry is allocated or unallocated
- link count shows how many directories have entries for this MFT entry; incremented by 1 for each hard link
- offset is relative to start of entry
- flag field has 2 values; 0x01 is set when entry is in use 0x02 is set when entry is for directory
- *dd* with *icat* can skip to correct location; see entry 1234
``` sh
$ icat -f ntfs ntfs1.dd 0 | dd bs=1024 skip=1234 count=1 | xxd
```

## 3. MFT entry example
- $DATA for $MFT output
``` sh 
$ icat –f ntfs ntfs1.dd 0-128 | xxd
0000000: 4649 4c45 3000 0300 4ba7 6401 0000 0000 FILE0...K.d.....
0000016: 0100 0100 3800 0100 b801 0000 0004 0000 ....8...........
0000032: 0000 0000 0000 0000 0600 0000 0000 0000 ................
0000048: 5800 0000 0000 0000 1000 0000 6000 0000 X...........`...
[REMOVED]
0000496: 3101 b43a 0500 0000 ffff ffff 0000 5800 1..:..........X.
0000512: 0000 0000 0000 0000 0000 0000 0000 0000 ................
[REMOVED]
0001008: 0000 0000 0000 0000 0000 0000 0000 5800 ..............X.
```
- bytes 0~3: FILE signature
- bytes 4~5: fixup array located 48 bytes into MFT entry
- bytes 6~7: array has 3 values in it
- bytes 16~17: sequence value for MFT entry is `
- bytes 18~19: link count 1; only 1 name
- bytes 20~21: first attribute located at offset 56
- bytes 22~23: entry is in use
- bytes 32~39: this is base entry
- bytes 40~41: next attribute ID is 6
- bytes 48~49: signature value
- bytes 50~53: original values 
- bytes 56~509: attribute data
- bytes 510~511: signature value
- bytes 512~1021: attribute data
- bytes 1022~1023: signature value

## 4. attribute header
- first 16 bytes of attribute header

	| Byte Range | Description | Essential |
	| --- | --- | --- |
	| 0–3 | Attribute type identifier | Yes |
	| 4–7 | Length of attribute | Yes |
	| 8–8 | Non-resident flag | Yes |
	| 9–9 | Length of name | Yes |
	| 10–11 | Offset to name | Yes |
	| 12–13 | Flags | Yes |
	| 14–15 | Attribute identifier | Yes |
	- non-resident flag set to 1 when non-resident
	- flag value identifies:
		- 0x0001: compressed
		- 0x4000: encrypted
		- 0x8000: sparse
	- offset to name is relative to start of attribute
- resident attribute

	| Byte Range | Description | Essential |
	| --- | --- | --- |
	| 0–15 | General header | Yes | 
	| 16–19 | Size of content | Yes | 
	| 20–21 | Offset to content | Yes | 
	- give size and location relative to start of attribute
	- content is also called *stream*
	- example:
		``` sh
		0000000: 1000 0000 6000 0000 0000 1800 0000 0000 ....`...........
		0000016: 4800 0000 1800 0000 305a 7a1f f63b c301 H.......0Zz..;..
		```
		- bytes 0~3: attribute type, $STANDARD_INFORMATION
		- bytes 4~7: size of 96 bytes
		- byte 8: resident attribute
		- byte 9: no name
		- bytes 12~13: no flag
		- bytes 14~15: no id
		- bytes 16~19: attribute content 72 bytes long
		- bytes 20~21: offset of content is 24 bytes from start of attribute
		- content length 72 bytes + 24 bytes offset = 96 total length
- non-resident attribute

	| Byte Range | Description | Essential |
	| --- | --- | --- |
	| 0–15 | General header | Yes | 
	| 16–23 | Starting Virtual Cluster Number(VCN) of the runlist | Yes | 
	| 24–31 | Ending VCN of the runlist | Yes | 
	| 32–33 | Offset to the runlist | Yes | 
	| 34–35 | Compression unit size | Yes | 
	| 36–39 | Unused | No | 
	| 40–47 | Allocated size of attribute content | No | 
	| 48–55 | Actual size of attribute content | Yes | 
	| 56–63 | Initialized size of attribute content | No | 
	- start/end VCN numbers used when multiple MFT entries are needed to describe single attribute
	- compression unit size only needed for compressed attributes
	- example:
		``` sh
		0000000: 8000 0000 6000 0000 0100 4000 0000 0100 ....`.....@.....
		0000016: 0000 0000 0000 0000 ef20 0000 0000 0000 ......... ......
		0000032: 4000 0000 0000 0000 00c0 8300 0000 0000 @...............
		0000048: 00c0 8300 0000 0000 00c0 8300 0000 0000 ................
		0000064: 32c0 1eb5 3a05 2170 1b1f 2290 015f 7e31 2...:.!p..".._~1
		0000080: 2076 ed00 2110 8700 00b0 6e82 4844 7e82 v..!.....n.HD~.
		```
		- bytes 0~3: attribute type 128
		- bytes 4~7: total size 96 bytes
		- byte 8: non-resident attribute
		- byte 9: length of attribute name is 0; default $DATA, not ADS
		- bytes 12~13: no flags
		- bytes 16~23: starting VCN of run is 0
		- bytes 24~31: ending VCN of run is 8,431
		- bytes 32~33: offset of runlist is 64 bytes
		- bytes 40~47: allocated space 8,634,368 bytes
		- bytes 48~55: actual size 8,634,368 bytes
		- bytes 56~63: initialized size 8,634,368 bytes
		- byte 64: 3 bytes in offset field, 2 bytes in run length field
		- bytes 65~66: run length 7,872 clusters
		- bytes 67~69: run offset is 342,709 clusters
		- first run: clusters 342,709~350,580
		- byte 70: 2 bytes in offset field, 1 byte in run length field
		- byte 71: run length 112 clusters
		- bytes 72~73: run offset is 7,963 clusters
		- second run: clusters 350,672~350,783

## 5. run list

![[run list layout.png]]
- variable length, at least 1 byte
- first byte organized into 2 nibbles:
	- first nibble: number of bytes in run offset field
	- second nibble: number of bytes in run length field
- values are in cluster-sized units
- offset field is a signed value relative to previous offset:
	- offset of first run relative to start of FS
	- offset of second run relative to previous first offset
- negative number has *Most Significant Bit(MSB)* set to 1; all previous bits must be set to 1 to put in calculator

## 6. $STANDARD_INFORMATION
- always resident
- contains basic metadata for file/directory
- exists in every file/directory
- typically first attribute
- fields:

	| Byte Range | Description | Essential | 
	| --- | --- | --- | 
	| 0–7 | Creation time | No | 
	| 8–15 | File altered time | No | 
	| 16–23 | MFT altered time | No | 
	| 24–31 | File accessed time | No | 
	| 32–35 | Flags | No | 
	| 36–39 | Maximum number of versions | No | 
	| 40–43 | Version number | No | 
	| 44–47 | Class ID | No | 
	| 48–51 | Owner ID(version 3.0+) | No | 
	| 52–55 | Security ID(version 3.0+) | No | 
	| 56–63 | Quota Charged(version 3.0+) | No | 
	| 64–71 | Update Sequence Number(USN)(version 3.0+) | No | 
	- time value stored as number of 100 ns since Jan 1st, 1601 UTC
	- same time fields exist in $FILE_NAME; displayed by Windows and updated
	- ID values used for application-level features or security
	- Security ID is index to $Secure file, not SID value
	- flag values:

		| Flag Value | Description | Essential | 
		| --- | --- | --- | 
		| 0x0001 | Read Only | No | 
		| 0x0002 | Hidden | No | 
		| 0x0004 | System | No | 
		| 0x0020 | Archive | No | 
		| 0x0040 | Device | No | 
		| 0x0080 | \#Normal | No | 
		| 0x0100 | Temporary | No | 
		| 0x0200 | Sparse file | No | 
		| 0x0400 | Reparse point | No | 
		| 0x0800 | Compressed | No | 
		| 0x1000 | Offline | No | 
		| 0x2000 | Content is not being indexed for faster searches | No | 
		| 0x4000 | Encrypted | No | 
		- many are same as FAT 
		- flags for encrypted and sparse attributes are also given in attribute header; non-essential here

## 7. $STANDARD_INFORMATION example
- *icat* output
	``` sh
	$ icat -f ntfs ntfs1.dd 0-16 | xxd
	0000000: 305a 7a1f f63b c301 305a 7a1f f63b c301 0Zz..;..0Zz..;..
	0000016: 305a 7a1f f63b c301 305a 7a1f f63b c301 0Zz..;..0Zz..;..
	0000032: 0600 0000 0000 0000 0000 0000 0000 0000 ................
	0000048: 0000 0000 0001 0000 0000 0000 0000 0000 ................
	0000064: 0000 0000 0000 0000                     ........
	```
	- bytes 0~7: creation time; same for each 4 time fields
	- bytes 32~35: flag bits for hidden and system; not expected for FS metadata file
	- bytes 36~39, 40~43: file versions not being used
	- bytes 44~47: class ID 0
	- bytes 48~51: owner ID 0
	- bytes 52~55: security ID 0

## 8. $FILE_NAME
- placed in MFT entry to store file name and parent directory information
- also used in directory index
- doesn't contain essential information as MFT entry; but does as directory index
- usually second attribute; $ATTRIBUTE_LIST exists between this and $STANDARD_INFORMATION if multiple MFT entries are required
- always resident
- fields:

	| Byte Range | Description | Essential | 
	| --- | --- | --- | 
	| 0–7 | File reference of parent directory | No | 
	| 8–15 | File creation time | No | 
	| 16–23 | File modification time | No | 
	| 24–31 | MFT modification time | No | 
	| 32–39 | File access time | No | 
	| 40–47 | Allocated size of file | No | 
	| 48–55 | Real size of file | No | 
	| 56–59 | Flags | No | 
	| 60–63 | Reparse value | No | 
	| 64–64 | Length of name | Yes / No | 
	| 65–65 | Namespace | Yes / No | 
	| 66+ | Name | Yes / No | 
	- final 3 name fields essential when used in directory index
	- flag field uses same values as $STANDARD_INFORMATION
	- namespace byte identifies what rules the name follows

		| Name space value | Style | Description |
		| --- | --- | --- |
		| 0 | POSIX |  The name is case sensitive and allows all Unicode characters except for '/' and NULL. |
		| 1 | Win32 |  The name is case insensitive and allows most Unicode characters except for special values such as '/', '\', ':', '>', '<', and '?'. |
		| 2 | DOS |  The name is case insensitive, upper case, and no special characters. The name must have eight or fewer characters in the name and three or less in the extension. |
		| 3 | Win32 & DOS |  Used when the original name already fits in the DOS namespace and two names are not needed. |

## 9. $FILE_NAME example
- *icat* output:
	``` sh
	$ icat -f ntfs ntfs1.dd 0-48 | xxd
	0000000: 0500 0000 0000 0500 305a 7a1f f63b c301 ........0Zz..;..
	0000016: 305a 7a1f f63b c301 305a 7a1f f63b c301 0Zz..;..0Zz..;..
	0000032: 305a 7a1f f63b c301 0040 0000 0000 0000 0Zz..;...@......
	0000048: 0040 0000 0000 0000 0600 0000 0000 0000 .@..............
	0000064: 0403 2400 4d00 4600 5400                ..$.M.F.T.
	```
	- bytes 0~5: MFT entry in file reference
	- bytes 6~7: sequence number in file reference
	- bytes 8~15: creation time
	- bytes 40~47: allocated size
	- bytes 48~55: actual size; not always accurate when used in MFT entry, but accurate when used as directory index
	- bytes 56~56: hidden and system flags set; same as $STANDARD_INFORMATION
	- byte 64: name is 4 letters long
	- byte 65: name space 3
	- bytes 66+: name in UTF-16 Unicode; $MFT
- *icat* output for two $FILE_NAME:
	``` sh
	$ icat -f ntfs ntfs1.dd 5009-48-2 | xxd
	0000000: 3920 0000 0000 0300 00b6 89a9 086a c401 9 ...........j..
	0000016: 00b6 89a9 086a c401 00b6 89a9 086a c401 .....j.......j..
	0000032: 00b6 89a9 086a c401 0000 0000 0000 0000 .....j..........
	0000048: 0000 0000 0000 0000 2020 0000 0000 0000 ........ ......
	0000064: 0b01 3500 3700 3300 3900 3800 3400 3000 ..5.7.3.9.8.4.0.
	0000080: 3800 6400 3000 3100                     8.d.0.1.
	$ icat -f ntfs ntfs1.dd 5009-48-3 | xxd
	0000000: 3920 0000 0000 0300 00b6 89a9 086a c401 9 ...........j..
	0000016: 00b6 89a9 086a c401 00b6 89a9 086a c401 .....j.......j..
	0000032: 00b6 89a9 086a c401 0000 0000 0000 0000 .....j..........
	0000048: 0000 0000 0000 0000 2020 0000 0000 0000 ........ ......
	0000064: 0802 3500 3700 3300 3900 3800 3400 7e00 ..5.7.3.9.8.4.~.
	0000080: 3100                                    1.
	```
	- name space 1 and 2 were used

## 10. $DATA
- no native structure
- type ID 128
- no min/max size
- non-resident if content is over 700 bytes
- usually last attribute in MFT entry

## 11. $ATTRIBUTE_LIST
- exists to show where other attributes can be located
- used for files with attribute headers that don't fit in one MFT entry
- contains list with an entry for every attribute in file/directory
- type ID 32
- list entry field:

	| Byte Range | Description | Essential | 
	| --- | --- | --- | 
	| 0–3 | Attribute type | Yes | 
	| 4–5 | Length of this entry | Yes | 
	| 6–6 | Length of name | Yes | 
	| 7–7 | Offset to name(relative to start of this entry) | Yes | 
	| 8–15 | Starting VCN in attribute | Yes | 
	| 16–23 | File reference where attribute is located | Yes | 
	| 24–24 | Attribute ID | Yes | 
	- starting VCN used when multiple MFT entries are used to describe single attribute

## 12. $ATTRIBUTE_LIST example
- *icat* output
	``` sh
	$ icat -f ntfs ntfs1.dd 5009-32 | xxd
	0000000: 1000 0000 2000 001a 0000 0000 0000 0000 .... ...........
	0000016: 9113 0000 0000 0800 0000 0000 0000 0000 ................
	0000032: 3000 0000 2000 001a 0000 0000 0000 0000 0... ...........
	0000048: 9113 0000 0000 0800 0300 0000 0006 0000 ................
	0000064: 3000 0000 2000 001a 0000 0000 0000 0000 0... ...........
	0000080: 9113 0000 0000 0800 0200 0200 502d 40bc ............P-@.
	0000096: 8000 0000 2000 001a 0000 0000 0000 0000 .... ...........
	0000112: 3713 0000 0000 1200 0000 0000 1000 0000 7...............
	0000128: 8000 0000 2000 001a 2014 0000 0000 0000 .... ... .......
	0000144: ad13 0000 0000 0800 0000 0000 0000 0000 ................
	```
	- bytes 0~3: type of first entry, 16; $STANDARD_INFORMATION
	- bytes 4~5: length of list entry: 32 bytes
	- bytes 16~21: attribute located in MFT entry 5,009; current MFT entry
	- next 2 entries starting at bytes 32 and 64 are $FILE_NAME; also located in current MFT entry
	- byte 96: first entry of $DATA
	- bytes 104~111: VCN 0
	- bytes 112~117: located in MFT entry 4,818
	- second $DATA entry begins at byte 128: ID value is same as first $DATA entry; same $DATA
	- bytes 136~143: second $DATA entry starting VCN 5,152
	- bytes 144~149: file reference to second $DATA entry

	![[data attribute example layout.png]]

- *istat* on non-base entry 4,919
	``` sh
	$ istat –f ntfs ntfs1.dd 4919
	MFT Entry Header Values:
	Entry: 4919 Sequence: 18
	Base File Record: 5009
	$LogFile Sequence Number: 66117460
	Allocated File
	Links: 0
	[REMOVED]
	Attributes:
	Type: $DATA (128-0) Name: $Data Non-Resident size: 5787792
	929409 929410 929411 929412 929413 929414 929415 929416
	[REMOVED]
	```
	- no $FILE_NAME or $STANDARD_INFORMATION

## 13. $OBJECT_ID
- type ID 64
- stores file's 128-bit global object identifier
- object identifier can be used to address file instead of name; allows file to be found after being renamed
- \$Extend\$ObjId index sorted by object IDs and contains file reference addresses where each file can be found
- fields:

	| Byte Range | Description | Essential | 
	| --- | --- | --- | 
	| 0–15 | Object ID | Yes | 
	| 16–31 | Birth volume ID | No | 
	| 32–47 | Birth object ID | No | 
	| 48–63 | Birth domain ID | No | 
	- many with object ID assigned have only the first value and attribute size is 16 bytes
- $Volume file frequently contains $OBJECT_ID:
	``` sh
	$ icat -f ntfs img.dd 3-64 | xxd
	0000000: fe24 b024 e292 fe47 95ac e507 4bf5 6782 .$.$...G....K.g.
	```

## 14. $REPARSE_POINT
- type ID 192
- used for files that are reparse points
- reparse points are used for symbolic links, junctions, and mount points for volumes
- application specific content can be developed
- content structure of junction and mount point:

	| Byte Range | Description | Essential | 
	| --- | --- | --- | 
	| 0–3 | Reparse type flags | Yes | 
	| 4–5 | Size of reparse data | Yes | 
	| 6–7 | Unused | No | 
	| 8–9 | Offset to target name(relative to byte 16) | Yes | 
	| 10–11 | Length of target name | Yes | 
	| 12–13 | Offset to print name of target(relative to byte 16) | Yes | 
	| 14–15 | Length of print name | Yes | 
	- type flags for a junction or mount point will have the 0xa000 0000 flag set
- reparse point linking to c:\windows:
	``` sh
	$ icat -f ntfs ntfs2.dd 167-192 | xxd
	0000000: 0300 00a0 2800 0000 0000 1c00 1e00 0000 ....(...........
	0000016: 5c00 3f00 3f00 5c00 6300 3a00 5c00 7700 \.?.?.\.c.:.\.w.
	0000032: 6900 6e00 6400 6f00 7700 7300 0000 1200 i.n.d.o.w.s.....
	```
	- bytes 8~9: offset to target name is 0; starts at byte 16
	- bytes 10~11: name length
	- bytes 16+: name of target; \??\c:\windows

## 15. $INDEX_ROOT
- always resident
- type ID 144
- always root of index tree
- can only store small list of index entries
- has a 16-byte header followed by node header and list of index entries
	![[INDEXROOT layout.png]]
- index root header fields:

	| Byte Range | Description | Essential | 
	| --- | --- | --- | 
	| 0–3 | Type of attribute in index(0 if entry does not use an attribute) | Yes | 
	| 4–7 | Collation sorting rule | Yes | 
	| 8–11 | Size of each index record in bytes | Yes | 
	| 12–12 | Size of each index record in clusters | Yes | 
	| 13–15 | Unused | No | 
	| 16+ | Node header | Yes | 
	- identifies:
		- type of attribute that index entries will contain
		- how entries are sorted
		- size of each index record in $INDEX_ALLOCATION
	- byte 12 is number of clusters or log of size

## 16. $INDEX_ROOT example
- *icat* output:
	``` sh
	$ icat -f ntfs ntfs1.dd 7774-144 | xxd
	0000000: 3000 0000 0100 0000 0010 0000 0400 0000 0...............
	0000016: 1000 0000 a000 0000 a000 0000 0100 0000 ................
	[REMOVED]
	```
	- bytes 0~3: attribute in index is for type 48; $FILE_NAME
	- bytes 8~11: each index record size 4,096 bytes

## 17. $INDEX_ALLOCATION
- non-resident
- filled with *index records*
- index record has static size and contains one node in sorted tree
- typical index record size is 4,096 bytes
- type ID 160
- should not exist without $INDEX_ROOT
- index record starts with special header DS followed by node header and list of index entries
	![[INDEXALLOCATION layout.png]]
- index record header:

	| Byte Range | Description | Essential | 
	| --- | --- | --- | 
	| 0–3 | Signature value("INDX") | No | 
	| 4–5 | Offset to fixup array | Yes | 
	| 6–7 | Number of entries in fixup array | Yes | 
	| 8–15 | $LogFile Sequence Number(LSN) | No | 
	| 16–23 | The VCN of this record in the full index stream | Yes | 
	| 24+ | Node header | Yes | 
	- first 4 fields almost identical to MFT entry; only difference is signature
	- VCN value identifies where record fits in tree
	- when index entry points to child node, it uses VCN address in node's index record header

## 18. $INDEX_ALLOCATION example
- *icat* output
	``` sh
	$ icat –f ntfs ntfs1.dd 7774-160 | xxd
	0000000: 494e 4458 2800 0900 4760 2103 0000 0000 INDX(...G`!.....
	0000016: 0000 0000 0000 0000 2800 0000 f808 0000 ........(.......
	[REMOVED]
	0004096: 494e 4458 2800 0900 ed5d 2103 0000 0000 INDX(....]!.....
	0004112: 0400 0000 0000 0000 2800 0000 6807 0000 ........(...h...
	0004128: e80f 0000 0000 0000 3b00 0500 6900 c401 ........;...i...
	[REMOVED]
	```
	- bytes 0~3: signature value
	- bytes 4~5, 6~7: fixup record values
	- bytes 16~23: index record is VCN 0 buffer
	- byte 24: start of node header
	- bytes 4,112~4,119: VCN 

## 19. $BITMAP
- used to keep track of which index record in $INDEX_ALLOCATION are allocated to an index record
- also used by $MFT to keep track of allocated MFT entries
- type ID 176
- organized by bytes, each bit corresponds to index record
- *icat* output:
	``` sh 
	$ icat -f ntfs ntfs1.dd 7774-176 | xxd
	0000000: 0300 0000 0000 0000 ........
	```
	- byte 0: 0000 0011 in binary; index records 0 and 1 are allocated

## 20 index node header
- used to show where list of index entries starts/ends
- fields:

	| Byte Range | Description | Essential |
	| --- | --- | --- |
	| 0–3 | Offset to start of index entry list(relative to start of the node header) | Yes |
	| 4–7 | Offset to end of used portion of index entry list (relative to start of the node header) | Yes |
	| 8–11 | Offset to end of allocated index entry list buffer (relative to start of the node header) | Yes |
	| 12–15 | Flags | No |
	- flag field only has one flag: 0x01 set when there are children node pointed to by the entries in this list; same flag exists in each index entry
- *icat* output
	``` sh
	$ icat -f ntfs ntfs1.dd 7774-144 | xxd
	0000000: 3000 0000 0100 0000 0010 0000 0400 0000 0...............
	0000016: 1000 0000 a000 0000 a000 0000 0100 0000 ................
	[REMOVED]
	```
	- byte 16: start of node header
	- bytes 16~19: list starts 16 bytes away
	- bytes 20~23: list ends 160 bytes away
	- byte 28: there are children nodes to this node
- *icat* output 2
	``` sh
	$ icat –f ntfs ntfs1.dd 7774-160 | xxd
	0000000: 494e 4458 2800 0900 4760 2103 0000 0000 INDX(...G`!.....
	0000016: 0000 0000 0000 0000 2800 0000 f808 0000 ........(.......
	0000032: e80f 0000 0000 0000 2100 0000 0600 0000 ........!.......
	[REMOVED]
	```
	- bytes 0~23: index record header
	- bytes 24~27: index entry list starts at byte offset 40
	- index entry list does not immediately follow node header because fixup record array is in between 
	- bytes 28~31: offset to end of last index entry 2,296
	- bytes 32~35: end of allocated list buffer 4,072; 1,776 bytes of unused space in buffer
	- bytes 36~39: flag value is 0; no children nodes

## 21. generic index entry 
- standard fields:

	| Byte Range | Description | 
	| --- | --- | 
	| 0–7 | Undefined | 
	| 8–9 | Length of this entry | 
	| 10–11 | Length of content | 
	| 12–15 | Flags | 
	| 16+ | Content | 
	| Last 8 bytes of entry starting on an 8-byte boundary | VCN of child node in $INDEX_ALLOCATION (field exists only if flag is set) | 
	- first 8 bytes used to store data specific to index entry
	- flag field values:

		| Value | Description |
		| --- | --- |
		| 0x01 | Child node exists |
		| 0x02 | Last entry in list |

## 22. directory index entry
- used for file names
- includes file reference address and $FILE_NAME
- fields

	| Byte Range | Description | Essential | 
	| --- | --- | --- | 
	| 0–7 | MFT file reference for file name | Yes | 
	| 8–9 | Length of this entry | Yes | 
	| 10–11 | Length of $FILE_NAME attribute | No | 
	| 12–15 | Flags | Yes | 
	| 16+ | $FILE_NAME Attribute(if length is > 0) | Yes | 
	| Last 8 bytes of entry starting on an 8-byte boundary | VCN of child node in $INDEX_ALLOCATION (field exists only if flag is set) | Yes |

## 23. directory index entry example
- *icat* output:
	``` sh 
	$ icat -f ntfs ntfs1.dd 7774-144 | xxd
	0000000: 3000 0000 0100 0000 0010 0000 0400 0000 0...............
	0000016: 1000 0000 a000 0000 a000 0000 0100 0000 ................
	0000032: c51e 0000 0000 0500 7800 5a00 0100 0000 ........x.Z.....
	0000048: 5e1e 0000 0000 0300 e03d ca37 5029 c401 ^........=.7P)..
	0000064: 004c c506 0202 c401 e09a 2a36 5029 c401 .L........*6P)..
	0000080: d0e4 22b5 096a c401 0004 0000 0000 0000 .."..j..........
	0000096: 7003 0000 0000 0000 2120 0000 0000 0000 p.......! ......
	0000112: 0c02 4d00 4100 5300 5400 4500 5200 7e00 ..M.A.S.T.E.R.~.
	0000128: 3100 2e00 5400 5800 5400 0000 0000 0300 1...T.X.T.......
	0000144: 0000 0000 0000 0000 0000 0000 0000 0000 ................
	0000160: 1800 0000 0300 0000 0400 0000 0000 0000 ................
	[REMOVED]
	```
	- bytes 32~37: for MFT entry 7,877
	- bytes 40~41: size of index entry is 120 bytes
	- bytes 42~43: size of attribute is 90 bytes
	- byte 44: child node exists
	- bytes 48~137: $FILE_NAME
	- bytes 144~151: VCN of child node; cluster 0
	- bytes 160~161: entry size 24 bytes
	- bytes 164~167: final entry, contains child
	- bytes 168~175: child node VCN 4

![[INDEX ROOT and INDEX ALLOCATION layout.png]]

## 24. $MFT file
- MFT entry 0
- contains unique $BITMAP used to organize allocation status of MFT entries
- $BITMAP *icat* output:
	``` sh
	$ icat -f ntfs ntfs1.dd 0-176 | xxd
	0000000: ffff 00ff ffff ffff ffff ffff ffff ffff ................
	0000016: ffff ffff ffff ffff ffff ffff ffff ffff ................
	[REMOVED]
	```

## 25. $Boot file
- MFT entry 7
- contains boot sector and boot code in $DATA
- boot sector fields:

	| Byte Range | Description | Essential | 
	| --- | --- | --- | 
	| 0–2 | Assembly instruction to jump to boot code | No (unless it is the bootable file system) |
	| 3–10 | OEM Name | No | 
	| 11–12 | Bytes per sector | Yes | 
	| 13–13 | Sectors per cluster | Yes | 
	| 14–15 | Reserved sectors (Microsoft says it must be 0) | No | 
	| 16–20 | Unused(Microsoft says it must be 0) | No | 
	| 21–21 | Media descriptor | No | 
	| 22–23 | Unused(Microsoft says it must be  0) | No | 
	| 24–31 | Unused(Microsoft says it is not checked) | No | 
	| 32–35 | Unused(Microsoft says it must be 0) | No | 
	| 36–39 | Unused(Microsoft says it is not checked) | No | 
	| 40–47 | Total sectors in file system | Yes | 
	| 48–55 | Starting cluster address of MFT | Yes | 
	| 56–63 | Starting cluster address of MFT Mirror $DATA attribute | No | 
	| 64–64 | Size of file record(MFT entry) | Yes | 
	| 65–67 | Unused | No | 
	| 68–68 | Size of index record | Yes | 
	| 69–71 | Unused | No | 
	| 72–79 | Serial number | No | 
	| 80–83 | Unused | No | 
	| 84–509 | Boot code | No | 
	| 510–511 | Signature(0xaa55) | No | 
	- fields not used correspond to *BIOS Parameter Block(BPB)* fields in FAT boot sector
	- Windows XP does not mount disk when values are non-zero
	- size of each sector and cluster are required to identify the location of everything
	- address of $DATA in $MFTMirr allows recovery tool to determine location of $MFT backup
	- fields that show size of MFT entry and index record has a special format:
		- if larger than 0: number of clusters for each DS
		- less than 0: minus log base-2 of number of bytes in each DS

## 26. boot sector example
- *icat* output
	``` sh
	$ icat –f ntfs ntfs1.dd 7-128 | xxd
	0000000: eb52 904e 5446 5320 2020 2000 0202 0000 .R.NTFS .....
	0000016: 0000 0000 00f8 0000 3f00 ff00 3f00 0000 ........?...?...
	0000032: 0000 0000 8000 8000 4060 1f00 0000 0000 ........@`......
	0000048: b53a 0500 0000 0000 10d8 0700 0000 0000 .:..............
	0000064: 0100 0000 0400 0000 947c 2250 8422 5004 .........|"P."P.
	0000080: 0000 0000 fa33 c08e d0bc 007c fbb8 c007 .....3.....|....
	0000096: 8ed8 e816 00b8 000d 8ec0 33db c606 0e00 ..........3.....
	[REMOVED]
	0000448: 6d70 7265 7373 6564 000d 0a50 7265 7373 mpressed...Press
	0000464: 2043 7472 6c2b 416c 742b 4465 6c20 746f Ctrl+Alt+Del to
	0000480: 2072 6573 7461 7274 0d0a 0000 0000 0000 restart........
	0000496: 0000 0000 0000 0000 83a0 b3c9 0000 55aa ..............U.
	[REMOVED]
	```
	- bytes 3~10: standard OEM name assigned by Windows
	- bytes 11~12: number of bytes in cluster, 512
	- byte 13: 2 sectors per cluster, 1,024 bytes
	- bytes 40~47: total number of sectors in FS, 2,056,256; FS is 1GB in size
	- bytes 48~55: starting cluster of MFT 342,709
	- bytes 56~63: starting cluster of $DATA of MFT mirror, 514,064
	- byte 64: MFT entry is 1 cluster
	- byte 68: index record is 4 clusters
	- bytes 72~79: serial number of FS, 0x0450 2284 5022 7C94
	- bytes 84~509: boot code
	- bytes 510~511: signature 0xaa55

## 27. $AttrDef file
- MFT entry 4
- defines FS attribute names and identifiers
- $DATA contains list of entries
- entry fields:

	| Byte Range | Description | Essential | 
	| --- | --- | --- | 
	| 0–127 | Name of attribute | Yes | 
	| 128–131 | Type identifier | Yes | 
	| 132–135 | Display rule | No | 
	| 136–139 | Collation rule | No | 
	| 140–143 | Flags | Yes | 
	| 144–151 | Minimum size | No | 
	| 152–159 | Maximum size | No | 
	- collation rule used when attribute is in an index; determines how it should be sorted

- entry flag values:

	| Value | Description | 
	| --- | --- | 
	| 0x02 | Attribute can be used in an index | 
	| 0x04 | Attribute is always resident | 
	| 0x08 | Attribute can be non-resident | 

## 28. $AttrDef example
- *icat* output:
	``` sh
	$ icat -f ntfs ntfs1.dd 4-128 | xxd
	0000000: 2400 5300 5400 4100 4e00 4400 4100 5200 $.S.T.A.N.D.A.R.
	0000016: 4400 5f00 4900 4e00 4600 4f00 5200 4d00 D._.I.N.F.O.R.M.
	0000032: 4100 5400 4900 4f00 4e00 0000 0000 0000 A.T.I.O.N.......
	0000048: 0000 0000 0000 0000 0000 0000 0000 0000 ................
	0000064: 0000 0000 0000 0000 0000 0000 0000 0000 ................
	0000080: 0000 0000 0000 0000 0000 0000 0000 0000 ................
	0000096: 0000 0000 0000 0000 0000 0000 0000 0000 ................
	0000112: 0000 0000 0000 0000 0000 0000 0000 0000 ................
	0000128: 1000 0000 0000 0000 0000 0000 4000 0000 ............@...
	0000144: 3000 0000 0000 0000 4800 0000 0000 0000 0.......H.......
	[REMOVED]
	```
	- first attribute definition for $STANDARD_INFORMATION
	- bytes 128~131: type of attribute 16
	- bytes 140~143: entry always resident
	- bytes 144~151: min size 48 bytes
	- bytes 152~159: max size 72

## 29. $Bitmap file
- MFT entry 6
- $DATA used to manage allocation status of clusters
floor(cluster address) = (bitmap byte)
rem(cluster address) = (bitmap bit)

![[cluster address to bitmap address.png]]

## 30. $Volume file
- MFT entry 3
- 2 unique attributes:
	- $VOLUME_NAME
	- $VOLUME_INFORMATION

## 31. $VOLUME_NAME
- type ID 96
- allocated only to $Volume
- contains only name of volume in UTF-16 unicode
- *icat* output:
	``` sh
	$ icat -f ntfs ntfs1.dd 3-96 | xxd
	0000000: 4e00 5400 4600 5300 2000 4400 6900 7300 N.T.F.S. .D.i.s.
	0000016: 6b00 2000 3200                          k. .2.
	```

## 32. $VOLUME_INFORMATION
- type ID 112
- contains version of FS
- fields:

	| Byte Range | Description | Essential | 
	| --- | --- | --- | 
	| 0–7 | Unused | No | 
	| 8–8 | Major version | Yes | 
	| 9–9 | Minor version | Yes | 
	| 10–11 | Flags | No | 

- flag values:

	| Flag | Description | 
	| --- | --- | 
	| 0x0001 | Dirty | 
	| 0x0002 | Resize $LogFile(file system journal) | 
	| 0x0004 | Upgrade volume next time | 
	| 0x0008 | Mounted in NT | 
	| 0x0010 | Deleting change journal | 
	| 0x0020 | Repair object IDs | 
	| 0x8000 | Modified by chkdsk | 
- *icat* output:
	``` sh
	$ icat -f ntfs ntfs1.dd 3-112 | xxd
	0000000: 0000 0000 0000 0000 0301 0000 ............
	```

## 33. $ObjId file
- \$Extend\$ObjId has index called $O that correlates file's object ID to MFT entry
- $ObjId file is not typically located in reserved MFT entry
- index has typical $INDEX_ROOT and $INDEX_ALLOCATION
- entry fields:

	| Byte Range | Description | Essential | 
	| --- | --- | --- | 
	| 0–1 | Offset to file information | Yes | 
	| 2–3 | Size of file information | Yes | 
	| 4–7 | Unused | No | 
	| 8–9 | Size of index entry | Yes | 
	| 10–11 | Size of object ID(16-bytes) | Yes | 
	| 12–15 | Flags | Yes | 
	| 16–31 | Object ID | Yes | 
	| 32–39 | File reference | Yes | 
	| 40–55 | Birth volume ID | No | 
	| 56–71 | Birth object ID | No |
	| 72–87 | Birth domain ID | No |
	- flag field has 0x01 when child node exists and 0x02 when last entry in index entry list

## 34. $Quota file
- \$Extend\$Quota is used by user quota feature
- not located in rserved MFT entry
- contains 2 indexes that both use standard $INDEX_ROOT and $INDEX_ALLOCATION to store index entries
- $O index correlates SID to owner ID
- $Q index correlates owner ID to quota information
- $O index entry fields:

	| Byte Range | Description | Essential | 
	| --- | --- | --- | 
	| 0–1 | Offset to owner ID(OFF) | Yes | 
	| 2–3 | Length of owner ID | Yes | 
	| 4–7 | Unused | No | 
	| 8–9 | Size of index entry | Yes | 
	| 10–11 | Size of SID(L) | Yes | 
	| 12–15 | Flags | Yes | 
	| 16–(16+L-1) | SID | Yes | 
	| OFF+ | Owner ID | Yes | 
	- if child exists, last 8 bytes are used as VCN of child
- *icat* output:
	``` sh
	0000000: 1c00 0400 0000 0000 2000 0c00 0000 0000 ........ .......
	0000016: 0101 0000 0000 0005 1200 0000 0401 0000 ................
	0000032: 1c00 0400 0000 0000 2000 0c00 0000 0000 ........ .......
	0000048: 0101 0000 0000 0005 1300 0000 0301 0000 ................
	[REMOVED]
	```
	- bytes 0~1: owner ID located at offset 28 from start of entry
	- bytes 2~3: owner ID 4 bytes
	- bytes 8~9: index entry 32 bytes
	- bytes 10~11: SID is 12 bytes
	- bytes 16~27: SID
	- bytes 28~31: owner ID
	- second entry starts at byte 32, owner ID found in bytes 60~63

- $Q index entry fields

	| Byte Range | Description | Essential |
	| --- | --- | --- |
	| 0–1 | Offset to quota information | Yes |
	| 2–3 | Size of quota information | Yes |
	| 4–7 | Unused | No |
	| 8–9 | Size of index entry | Yes |
	| 10–11 | Size of owner ID(4 bytes) | Yes |
	| 12–15 | Flags | Yes |
	| 16–19 | Owner ID | Yes |
	| 20–23 | Version | No |
	| 24–27 | Quota flags | Yes |
	| 28–35 | Bytes charged to user | Yes |
	| 36–43 | Time of last charge | No |
	| 44–51 | Threshold value(a soft limit) | Yes |
	| 52–59 | Hard limit value | Yes |
	| 60–67 | Exceeded time | Yes |
	| 68–79 | SID | Yes |
- quota flag values:

	| Flag | Description | 
	| --- | --- | 
	| 0x00000001 | Default limits being used | 
	| 0x00000002 | Limit reached | 
	| 0x00000004 | ID deleted | 
	| 0x00000010 | Tracking data usage | 
	| 0x00000020 | Enforcing data usage | 
	| 0x00000040 | Usage tracking requested | 
	| 0x00000080 | Create log when threshold is met | 
	| 0x00000100 | Create log when limit is met | 
	| 0x00000200 | Out of date | 
	| 0x00000400 | Corrupt | 
	| 0x00000800 | Pending deletes | 
- *icat* output:
	``` sh
	0000000: 1400 3c00 0000 0000 5000 0400 0000 0000 ..<.....P.......
	0000016: 0301 0000 0200 0000 0100 0000 0028 0500 .............(..
	0000032: 0000 0000 401b 7c3c 7751 c401 ffff ffff ....@.|<wQ......
	0000048: ffff ffff ffff ffff ffff ffff 0000 0000 ................
	0000064: 0000 0000 0101 0000 0000 0005 1300 0000 ................
	0000080: 1400 3c00 0000 0000 5000 0400 0000 0000 ..<.....P.......
	0000096: 0401 0000 0200 0000 0100 0000 0094 6602 ..............f.
	0000112: 0000 0000 90fe 8bdf d769 c401 ffff ffff .........i......
	0000128: ffff ffff ffff ffff ffff ffff 0000 0000 ................
	0000144: 0000 0000 0101 0000 0000 0005 1200 0000 ................
	```
	- bytes 0~1: offset to quota information 20 bytes
	- bytes 2~3: 60 bytes of quota information
	- bytes 16~19: for owner ID 259
	- bytes 24~27: quota flags
	- bytes 28~35: user hasonly 337,920 charged to account

## 35. $LogFile file
- MFT entry 2
- used as NTFS journal
- has standard file attributes
- stores log data in $DATA
- exact DS unknown
- log details: 
	- 4,096 byte pages
	- first two for restart area; RSTR signature in first 4 bytes
		``` sh
		$ icat -f ntfs ntfs1.dd 2 | xxd | grep RSTR
		0000000: 5253 5452 1e00 0900 0000 0000 0000 0000 RSTR............
		0004096: 5253 5452 1e00 0900 0000 0000 0000 0000 RSTR............
		```
	- records come after restart area
		``` sh
		$ icat –f ntfs ntfs1.dd 2 | xxd | grep RCRD
		0008192: 5243 5244 2800 0900 0050 2500 0000 0000 RCRD(....P%.....
		0012288: 5243 5244 2800 0900 0050 2500 0000 0000 RCRD(....P%.....
		[REMOVED]
		```
	- possible to see resident data attributes in log file:
		``` sh
		2215936: 5243 5244 2800 0900 f93b 9403 0000 0000 RCRD(....;......
		[REMOVED]
		2217312: 3801 1800 0000 0000 ec12 0000 0000 0000 8...............
		2217328: a14d 0500 0000 0000 4d79 206e 6577 2066 .M......My new f
		2217344: 696c 652c 2063 616e 2079 6f75 2073 6565 ile, can you see
		2217360: 2069 743f 0000 0000 0000 0000 0000 0000 it?............
		2217376: 0000 0000 0000 0000 0000 0000 0000 0000 ................
		[REMOVED]
		2217808: 0003 0000 0094 7c22 5010 0000 004e 5446 ......|"P....NTF
		2217824: 5320 4469 736b 2032 0043 3a5c 6c6f 672d S Disk 2.C:\log-
		2217840: 7465 7374 2e74 7874 0000 1500 2e00 2e00 test.txt........
		2217856: 5c00 2e00 2e00 5c00 2e00 2e00 5c00 6c00 \.....\.....\.l.
		2217872: 6f00 6700 2d00 7400 6500 7300 7400 2e00 o.g.-.t.e.s.t...
		2217888: 7400 7800 7400 0300 4300 3a00 5c00 6000 t.x.t...C.:.\.`.
		```

## 36. $UsrJrnl file
- changes are recorded in $DATA named $J of \$Extend\$UsrJrnl
- not located in reserved MFT entry
- $J is sparse and contains list of different sized DS, called *journal entries*
- $DATA called $Max that contains information about max settings for user journal also exists
- $J fields:

	| Byte Range | Description | Essential | 
	| --- | --- | --- | 
	| 0–3 | Size of this journal entry | Yes | 
	| 4–5 | Major version | Yes | 
	| 6–7 | Minor version | Yes | 
	| 8–15 | File reference of file that caused this entry | Yes | 
	| 16–23 | Parent directory file reference for file that caused this entry | No | 
	| 24–31 | USN for entry | Yes | 
	| 32–39 | Timestamp | Yes | 
	| 40–43 | Flags for type of change | Yes | 
	| 44–47 | Source information | No | 
	| 48–51 | Security ID(SID) | No | 
	| 52–55 | File attributes | No | 
	| 56–57 | Size of file name | Yes | 
	| 58+ | file name | yes | 
	- essential column refers to whether data is essential for providing a log of file changes
	- source byte typically 0; can be non-zero if OS caused entry to be made
- $J entry flag values:

	| Value | Description | 
	| --- | --- | 
	| 0x00000001 | The default $DATA attribute was overwritten | 
	| 0x00000002 | The default $DATA attribute was extended | 
	| 0x00000004 | The default $DATA attribute was truncated | 
	| 0x00000010 | A named $DATA attribute was overwritten | 
	| 0x00000020 | A named $DATA attribute was extended | 
	| 0x00000040 | A named $DATA attribute was truncated | 
	| 0x00000100 | The file or directory was created | 
	| 0x00000200 | The file or directory was deleted | 
	| 0x00000400 | The extended attributes of the file were changed | 
	| 0x00000800 | The security descriptor was changed | 
	| 0x00001000 | The name changed—change journal entry has old name | 
	| 0x00002000 | The name changed—change journal entry has new name | 
	| 0x00004000 | Content indexed status changed | 
	| 0x00008000 | Changed basic file or directory attributes | 
	| 0x00010000 | A hard link was created or deleted | 
	| 0x00020000 | Compression status changed | 
	| 0x00040000 | Encryption status changed | 
	| 0x00080000 | Object ID changed | 
	| 0x00100000 | Reparse point value changed | 
	| 0x00200000 | A named $DATA attribute was created, deleted, or changed | 
	| 0x80000000 | The file or directory was closed | 

- $Max contains general change journal administrative information
- $Max fields:

	| Byte Range | Description | Essential |
	| --- | --- | --- |
	| 0–7 | Maximum size | Yes |
	| 8–15 | Allocation size | Yes |
	| 16–23 | USN ID | Yes |
	| 24–31 | Lowest USN | Yes |
- $Max *icat* output:
	``` sh 
	$ icat –f ntfs ntfs3.dd 27-128-4 | xxd
	0000000: 0000 8000 0000 0000 0000 1000 0000 0000 ................
	0000016: 4057 7491 eb69 c401 0000 0000 0000 0000 @Wt..i..........
	```

## 37. $UserJrnl $J example
- *icat* output:
	``` sh
	$ icat –f ntfs ntfs3.dd 27-128-3 | xxd
	0000000: 5000 0000 0200 0000 1c00 0000 0000 0100 P...............
	0000016: 0500 0000 0000 0500 0000 0000 0000 0000 ................
	0000032: 3000 e2b9 eb69 c401 0001 0000 0000 0000 0....i..........
	0000048: 0201 0000 2000 0000 1400 3c00 6600 6900 .... .....<.f.i.
	0000064: 6c00 6500 2d00 3000 2e00 7400 7800 7400 l.e.-.0...t.x.t.
	```
	- bytes 0~3: 80 bytes long
	- bytes 8~13: entry is for MFT entry 28
	- bytes 24~31: entry has USN of 0
	- bytes 40~43: for overwriting default $DATA
	- entry is for file file-o.txt
