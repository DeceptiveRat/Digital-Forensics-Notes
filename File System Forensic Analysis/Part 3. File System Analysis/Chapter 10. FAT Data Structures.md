## 1. boot sector
- located in first sector of FAT FS
- contains bulk of FS category data
- FAT12/16 and FAT32 have different versions, but identical initial 36 bytes:

	| Byte Range | Description | Essential |
	| --- | --- | --- |
	| 0–2 | Assembly instruction to jump to boot code. | No (unless it is a bootable file system) |
	| 3–10 | OEM Name in ASCII. | No |
	| 11–12 | Bytes per sector. Allowed values include 512, 1024, 2048, and 4096. | Yes |
	| 13–13 | Sectors per cluster(data unit). Allowed values are powers of 2, but the cluster size must be 32KB or smaller. | Yes |
	| 14–15 | Size in sectors of the reserved area. | Yes |
	| 16–16 | Number of FATs. Typically two for redundancy, but according to Microsoft it can be one for some small storage devices. | Yes |
	| 17–18 | Maximum number of files in the root directory for FAT12 and FAT16. This is 0 for FAT32 and typically 512 for FAT16. | Yes |
	| 19–20 | 16-bit value of number of sectors in file system. If the number of sectors is larger than can be represented in this 2-byte value, a 4-byte value exists later in the data structure and this should be 0. | Yes |
	| 21–21 | Media type. According to the Microsoft documentation, 0xf8 should be used for fixed disks and 0xf0 for removable. | No |
	| 22–23 | 16-bit size in sectors of each FAT for FAT12 and FAT16. For FAT32, this field is 0. | Yes |
	| 24–25 | Sectors per track of storage device. | No |
	| 26–27 | Number of heads in storage device. | No |
	| 28–31 | Number of sectors before the start of partition. | No |
	| 32-35 | 32-bit value of number of sectors in file system. Either this value or the 16-bit value above must be 0 | Yes |

	- media type value is used to identify if file system is on fixed or removable media; Windows uses copy in FAT instead

- FAT12/16 remainder:

	| Byte Range | Description | Essential |
	| --- | --- | --- |
	| 0–35 | See Table 10.1. | Yes |
	| 36–36 | BIOS INT13h drive number. | No |
	| 37–37 | Not used. | No |
	| 38–38 | Extended boot signature to identify if the next three values are valid. The signature is 0x29. | No |
	| 39–42 | Volume serial number, which some versions of Windows will calculate based on the creation date and time. | No |
	| 43–53 | Volume label in ASCII. The user chooses this value when creating the file system. | No |
	| 54–61 | File system type label in ASCII. Standard values include "FAT," "FAT12," and "FAT16," but nothing is required. | No |
	| 62–509 | Not used. | No |
	| 510–511 | Signature value(0xAA55). | No |

- FAT32 remainder:

	| Byte Range | Description | Essential |
	| --- | --- | --- |
	| 0–35 | See Table 10.1. | Yes |
	| 36–39 | 32-bit size in sectors of one FAT. | Yes |
	| 40–41 | Defines how multiple FAT structures are written to. If bit 7 is 1, only one of the FAT structures is active and its index is described in bits 0–3. Otherwise, all FAT structures are mirrors of each other. | Yes |
	| 42–43 | The major and minor version number. | Yes |
	| 44–47 | Cluster where root directory can be found. | Yes |
	| 48–49 | Sector where FSINFO structure can be found. | No |
	| 50–51 | Sector where backup copy of boot sector is located(default is 6). | No |
	| 52–63 | Reserved. | No |
	| 64–64 | BIOS INT13h drive number. | No |
	| 65–65 | Not used. | No |
	| 66–66 | Extended boot signature to identify if the next three values are valid. The signature is 0x29. | No |
	| 67–70 | Volume serial number, which some versions of Windows will calculate based on the creation date and time. | No |
	| 71–81 | Volume label in ASCII. The user chooses this value when creating the file system. | No |
	| 82–89 | File system type label in ASCII. Standard values include "FAT32," but nothing is required. | No |
	| 90–509 | Not used. | No |
	| 510-511 | Signature value (0xAA55). | No |
- bytes 62-509(FAT12/16) and 90-509(FAT32) typically store boot code and error messages
- certain Windows versions assign volume serial number using FS creation date and time values
	![[volume serial number generation.png]]

## 2. FAT32 boot sector hex dump analysis
``` sh
$ dcat –f fat fat-4.dd 0 | xxd
0000000: eb58 904d 5344 4f53 352e 3000 0202 2600 .X.MSDOS5.0...&.
0000016: 0200 0000 00f8 0000 3f00 4000 c089 0100 ........?.@.....
0000032: 4023 0300 1d03 0000 0000 0000 0200 0000 @#..............
0000048: 0100 0600 0000 0000 0000 0000 0000 0000 ................
0000064: 8000 2903 4619 4c4e 4f20 4e41 4d45 2020 ..).F.LNO NAME
0000080: 2020 4641 5433 3220 2020 33c9 8ed1 bcf4 FAT32 3.....
0000096: 7b8e c18e d9bd 007c 884e 028a 5640 b408 {......|.N..V@..
0000112: cd13 7305 b9ff ff8a f166 0fb6 c640 660f ..s......f...@f.
0000128: b6d1 80e2 3ff7 e286 cdc0 ed06 4166 0fb7 ....?.......Af..
[REMOVED]
0000416: 0000 0000 0000 0000 0000 0000 0d0a 5265 ..............Re
0000432: 6d6f 7665 2064 6973 6b73 206f 7220 6f74 move disks or ot
0000448: 6865 7220 6d65 6469 612e ff0d 0a44 6973 her media....Dis
0000464: 6b20 6572 726f 72ff 0d0a 5072 6573 7320 k error...Press
0000480: 616e 7920 6b65 7920 746f 2072 6573 7461 any key to resta
0000496: 7274 0d0a 0000 0000 00ac cbd8 0000 55aa rt............U.
```
- first line shows OEM name: MSDOS5.0
- data is written in little endian order
- bytes 11~12: each sector is 512 bytes
- byte 13: size of each cluster is 2 sectors
- bytes 14~15: there are 38 sectors in reserved area
- byte 16: there are 2 FAT structures
- bytes 19~20: FS size too large for 2 bytes
- bytes 28~31: 100,800 sectors before start of file system
- bytes 32~35: FS size 205,632 sectors
- bytes 36~39: size of each FAT structure is 797 sectors
- bytes 48~49: FSINFO located in sector 1
- bytes 50~51: backup copy of boot sector in sector 6
- bytes 67~70: volume serial number, 0x4c19 4603
- bytes 71~81: volume label, NO NAME
- bytes 82~89: type label, FAT32
- bytes 90~509: data used if system boots this FS
- bytes 510~511: signature 0xAA55 value

## 3. FAT32 FSINFO
| Byte Range | Description | Essential |
| --- | --- | --- |
| 0–3 | Signature(0x41615252) | No |
| 4–483 | Not used | No |
| 484–487 | Signature(0x61417272) | No |
| 488–491 | Number of free clusters | No |
| 492–495 | Next free cluster | No |
| 496–507 | Not used | No |
| 508–511 | Signature(0xAA550000) | No |

## 4. FAT32 FSINFO example
``` sh 
$ dcat –f fat fat-4.dd 1 | xxd
0000000: 5252 6141 0000 0000 0000 0000 0000 0000 RRaA............
0000016: 0000 0000 0000 0000 0000 0000 0000 0000 ................
[REMOVED]
0000464: 0000 0000 0000 0000 0000 0000 0000 0000 ................
0000480: 0000 0000 7272 4161 1e8e 0100 4b00 0000 ....rrAa....K...
0000496: 0000 0000 0000 0000 0000 0000 0000 55aa ..............U.
```
- bytes 0~3: signature
- bytes 484~487: signature
- bytes 488~491: number of free clusters, 101,918
- bytes 492~495: next free cluster, 75
- bytes 508~511: signature

## 5. FAT
- used to determine allocation status of a cluster
- used to chain clusters
- typically two FATs exist in a FAT FS
- consists of equal-sized entries with no header or footer values
- size of entry depends on FS version
- unallocated cluster will have 0 value, allocated cluster will have address of next cluster in chain or EOF
- EOF values: 
	- FAT12: 0x0ff8~
	- FAT16: 0xfff8~
	- FAT32: 0x0fff fff8~
- damaged cluster value:
	- FAT12: 0x0ff7
	- FAT16: 0xfff7
	- FAT32: 0x0fff fff7
- entry 0 typically stores copy of media type; non-essential
- entry 1 typically stores dirty status of FS; denotes improper unmount or improper shutdown or hardware surface errors; non-essential

## 6. FAT example
``` sh
$ dcat –f fat fat-4.dd 38 | xxd
[REMOVED]
0000288: 4900 0000 4a00 0000 4c00 0000 0000 0000 I...J...L.......
0000304: 4d00 0000 ffff ff0f 4f00 0000 ffff ff0f M.......O.......
0000320: 5100 0000 5200 0000 ffff ff0f ffff ff0f Q...R...........
0000336: ffff ff0f 0000 0000 0000 0000 0000 0000 ................
0000352: 0000 0000 0000 0000 0000 0000 0000 0000 ................
```
- entry at byte 288 is entry 72 (288/4)
- entry 72 contins value 73
``` sh
$ dstat -f fat fat-4.dd 1778
Sector: 1778
Not Allocated
Cluster: 75
```

## 7. directory entry
| Byte Range | Description | Essential |
| --- | --- | --- |
| 0–0 | First character of file name in ASCII and allocation status (0xe5 or 0x00 if unallocated) | Yes |
| 1–10 | Characters 2 to 11 of file name in ASCII | Yes |
| 11–11 | File Attributes | Yes |
| 12–12 | Reserved | No |
| 13–13 | Created time(tenths of second) | No |
| 14–15 | Created time(hours, minutes, seconds) | No |
| 16–17 | Created day | No |
| 18–19 | Accessed day | No |
| 20–21 | High 2 bytes of first cluster address(0 for FAT12 and FAT16) | Yes |
| 22–23 | Written time(hours, minutes, seconds) | No |
| 24–25 | Written day | No |
| 26–27 | Low 2 bytes of first cluster address | Yes |
| 28–31 | Size of file(0 for directories) | Yes |
- unused name bytes are typically filled with 0x20, ASCII for space
- max file size is limited to 4GB
- flag values for attribute field

	| Flag | Value(in bits) | Description | Essential |
	| --- | --- | --- | --- |
	| 0000 0001(0x01) | Read only | No |
	| 0000 0010(0x02) | Hidden file | No |
	| 0000 0100(0x04) | System file | No |
	| 0000 1000(0x08) | Volume label | Yes |
	| 0000 1111(0x0f) | Long file name | Yes |
	| 0001 0000(0x10) | Directory | Yes |
	| 0010 0000(0x20) | Archive | No |

- directory entries with long file name attributes have different structures
- FAT date format
	![[directory entry date stamp format.png]]
- FAT time format
	![[directory entry time stamp format.png]]

## 8. directory entry example
``` sh 
$ dcat –f fat fat-4.dd 1632 | xxd
0000000: 4641 5420 4449 534b 2020 2008 0000 0000 FAT DISK .....
0000016: 0000 0000 0000 874d 252b 0000 0000 0000 .......M%+......
0000032: 5245 5355 4d45 2d31 5254 4620 00a3 347e RESUME-1RTF ..4~
0000048: 4a30 8830 0000 4a33 7830 0900 f121 0000 .0.0.....0...!..
```
- first entry has volume label set
- bytes 22~25: write time and date
- second entry is for file RESUME-1.RTF
- byte 43: archive attribute is set
- byte 45: tenths of a second for create time
- bytes 46~47: created time
- bytes 48~49: created date
- bytes 52~53, 58~59: starting cluster is 9
	``` sh 
	$ dcat –f fat fat-4.dd 38 | xxd
	[REMOVED]
	0000032: ffff ff0f 0a00 0000 0b00 0000 0c00 0000 ................
	0000048: 0d00 0000 0e00 0000 0f00 0000 1000 0000 ................
	0000064: 1100 0000 ffff ff0f 1300 0000 1400 0000 ................
	```
	- cluster chain: 9~17
	- 9 clusters of 1,024 byte clusters = 9,216 bytes
- bytes 60~63: file size 8,689 bytes
- *fsstat* for FAT content
	``` sh
	$ fsstat –f fat fat-4.dd
	[REMOVED]
	1642-1645 (4) -> EOF
	1646-1663 (18) -> EOF
	1664-1681 (18) -> EOF
	[REMOVED]
	```
	- cluster 9 at sector 1,646 contains 18 sectors (9 clusters)
- *istat* for details on directory entry
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

## 9. LFN directory entry
| Byte Range | Description | Essential |
| --- | --- | --- |
| 0–0 | Sequence number(final number is ORed with 0x40) and allocation status (0xe5 if unallocated) | Yes | 
| 1–10 | File name characters 1–5(Unicode) | Yes | 
| 11–11 | File attributes(0x0f) | Yes | 
| 12–12 | Reserved | No | 
| 13–13 | Checksum | Yes | 
| 14–25 | File name characters 6–11(Unicode) | Yes | 
| 26–27 | Reserved | No | 
| 28–31 | File name characters 12–13(Unicode) | Yes | 
- first LFN entry has the largest sequence value
- unused characters are padded with 0xff
- name should be NULL-terminated if possible
- file attribute must be 0x0f
- checksum is calculated by:
	``` c
	c = 0;
	for (i = 0; i < 11; i++) {
	// Rotate c to the right
	c = ((c & 0x01) ? 0x80 : 0) + (c >> 1);
	// Add ASCII character from name
	c = c + shortname[i];
	}
	```

## 10. LFN directory entry example
``` sh
$ dcat –f fat fat-4.dd 1632 | xxd
[REMOVED]
0000064: 424e 0061 006d 0065 002e 000f 00df 7200 BN.a.m.e......r.
0000080: 7400 6600 0000 ffff ffff 0000 ffff ffff t.f.............
0000096: 014d 0079 0020 004c 006f 000f 00df 6e00 .M.y. .L.o....n.
0000112: 6700 2000 4600 6900 6c00 0000 6500 2000 g. .F.i.l...e. .
0000128: 4d59 4c4f 4e47 7e31 5254 4620 00a3 347e MYLONG~1RTF ..4~
0000144: 4a30 8830 0000 4a33 7830 1a00 8f13 0000 J0.0..J3x0......
```
- byte 64: sequence number of 0x42; there are 2 LFN entries
- bytes 65~74: first part of name, "Name."
- byte 75: first entry is LFN entry
- byte 77: checksum of 0xdf
- bytes 78~89: second part of name, "rtf."
- byte 96: sequence number of 1
- bytes 97~106: first part of name, "My Lo"
- byte 107: second entry is LFN entry
- byte 109: checksum of 0xdf
- bytes 110~121: second part of name, "ng Fil"
- bytes 124~127: last part of name, "e "
- bytes 128~138: short version of name: MYLONG~1RTF
- *fls* output:
	``` sh
	$ fls -f fat fat-2.dd
	r/r 3: FAT DISK (Volume Label Entry)
	r/r 4: RESUME-1.RTF
	r/r 7: My Long File Name.rtf (MYLONG~1.RTF)
	r/r * 8: _ile6.txt
	```
