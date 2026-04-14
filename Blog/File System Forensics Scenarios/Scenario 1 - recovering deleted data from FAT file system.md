## 1. Scenario
While Alice was having lunch in the cafeteria, some of her friends deleted everything on her USB! She is very forgetful and can't do anything without her todo list. Help her recover the USB contents. 

According to Alice, the contents in the USB are as follows:
1. a directory containing pictures.
2. a txt file with her homework written inside. 
3. a txt file with her todo list written inside. 

## 2. Recovery
### 2.1. creating USB image
First, I will create a disk image for analysis.

``` sh
$ lsblk
[REMOVED]
sdc           8:32   1   3.8G  0 disk
└─sdc1        8:33   1   3.8G  0 part /media/workstation/USB_PART
[REMOVED]
$ sudo dd if=/dev/sdc of=usb.dd bs=512 status=progress
4022963200 bytes (4.0 GB, 3.7 GiB) copied, 158 s, 25.5 MB/s
7897088+0 records in
7897088+0 records out
4043309056 bytes (4.0 GB, 3.8 GiB) copied, 158.798 s, 25.5 MB/s
```

### 2.2. identifying partition
Now that we have the image, we have to identify the partition. Let's check the first MBR entry.

``` sh
$ xxd -s 446 -l 16 usb.dd
000001be: 0021 0300 0c7a d8fa 0008 0000 0078 7800  .!...z.......xx.
```

The first MBR entry is not of type `0xEE` which would indicate a GPT partition.

The first partition in the image is:
- byte 0: not bootable
- bytes 1~3: starts at CHS address `0x000321`
- byte 4: partition type `0x0c`, indiciating a FAT32 File System with LBA<sup>[1]</sup>
- bytes 5~7: ending CHS address `0xfad87a`
- bytes 8~11: starting LBA address `0x00000800`
- bytes 12~15: `0x00787800` sectors long; 4042260480 bytes long

Let's view the next MBR entry.

``` sh
$ xxd -s 462 -l 16 usb.dd
000001ce: 0000 0000 0000 0000 0000 0000 0000 0000  ................
$ xxd -s 478 -l 16 usb.dd
000001de: 0000 0000 0000 0000 0000 0000 0000 0000  ................
$ xxd -s 494 -l 16 usb.dd
000001ee: 0000 0000 0000 0000 0000 0000 0000 0000  ................
```

There are no more MBR entries. This means there is only 1 partition and it is a FAT32 File System starting at sector offset 2048.

We can confirm this with the automated tool *mmls*.

``` sh
$ mmls usb.dd
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

      Slot      Start        End          Length       Description
000:  Meta      0000000000   0000000000   0000000001   Primary Table (#0)
001:  -------   0000000000   0000002047   0000002048   Unallocated
002:  000:000   0000002048   0007897087   0007895040   Win95 FAT32 (0x0c)
```

### 2.3. parsing FAT32 boot sector
Now we have to parse the FAT32 boot sector. 

``` sh
$ dd if=usb.dd skip=2048 count=1 status=none | xxd -l 2 -s 11
0000000b: 0002                                     ..
$ dd if=usb.dd skip=2048 count=1 status=none | xxd -l 1 -s 13
0000000d: 08                                       .
```

Each sector is 512 bytes long and each cluster contains 8 sectors.

``` sh
$ dd if=usb.dd skip=2048 count=1 status=none | xxd -l 2 -s 14
0000000e: 2000                                      .
```

32 sectors are reserved.

``` sh
$ dd if=usb.dd skip=2048 count=1 status=none | xxd -l 1 -s 16
00000010: 02                                       .
```

There are 2 FAT structures

``` sh
$ dd if=usb.dd skip=2048 count=1 status=none | xxd -l 2 -s 19
00000013: 0000                                     ..
$ dd if=usb.dd skip=2048 count=1 status=none | xxd -l 4 -s 32
00000020: ea77 7800                                .wx.
```

There are 7895018 sectors in this File System. Note how this is different from what is said on the MBR entry, which is 7895040 sectors. This means the last 22 sectors in this File System are in the partition slack space.

``` sh
$ dd if=usb.dd skip=2048 count=1 status=none | xxd -l 4 -s 36
00000024: 101e 0000                                ....
```

Each FAT is 7696 sectors in size. 

``` sh
$ dd if=usb.dd skip=2048 count=1 status=none | xxd -l 2 -s 40
00000028: 0000                                     ..
```

Both FAT structures are mirrors of each other.

``` sh
$ dd if=usb.dd skip=2048 count=1 status=none | xxd -l 4 -s 44
0000002c: 0200 0000                                ....
```

The root directory can be found in cluster 2.

``` sh
$ dd if=usb.dd skip=2048 count=1 status=none | xxd -l 2 -s 48
00000030: 0100                                     ..
```

The FSINFO structure can be found in sector 1.

``` sh
$ dd if=usb.dd skip=2048 count=1 status=none | xxd -l 2 -s 50
00000032: 0600                                     ..
```

The boot sector is backed up in sector 6.

### 2.4. finding root directory
The root directory is in cluster 2, meaning it is at the start of the data area. 

To get the start of the data area, we need the size of the reserved area and FAT area. The size of the reserved area is 32 sectors as we found out earlier. The size of the FAT area can be calculated by multiplying the number of FAT structures and their size. There are 2 FATs each with 7696 sectors, which means the FAT area is 15392 sectors long. Adding this to the reserved area size, we get 15424, which is the sector offset of the data area from the beginning of the File System.

It is important to note 15424 is not the sector offset from the beginning of the disk. The sector offset from the beginning of the disk can be calculated by adding the sector offset of the File System, which is 2048, giving us 17472.

### 2.5. recovering deleted files

``` sh
$ dd if=usb.dd skip=17472 count=1 status=none | xxd
00000000: 5553 425f 5041 5254 2020 2008 0000 e95c  USB_PART   ....\
00000010: 895c 895c 0000 e95c 895c 0000 0000 0000  .\.\...\.\......
00000020: e568 006f 006d 0065 0077 000f 0043 6f00  .h.o.m.e.w...Co.
00000030: 7200 6b00 2e00 7400 7800 0000 7400 0000  r.k...t.x...t...
[REMOVED]
```

First directory entry is the volume label, which is not relevant to our purposes and can be ignored. 

The second directory entry has an attribute value of `0x0f` at byte 11. This means it is a *Long File Name(LFN)* entry. Parsing it accordingly, we can get the following information:
- byte 0: `0xe5` meaning it is unallocated
- bytes 1~10: string containing the first 5 characters; "homew"
- byte 11: attribute `0x0f` for LFN entry
- byte 12: reserved and empty
- byte 13: checksum `0x43`
- bytes 14~25: string containing characters 6~11; "ork.tx"
- bytes 26~27: reserved and empty
- bytes 28~31: strint containing characters 12~13; "t"

Because this is a LFN entry, a corresponding *Short File Name(SFN)* entry is required to get file metadata such as timestamps, file size, and content address.

``` sh
$ dd if=usb.dd skip=17472 count=1 status=none | xxd -s 64 -l 32
00000040: e54f 4d45 574f 524b 5458 5420 0039 4c5d  .OMEWORKTXT .9L]
00000050: 895c 895c 0000 4c5d 895c 0600 4000 0000  .\.\..L].\..@...
```

The third directory entry contains the following data:
- byte 0: `0xe5` for unallocated entry
- bytes 1~10: string containing characters 2~11 of file name in ASCII; "OMEWORKTXT"
- byte 11: file attribute `0x20`; archive flag set
- byte 12: reserved and empty
- byte 13: created time in tenths of second; `0x39`
- bytes 14~15: created time `0x5d4c`
- bytes 16~17: created date `0x5c89`
- bytes 18~19: accessed date `0x5c89`
- bytes 20~21: high bytes of cluster address `0x0000`
- bytes 22~23: written time `0x5d4c`
- bytes 24~25: written date `0x5c89`
- bytes 26~27: low bytes of cluster addresss `0x0006`
- bytes 28~31: file size `0x00000040`

The date and time stamps should be converted to a more readable format.

``` sh
$ fat_time_converter -d 5c89 -x
2026/4/9
$ fat_time_converter -t 5d4c -x
11:42:24
```

All time stamps can be converted to `2026-04-09 11:42:24.000000 UTC`

To summarize, the first file in the root directory is called "homework.txt", created and acessed on `2026-04-09 11:42:24.000000 UTC`, with data at cluster `0x06`.

Using the cluster address and file size, we can get the contents of "homework.txt". Root directory at cluster 2 is at sector offset 17472 so cluster 6 is at sector offset `17472 + 4*8 = 17504`

``` sh
$ dd if=usb.dd skip=17504 count=1 status=none | xxd -l 64
00000000: 312e 2053 7475 6479 204f 5320 636f 6e63  1. Study OS conc
00000010: 6570 7473 0a32 2e20 5374 7564 7920 4320  epts.2. Study C
00000020: 7072 6f67 7261 6d6d 696e 670a 332e 2053  programming.3. S
00000030: 7475 6479 2041 6c67 6f72 6974 686d 730a  tudy Algorithms.
```

Using the same method, we can get the contents of the next file, which is called "todo.txt" at cluster address `0x0a`.

``` sh
$ dd if=usb.dd skip=17536 count=1 status=none | xxd -l 79
00000000: 312e 2042 7579 204f 7065 7261 7469 6e67  1. Buy Operating
00000010: 2053 7973 7465 6d20 436f 6e63 6570 7473   System Concepts
00000020: 0a32 2e20 5374 7564 7920 666f 7220 7175  .2. Study for qu
00000030: 697a 0a33 2e20 4669 6e69 7368 2057 6565  iz.3. Finish Wee
00000040: 6b20 3320 6173 7369 676e 6d65 6e74 0a    k 3 assignment.
```

The final 2 directory entries can be processed in the same way, too.

``` sh
$ dd if=usb.dd skip=17472 count=1 status=none | xxd -s 160 -l 64
000000a0: e570 0069 0063 0074 0075 000f 0059 7200  .p.i.c.t.u...Yr.
000000b0: 6500 7300 0000 ffff ffff 0000 ffff ffff  e.s.............
000000c0: e549 4354 5552 4553 2020 2010 0047 845d  .ICTURES   ..G.]
000000d0: 895c 895c 0000 d45d 895c 0b00 0000 0000  .\.\...].\......
```

But it is important to note the file attributes of the final directory entry. It is `0x10`, which means it is an entry for a directory, not a file. 

``` sh
$ dd if=usb.dd skip=17544 count=2 status=none | xxd
00000000: 2e20 2020 2020 2020 2020 2010 0047 845d  .          ..G.]
00000010: 895c 895c 0000 845d 895c 0b00 0000 0000  .\.\...].\......
00000020: 2e2e 2020 2020 2020 2020 2010 0047 845d  ..         ..G.]
00000030: 895c 895c 0000 845d 895c cd0c 0000 0000  .\.\...].\......
00000040: e572 0073 0035 002e 006a 000f 00af 7000  .r.s.5...j....p.
00000050: 6700 0000 ffff ffff ffff 0000 ffff ffff  g...............
00000060: e546 0072 0065 0065 0020 000f 00af 5700  .F.r.e.e. ....W.
00000070: 6100 6c00 6c00 7000 6100 0000 7000 6500  a.l.l.p.a...p.e.
00000080: e552 4545 5741 7e31 4a50 4720 00b4 d25d  .REEWA~1JPG ...]
00000090: 895c 895c 0000 bd5d 895c 0c00 38c5 1400  .\.\...].\..8...
000000a0: e572 0073 0031 002e 006a 000f 000f 7000  .r.s.1...j....p.
000000b0: 6700 0000 ffff ffff ffff 0000 ffff ffff  g...............
000000c0: e546 0072 0065 0065 0020 000f 000f 5700  .F.r.e.e. ....W.
000000d0: 6100 6c00 6c00 7000 6100 0000 7000 6500  a.l.l.p.a...p.e.
000000e0: e552 4545 5741 7e32 4a50 4720 0019 d35d  .REEWA~2JPG ...]
000000f0: 895c 895c 0000 bb5d 895c 5901 3931 3f00  .\.\...].\Y.91?.
00000100: e572 0073 0034 002e 006a 000f 00ef 7000  .r.s.4...j....p.
00000110: 6700 0000 ffff ffff ffff 0000 ffff ffff  g...............
00000120: e546 0072 0065 0065 0020 000f 00ef 5700  .F.r.e.e. ....W.
00000130: 6100 6c00 6c00 7000 6100 0000 7000 6500  a.l.l.p.a...p.e.
00000140: e552 4545 5741 7e33 4a50 4720 0071 d35d  .REEWA~3JPG .q.]
00000150: 895c 895c 0000 ba5d 895c 4d05 6599 1800  .\.\...].\M.e...
00000160: e572 0073 0032 002e 006a 000f 00cf 7000  .r.s.2...j....p.
00000170: 6700 0000 ffff ffff ffff 0000 ffff ffff  g...............
00000180: e546 0072 0065 0065 0020 000f 00cf 5700  .F.r.e.e. ....W.
00000190: 6100 6c00 6c00 7000 6100 0000 7000 6500  a.l.l.p.a...p.e.
000001a0: e552 4545 5741 7e34 4a50 4720 009a d35d  .REEWA~4JPG ...]
000001b0: 895c 895c 0000 b95d 895c d706 472e 2300  .\.\...].\..G.#.
000001c0: e572 0073 0033 002e 006a 000f 002f 7000  .r.s.3...j.../p.
000001d0: 6700 0000 ffff ffff ffff 0000 ffff ffff  g...............
000001e0: e546 0072 0065 0065 0020 000f 002f 5700  .F.r.e.e. .../W.
000001f0: 6100 6c00 6c00 7000 6100 0000 7000 6500  a.l.l.p.a...p.e.
00000200: e552 4545 5741 7e35 4a50 4720 0009 d45d  .REEWA~5JPG ...]
00000210: 895c 895c 0000 b85d 895c 0a09 e805 3c00  .\.\...].\....<.
00000220: 0000 0000 0000 0000 0000 0000 0000 0000  ................
[REMOVED]
```

The first 2 entries are entries for "." and ".." which correspond to the current directory and parent directory respectively.

The files in the "pictures" directory can be processed as before, but there are 2 important differences. The first is that there are 2 LFNs for each SFN, and the second is that the file sizes are larger than a cluster, which means the FAT has to be used to traverse the "cluster chain".

Let's check the first FAT structure.

``` sh
$ dd if=usb.dd skip=2080 count=1 status=none | xxd
00000000: f8ff ff0f ffff ff0f f8ff ff0f 0000 0000  ................
00000010: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000020: 0000 0000 0000 0000 0000 0000 0000 0000  ................
[REMOVED]
```

Because the files were deleted, the clusters were marked as free, meaning the cluster chain in the FAT structure was unallocated as well. This means we have to find the picture files without the FAT. 

The first picture in "pictures" is at cluster address `0x0c`. We can get the sector offset of that file like this: `17472+(12-2)*8 = 17552`. Let's see what is in that sector.

``` sh
$ dd if=usb.dd skip=17552 count=1 status=none | xxd
00000000: ffd8 ffe0 0010 4a46 4946 0001 0201 0048  ......JFIF.....H
00000010: 0048 0000 ffe2 0c58 4943 435f 5052 4f46  .H.....XICC_PROF
00000020: 494c 4500 0101 0000 0c48 4c69 6e6f 0210  ILE......HLino..
00000030: 0000 6d6e 7472 5247 4220 5859 5a20 07ce  ..mntrRGB XYZ ..
00000040: 0002 0009 0006 0031 0000 6163 7370 4d53  .......1..acspMS
00000050: 4654 0000 0000 4945 4320 7352 4742 0000  FT....IEC sRGB..
00000060: 0000 0000 0000 0000 0000 0000 f6d6 0001  ................
00000070: 0000 0000 d32d 4850 2020 0000 0000 0000  .....-HP  ......
[REMOVED]
```

As expected, the start of a jpg file is found. If the File System was not fragmented when the pictures were created, they could have been assigned contiguous memory locations.

If this is the case, we do not need the FAT to reconstruct the files. Let's see if this is the case.

Because the file size of the first file is `0x14c538`, we can calculate where it would end if it were allocated contiguously. `1361208/512 = 2658.609...` which means the file fits inside 2659 sectors. `2659/8 = 332.375` which means the file fits inside 333 clusters. The last cluster at sector offset `332*8 + 17552 = 20208`, should contain the footer of the jpg file. 

``` sh
$ dd if=usb.dd skip=20208 count=8 status=none | xxd
[REMOVED]
00000520: 95fa 8f24 a3f1 0d3c c4d8 e880 ad44 aad8  ...$...<.....D..
00000530: a82a ef83 12e7 ffd9 0000 0000 0000 0000  .*..............
[REMOVED]
```

Luckily for us, it does contain the jpg footer, `0xffd9`. This means the file was likely allocated contiguously and we can extract it. Let's use *dd* to see if we actually can extract it correctly.

``` sh
$ dd if=usb.dd skip=17552 count=2664 of=first_recovered_image.jpg
2664+0 records in
2664+0 records out
1363968 bytes (1.4 MB, 1.3 MiB) copied, 0.00786113 s, 174 MB/s
$ file first_recovered_image.jpg
first_recovered_image.jpg: JPEG image data, JFIF standard 1.02, resolution (DPI), density 72x72, segment length 16, progressive, precision 8, 3029x2019, components 3
```

It worked! Using this method, we can recover the rest of the files in the "pictures" directory.

File 2 starts at cluster `0x159`, 345, with size `0x3f3931`, 4143409. It should be in sector `(345-2)*8 + 17472 = 20216` with size `ceil(ceil(4143409/512)/8)*8 = 8096` sectors.

File 3 starts at cluster `0x54d`, 1357, with size `0x189965`, 1612133. It should be in sector `(1357-2)*8 + 17472 = 28312` with size `ceil(ceil(1612133/512)/8)*8 = 3152` sectors.

File 4 starts at cluster `0x6d7`, 1751, with size `0x232e47`, 2305607. It should be in sector `(1751-2)*8 + 17472 = 31464` with size `ceil(ceil(2305607/512)/8)*8 = 4504` sectors.

File 5 starts at cluster `0x90a`, 2314, with size `0x3c05e8`, 3933672. It should be in sector `(2314-2)*8 + 17472 = 35968` with size `ceil(ceil(3933672/512)/8)*8 = 7688` sectors.

``` sh
$ dd if=usb.dd skip=20216 count=8096 of=second_recovered_image.jpg
8096+0 records in
8096+0 records out
4145152 bytes (4.1 MB, 4.0 MiB) copied, 0.0246645 s, 168 MB/s
$ file second_recovered_image.jpg 
second_recovered_image.jpg: JPEG image data, JFIF standard 1.02, resolution (DPI), density 72x72, segment length 16, progressive, precision 8, 5462x3641, components 3
$ dd if=usb.dd skip=28312 count=3152 of=third_recovered_image.jpg
3152+0 records in
3152+0 records out
1613824 bytes (1.6 MB, 1.5 MiB) copied, 0.0093509 s, 173 MB/s
$ file third_recovered_image.jpg
third_recovered_image.jpg: JPEG image data, JFIF standard 1.02, resolution (DPI), density 72x72, segment length 16, progressive, precision 8, 2207x3130, components 3
$ dd if=usb.dd skip=31464 count=4504 of=fourth_recovered_image.jpg
4504+0 records in
4504+0 records out
2306048 bytes (2.3 MB, 2.2 MiB) copied, 0.013897 s, 166 MB/s
$ file fourth_recovered_image.jpg
fourth_recovered_image.jpg: JPEG image data, JFIF standard 1.02, resolution (DPI), density 72x72, segment length 16, progressive, precision 8, 3840x2160, components 3
$ dd if=usb.dd skip=35968 count=7688 of=fifth_recovered_image.jpg
7688+0 records in
7688+0 records out
3936256 bytes (3.9 MB, 3.8 MiB) copied, 0.0239841 s, 164 MB/s
$ file fifth_recovered_image.jpg 
fifth_recovered_image.jpg: JPEG image data, JFIF standard 1.02, resolution (DPI), density 72x72, segment length 16, progressive, precision 8, 6240x4160, components 3
```

## 3. Reflection
I was successfully able to recover all files without using automated tools. But it would have been much more difficult if the file allocations were not contiguous as it usually is in real life. 

Despite it being a somewhat "easy" challenge, I had to look up many details regarding the FAT File System and so it took longer than expected. However, I was able to review and reinforce my knowlege of the FAT FS as I intended. 

Perhaps I will try a more difficult FAT challenge with corrupted bytes or fragmented files. 

## references
[1] (Wikipedia) Partition type, 2026/04/09, https://en.wikipedia.org/w/index.php?title=Partition_type&oldid=1328533690#List_of_partition_IDs

## files
usb.dd file: https://drive.google.com/file/d/1IJmAcx7QKmwz1ZXuR731LjHWiPxK2b81/view?usp=drive_link
