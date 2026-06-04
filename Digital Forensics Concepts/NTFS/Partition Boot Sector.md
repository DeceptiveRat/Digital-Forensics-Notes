## Boot Sector
|Byte Offset|Field Length|Field Name|
|---|---|---|
|0x00|3 bytes|Jump Instruction|
|0x03|LONGLONG|OEM ID|
|0x0B|25 bytes|BPB|
|0x24|48 bytes|Extended BPB|
|0x54|426 bytes|Bootstrap Code|
|0x01FE|WORD|End of Sector Marker|

### BPB and Extended BPB fields
|Byte Offset|Field Length|Sample Value|Field Name|
|---|---|---|---|
|0x0B|WORD|0x0002|Bytes Per Sector|
|0x0D|BYTE|0x08|Sectors Per Cluster|
|0x0E|WORD|0x0000|Reserved Sectors|
|0x10|3 BYTES|0x000000|_always 0_|
|0x13|WORD|0x0000|_not used by NTFS_|
|0x15|BYTE|0xF8|Media Descriptor|
|0x16|WORD|0x0000|_always 0_|
|0x18|WORD|0x3F00|Sectors Per Track|
|0x1A|WORD|0xFF00|Number Of Heads|
|0x1C|DWORD|0x3F000000|Hidden Sectors|
|0x20|DWORD|0x00000000|_not used by NTFS_|
|0x24|DWORD|0x80008000|_not used by NTFS_|
|0x28|LONGLONG|0x4AF57F0000000000|Total Sectors|
|0x30|LONGLONG|0x0400000000000000|Logical Cluster Number for the file $MFT|
|0x38|LONGLONG|0x54FF070000000000|Logical Cluster Number for the file $MFTMirr|
|0x40|DWORD|0xF6000000|Clusters Per File Record Segment|
|0x44|DWORD|0x01000000|Clusters Per Index Block|
|0x48|LONGLONG|0x14A51B74C91B741C|Volume Serial Number|
|0x50|DWORD|0x00000000|Checksum|