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
