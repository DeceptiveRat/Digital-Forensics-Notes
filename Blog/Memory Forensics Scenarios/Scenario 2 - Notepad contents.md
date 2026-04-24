In this article, I will analyze *notepad.exe*'s memory with the objective of creating a plugin that can be used to reliably find the contents of the note. 

## 1. Scenario
There are 6 *notepad.exe* windows open:
1. *test.txt* open, modified, and unsaved
2. new note, unsaved
3. new note, saved as *test2.txt* with UTF-8
4. new note, saved as *test3.txt* as UTF-16 LE
5. new note, very long, unsaved
6. new note, text written and deleted, then undone with `ctrl+z`

Screenshot of all Windows:
![[All_windows_2.png]]

Window 6 had "moretextmoretext..." after "ctrl+z", that was deleted all at once before the memory dump was acquired:
![[All_windows_1.png]]

## 2. Analysis
First let's check the *pslist* plugin to make sure all processes were captured.

``` sh
$ vol -q -f Notepad_forensics.raw windows.pslist | grep -F -e "notepad.exe"
3136    4884    notepad.exe     0xa2837ed95080  2       -       1       False   2026-04-23 01:50:59.000000 UTC  N/A     Disabled
996     3136    notepad.exe     0xa2837f506080  2       -       1       False   2026-04-23 01:51:21.000000 UTC  N/A     Disabled
2632    3136    notepad.exe     0xa2838043d300  4       -       1       False   2026-04-23 01:51:29.000000 UTC  N/A     Disabled
6424    2632    notepad.exe     0xa2837fe5e300  3       -       1       False   2026-04-23 01:51:32.000000 UTC  N/A     Disabled
7152    2632    notepad.exe     0xa2837ea2e080  3       -       1       False   2026-04-23 01:51:35.000000 UTC  N/A     Disabled
2128    2632    notepad.exe     0xa2837f4a3080  2       -       1       False   2026-04-23 01:56:26.000000 UTC  N/A     Disabled
```

I do not know which process is which window, except for window 1 which should be the first process with PID 3136. So let's start with that. 

First we need the address of heaps.

``` sh
$ volshell -w --pid 3136 -f Notepad_forensics.raw --script extract_heap.py --script-only
[REMOVED]
Number of heaps: 4
Process heaps at:
0x000001920ff80000
0x000001920fe50000
0x0000019211a50000
0x0000019211a20000
[REMOVED]
```

Now we can extract them using *vadinfo*.

``` sh
$ vol -f Notepad_forensics.raw -q windows.vadinfo --pid 3136 --dump --address 0x000001920ff80000
$ vol -f Notepad_forensics.raw -q windows.vadinfo --pid 3136 --dump --address 0x000001920fe50000
$ vol -f Notepad_forensics.raw -q windows.vadinfo --pid 3136 --dump --address 0x0000019211a50000
$ vol -f Notepad_forensics.raw -q windows.vadinfo --pid 3136 --dump --address 0x0000019211a20000
```

From the *.dmp* files we have to find the text that is inside Window 1. 

``` sh
$ strings --print-file-name -el *.dmp > all_wide_strings.txt
$ cat all_wide_strings.txt | grep -F -e "test.txt"
[REMOVED]
pid.3136.vad.0x1920ff80000-0x1921007ffff.dmp: Original "test.txt" saved to disk!
pid.3136.vad.0x1920ff80000-0x1921007ffff.dmp: modifying test.txt...is text is ap
[REMOVED]
```

Because we want to find how the text is stored, we need to view all bytes around the text, not just the strings. For this, we can use *xxd*.

``` sh
$ strings -tx -el -a pid.3136.vad.0x1920ff80000-0x1921007ffff.dmp | grep -F -e "test.txt"
[REMOVED]
  755b0 Original "test.txt" saved to disk!
  755fc modifying test.txt...is text is ap
[REMOVED]
$ xxd pid.3136.vad.0x1920ff80000-0x1921007ffff.dmp | grep -C 50 -F -e "755b0"
[REMOVED]
00075590: 0000 0000 0000 0000 0000 0000 0000 0000  ................
000755a0: f02b f80f 9201 0000 2390 95f5 46c1 002e  .+......#...F...
000755b0: 4f00 7200 6900 6700 6900 6e00 6100 6c00  O.r.i.g.i.n.a.l.
000755c0: 2000 2200 7400 6500 7300 7400 2e00 7400   .".t.e.s.t...t.
000755d0: 7800 7400 2200 2000 7300 6100 7600 6500  x.t.". .s.a.v.e.
000755e0: 6400 2000 7400 6f00 2000 6400 6900 7300  d. .t.o. .d.i.s.
000755f0: 6b00 2100 0d00 0a00 0d00 0a00 6d00 6f00  k.!.........m.o.
00075600: 6400 6900 6600 7900 6900 6e00 6700 2000  d.i.f.y.i.n.g. .
00075610: 7400 6500 7300 7400 2e00 7400 7800 7400  t.e.s.t...t.x.t.
00075620: 2e00 2e00 2e00 6900 7300 2000 7400 6500  ......i.s. .t.e.
00075630: 7800 7400 2000 6900 7300 2000 6100 7000  x.t. .i.s. .a.p.
00075640: 0000 0000 0000 0000 0800 d713 9201 0000  ................
[REMOVED]
```

There is no easy indicator showing the text content is stored there. Time to look at the remaining windows.

``` sh
# extract all heaps and find the offset of text within the heaps
volshell -w --pid [PID] -f Notepad_forensics.raw --script extract_heap.py --script-only | grep -e "0x" > heap_addresses.txt; 
	cat heap_addresses.txt | xargs -I {} vol -f Notepad_forensics.raw -q windows.vadinfo --pid [PID] --dump --address {} > /dev/null; 
	mv *.dmp PID_[PID]; cd PID_[PID];
	strings --print-file-name -tx -el *.dmp | grep -e "Window [0-9]:" | awk '{print $1 $2}' | tr : " " | xargs sh -c 'xxd $1 | grep -C 15 -e $2 > ../xxd_[PID].txt' --;
	cd ..;
```

Using this command on the remaining PIDs, I extracted their text and surrounding bytes.

``` sh
$ cat xxd_6424.txt
[REMOVED]
00040090: d007 0002 0000 0000 6185 7244 a32a 0020  ........a.rD.*.
000400a0: 5700 6900 6e00 6400 6f00 7700 2000 3400  W.i.n.d.o.w. .4.
000400b0: 3a00 2000 6e00 6500 7700 2000 6e00 6f00  :. .n.e.w. .n.o.
000400c0: 7400 6500 2e00 2000 5700 6900 6c00 6c00  t.e... .W.i.l.l.
000400d0: 2000 6800 6100 7600 6500 2000 5500 5400   .h.a.v.e. .U.T.
000400e0: 4600 2d00 3100 3600 2000 4c00 4500 2000  F.-.1.6. .L.E. .
000400f0: 6500 6e00 6300 6f00 6400 6900 6e00 6700  e.n.c.o.d.i.n.g.
00040100: 2000 7700 6800 6500 6e00 2000 7300 6100   .w.h.e.n. .s.a.
00040110: 7600 6500 6400 2000 7400 6f00 2000 6400  v.e.d. .t.o. .d.
00040120: 6900 7300 6b00 2e00 2000 0000 0000 0000  i.s.k... .......
00040130: 0000 0000 0000 0000 0000 0000 0000 0000  ................
[REMOVED]
```

Unfortunately, bytes before and after the text content are not consistent and I can't seem to find a discernible pattern. We will have to use a different approach.

Because all content are saved to a different address, there could be pointers somewhere inside the process.

The content of PID 6424 is at offset `0x400a0` within the heap located at `0x2b088890000`. This means the content is at VA `0x2b0888d00a0`.

``` sh
$ get_byte_offset -f pid.6424.dmp -l 629407744 -p a0008d88b002
offset: 998336(0xf3bc0)
offset: 4443344(0x43ccd0)
offset: 4919304(0x4b1008)
offset: 6477680(0x62d770)
```

Searching for the address in little endian, we can see there are 4 search results. Let's take a look at each.

``` sh
$ xxd -s 998336 -l 1024 pid.6424.dmp
000f3bc0: a000 8d88 b002 0000 8007 8988 b002 0000  ................
000f3bd0: 0100 0000 0000 0000 c089 9188 b002 0000  ................
000f3be0: 0100 0000 0000 0000 e087 9188 b002 0000  ................
000f3bf0: 0100 0000 0000 0000 f086 9188 b002 0000  ................
000f3c00: 0100 0000 0000 0000 4091 9188 b002 0000  ........@.......
[REMOVED]
$ xxd -s 4443280 -l 144 pid.6424.dmp
0043cc90: 0000 0000 0000 0000 0000 0000 0000 0000  ................
0043cca0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
0043ccb0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
0043ccc0: 0000 0000 0000 0000 0c00 000c 972a 0b00  .............*..
0043ccd0: a000 8d88 b002 0000 7047 c28e b002 0000  ........pG......
0043cce0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
0043ccf0: 101b 0d8b b002 0000 d01c 0d8b b002 0000  ................
0043cd00: 0000 0000 0000 0000 0000 0000 0000 0000  ................
0043cd10: 0000 0000 0000 0000 0700 0007 a82a 0b00  .............*..
$ xxd -s 4919240 -l 144 pid.6424.dmp
004b0fc8: 0100 0100 0100 0100 0100 0100 0100 0100  ................
004b0fd8: 0100 0100 0100 0100 0100 0100 0100 0100  ................
004b0fe8: 0100 0100 0100 0100 0100 0100 0100 0100  ................
004b0ff8: 0100 0100 0100 0100 0300 0000 0000 0000  ................
004b1008: a000 8d88 b002 0000 0300 0000 0000 0000  ................
004b1018: 8048 8f88 b002 0000 0300 0000 0000 0000  .H..............
004b1028: 60cd 8c88 b002 0000 0300 0000 0000 0000  `...............
004b1038: 0078 8e88 b002 0000 0300 0000 0000 0000  .x..............
004b1048: f0de 8988 b002 0000 0300 0000 0000 0000  ................
$ xxd -s 6477616 -l 144 pid.6424.dmp
0062d730: 0000 0000 0101 0000 0100 0000 c809 0000  ................
0062d740: 0200 0000 0000 0000 0000 0000 0000 0000  ................
0062d750: 0000 0000 0000 0000 0000 0000 0000 0000  ................
0062d760: 0000 0000 0000 0000 0c00 000c b92a 1600  .............*..
0062d770: a000 8d88 b002 0000 d0fc 8b88 b002 0000  ................
0062d780: 0000 0000 0000 0000 0000 0000 0000 0000  ................
0062d790: 0000 0000 0000 0000 0000 0000 0000 0000  ................
0062d7a0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
0062d7b0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
```

Doing the same for PIDs 7152 and 2128, we can start to see a pattern.

``` sh
# find content pointer for PID 7152
$ get_byte_offset -f pid.7152.dmp -l 629895168 -p 90c026a02501
offset: 6905864(0x696008)
$ xxd -s 6905800 -l 144 pid.7152.dmp
00695fc8: 0100 0100 0100 0100 0100 0100 0100 0100  ................
00695fd8: 0100 0100 0100 0100 0100 0100 0100 0100  ................
00695fe8: 0100 0100 0100 0100 0100 0100 0100 0100  ................
00695ff8: 0100 0100 0100 0100 0300 0000 0000 0000  ................
00696008: 90c0 26a0 2501 0000 0300 0000 0000 0000  ..&.%...........
00696018: 90bd 27a0 2501 0000 0300 0000 0000 0000  ..'.%...........
00696028: 008f 24a0 2501 0000 0300 0000 0000 0000  ..$.%...........
00696038: 807f 26a0 2501 0000 0300 0000 0000 0000  ..&.%...........
00696048: 40f2 24a0 2501 0000 0300 0000 0000 0000  @.$.%...........
# find content pointer for PID 2128
$ get_byte_offset -f pid.2128.dmp -l 617717760 -p f0630b2aab02
offset: 444448(0x6c820)
offset: 14741512(0xe0f008)
$ xxd -s 14741448 -l 144 pid.2128.dmp
00e0efc8: 0100 0100 0100 0100 0100 0100 0100 0100  ................
00e0efd8: 0100 0100 0100 0100 0100 0100 0100 0100  ................
00e0efe8: 0100 0100 0100 0100 0100 0100 0100 0100  ................
00e0eff8: 0100 0100 0100 0100 0300 0000 0000 0000  ................
00e0f008: f063 0b2a ab02 0000 0300 0000 0000 0000  .c.*............
00e0f018: d09f 0c2a ab02 0000 0300 0000 0000 0000  ...*............
00e0f028: b0fc 0a2a ab02 0000 0300 0000 0000 0000  ...*............
00e0f038: a09b 092a ab02 0000 0300 0000 0000 0000  ...*............
00e0f048: a0ff 092a ab02 0000 0300 0000 0000 0000  ...*............
```

The first 64 bytes of the outputs match one of the outputs for PID 6424. 

Let's see what the pointers after the content pointer point to. 

``` python
(layer_name_Process2128_1) >>> db(0x02ab2a0c9fd0)
0x2ab2a0c9fd0    68 03 03 00 00 00 00 00 00 00 00 00 00 00 00 00    h...............
0x2ab2a0c9fe0    00 00 00 00 00 00 00 00 00 00 00 00 01 00 00 00    ................
0x2ab2a0c9ff0    08 00 00 00 e9 ff ff ff 00 00 00 00 00 00 00 00    ................
0x2ab2a0ca000    00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0x2ab2a0ca010    43 00 6f 00 6e 00 73 00 6f 00 6c 00 61 00 73 00    C.o.n.s.o.l.a.s.
0x2ab2a0ca020    00 00 6e 00 73 00 6f 00 6c 00 65 00 00 00 00 00    ..n.s.o.l.e.....
0x2ab2a0ca030    00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0x2ab2a0ca040    00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
(layer_name_Process2128_1) >>> db(0x02ab2a0afcb0)
0x2ab2a0afcb0    64 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    d...............
0x2ab2a0afcc0    00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0x2ab2a0afcd0    00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0x2ab2a0afce0    00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0x2ab2a0afcf0    00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0x2ab2a0afd00    00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0x2ab2a0afd10    00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0x2ab2a0afd20    00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
(layer_name_Process2128_1) >>> db(0x02ab2a099ba0)
0x2ab2a099ba0    90 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0x2ab2a099bb0    00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0x2ab2a099bc0    00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0x2ab2a099bd0    00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0x2ab2a099be0    00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0x2ab2a099bf0    00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0x2ab2a099c00    00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0x2ab2a099c10    00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
(layer_name_Process2128_1) >>> db(0x02ab2a09ffa0)
0x2ab2a09ffa0    1c 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0x2ab2a09ffb0    00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0x2ab2a09ffc0    00 00 00 00 00 00 00 00 48 00 f3 2d ab 02 00 00    ........H..-....
0x2ab2a09ffd0    00 00 00 00 00 00 00 00 49 6a fc 46 c2 9b 00 10    ........Ij.F....
0x2ab2a09ffe0    00 2e 06 00 9c 1d 65 00 84 00 d3 04 03 00 52 13    ......e.......R.
0x2ab2a09fff0    04 00 56 13 05 00 5a 13 06 00 5e 13 07 00 64 13    ..V...Z...^...d.
0x2ab2a0a0000    08 00 64 13 09 00 68 13 0a 00 72 13 0d 00 72 13    ..d...h...r...r.
0x2ab2a0a0010    0f 00 76 13 10 00 7c 13 11 00 7c 13 12 00 80 13    ..v...|...|.....
```

The first pointer after the content pointer points to a data block that contains the string "Consolas". The next pointer points to a block starting with `0x64`. The next one `0x90`, and so on. These 4 pointers seem to point to blocks with certain patterns, and we can use this to our advantage so keep it in mind. 

``` sh
# get the sha-256 hashes of 1088 bytes before content pointer
$ xxd -s 6904776 -l 1088 pid.7152.dmp | cut -d":" -f 2- | sha256sum
5c8c8c70c718d84a57f578308abdbfefdf7e590917d2cc33349f94de5c0bbd9b  -
$ xxd -s 4918216 -l 1088 pid.6424.dmp | cut -d":" -f 2- | sha256sum
5c8c8c70c718d84a57f578308abdbfefdf7e590917d2cc33349f94de5c0bbd9b  -
$ xxd -s 14740424 -l 1088 pid.2128.dmp | cut -d":" -f 2- | sha256sum
5c8c8c70c718d84a57f578308abdbfefdf7e590917d2cc33349f94de5c0bbd9b  -
```

Upon further examination, it seems at least 1088 byte that exist before the address of the content are identical. This may be the signature we were looking for. 

Let's extract these 1088 bytes and test them using the remaining *notepad.exe* processes.

``` sh
# getting signature
$ xxd -s 4918216 -l 1088 -p pid.6424.dmp | tr -d "\n"
# find signature in dump file
$ get_byte_offset -f pid.3136.dmp -l 629407744 -p [signature]
offset: 3038152(0x2e5bc8)
```

The signature offset is 3038152, meaning the content address should be at offset 3038152 + 1088, which is 3039240. 

``` sh
$ xxd -s 3039240 -l 8 -p pid.3136.dmp
b055ff0f92010000
```

This means the content is at VA `0x01920fff55b0`. We can verify this using *volshell*.

``` python
(layer_name_Process3136_1) >>> db(0x01920fff55b0)
0x1920fff55b0    4f 00 72 00 69 00 67 00 69 00 6e 00 61 00 6c 00    O.r.i.g.i.n.a.l.
0x1920fff55c0    20 00 22 00 74 00 65 00 73 00 74 00 2e 00 74 00    ..".t.e.s.t...t.
0x1920fff55d0    78 00 74 00 22 00 20 00 73 00 61 00 76 00 65 00    x.t."...s.a.v.e.
0x1920fff55e0    64 00 20 00 74 00 6f 00 20 00 64 00 69 00 73 00    d...t.o...d.i.s.
0x1920fff55f0    6b 00 21 00 0d 00 0a 00 0d 00 0a 00 6d 00 6f 00    k.!.........m.o.
0x1920fff5600    64 00 69 00 66 00 79 00 69 00 6e 00 67 00 20 00    d.i.f.y.i.n.g...
0x1920fff5610    74 00 65 00 73 00 74 00 2e 00 74 00 78 00 74 00    t.e.s.t...t.x.t.
0x1920fff5620    2e 00 2e 00 2e 00 69 00 73 00 20 00 74 00 65 00    ......i.s...t.e.
```

Repeating this procedure on PID 996 works, but for PID 2632, which contains the very long text, the signature is not found at all. 

Let's manually examine PID 2632 to see if we can locate the content. 

Assuming long content pointers also point to the "Consolas block" as the other content pointers do, if we can find the string "Consolas", we can locate the pointer to the block of data which should be 16 bytes after the pointer to the start of content. 

Let's check the heap for the string. 

``` sh
$ cat xxd_2632_full.txt | grep -F -B 4 -e "C.o.n.s.o.l.a.s."
00063740: a402 0500 0000 0000 0000 0000 0000 0000  ................
00063750: 0000 0000 0000 0000 0000 0000 0100 0000  ................
00063760: 0800 0000 e3ff ffff 0000 0000 0000 0000  ................
00063770: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00063780: 4300 6f00 6e00 7300 6f00 6c00 6100 7300  C.o.n.s.o.l.a.s.
```

The block is found, meaning the pointer to the data block is `0x1a47c783740`(`0x63740 + 0x1a47c720000`).

``` sh
# find pointer to "Consolas block"
$ get_byte_offset -f pid.2632.dmp -l 615596032 -p 4037787ca401
offset: 4112408(0x3ec018)
$ xxd -s 4112392 -l 24 pid.2632.dmp
003ec008: a000 657e a401 0000 0300 0000 0000 0000  ..e~............
003ec018: 4037 787c a401 0000                      @7x|....
# dump VAD pointed to by content pointer
$ vol -f Notepad_forensics.raw windows.vadinfo --pid 2632 --address 0x01a47e6500a0 --dump
$ xxd pid.2632.vad.0x1a47e610000-0x1a47e70ffff.dmp > xxd_2632_other.txt
$ cat xxd_2632_other.txt | grep -C 2 400a0
00040080: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00040090: 0000 0000 0000 0000 794c 1a6e cfc1 0526  ........yL.n...&
000400a0: 5700 6900 6e00 6400 6f00 7700 2000 3500  W.i.n.d.o.w. .5.
000400b0: 3a00 2000 5600 6500 7200 7900 2000 6c00  :. .V.e.r.y. .l.
000400c0: 6f00 6e00 6700 2000 7400 6500 7800 7400  o.n.g. .t.e.x.t.
```

One thing to note is that the pointer to the start of content is `0x1a47e6500a0`, which is not in any of the heaps. 

Using everything we've learned so far, let's see if we can create a plugin to extract notepad contents. 

First, let's verify we can use *vadyarascan* with the signature we found to extract the content pointer for short notes. 

``` sh
$ vol -f Notepad_forensics.raw vadyarascan --pid 7152 --yara-string "{ [signature] }"
```

This doesn't output anything, meaning there was no match anywhere. This is not that suprising if you think about it, because if the signature crosses VAD boundaries, it would fail to match. 

To deal with this problem, we can find the VAD containing the content pointer, find the offset of the pointer, and use a signature of the same length so it fits inside the same VAD.

``` sh
# search for VA of content
$ vol -f Notepad_forensics.raw vadyarascan --pid 7152 --yara-string "{ 90c026a02501 }"
Volatility 3 Framework 2.27.0
Progress:  100.00               PDB scanning finished
Offset  PID     CreateTime      PPID    ImageFileName   SessionId       Threads Rule    Component       Value

0x125a40e0008   7152    2026-04-23 01:51:35.000000 UTC  2632    notepad.exe     1       3       r1      $a
90 c0 26 a0 25 01                               ..&.%.
# find VAD containing content pointer
$ vol -f Notepad_forensics.raw vadinfo --pid 7152 --address 0x125a40e0008
Volatility 3 Framework 2.27.0
Progress:  100.00               PDB scanning finished
PID     Process Offset  Start VPN       End VPN Tag     Protection      CommitCharge    PrivateMemory   Parent  File    File output

7152    notepad.exe     0xffffa2837f8cdfa0      0x125a40e0000   0x125a41dffff   [REMOVED]
```

The content pointer is located at VA `0x125a40e0008`, which is in the VAD at `0x125a40e0000`. This means the pointer is located at offset 8 inside the VAD. 

It might not seem that important at first, but it can help us significantly if you think about it.

Because only the last 8 bytes of the signature are in the same VAD, the rest are in the VAD directly before it. This means we just have to find that VAD, and the VAD after that will always (for small enough notes) contain the pointer to the content at offset 8. 

Let's see if we can identify that VAD.

``` sh
$ vol -q -f Notepad_forensics.raw vadinfo --pid 7152 | grep -e 0x125a40e0000 -e 0x125a40dffff
7152    notepad.exe     0xffffa28380572600      0x125a2e80000   0x125a40dffff   Vad     PAGE_READONLY   0       0       0x0     \Windows\Fonts\StaticCache.dat  Disabled
7152    notepad.exe     0xffffa2837f8cdfa0      0x125a40e0000   0x125a41dffff   VadS    PAGE_READWRITE  1       1       0xffffa2838057ddc0      N/A     Disabled
```

Subtracting 1 from the starting address of our content pointer VAD, we can find the VAD directly before it. The VAD contains a file, `\Windows\Fonts\StaticCache.dat`, which should make identifying the VAD very easy. 

We now have a way to find the contents of short notes, but what about long notes? 

``` sh
$ vol -q -f Notepad_forensics.raw vadyarascan --yara-string "{ a000657ea401000003 }" --pid 2632
Volatility 3 Framework 2.27.0

Offset  PID     CreateTime      PPID    ImageFileName   SessionId       Threads Rule    Component       Value

0x1a47f160008   2632    2026-04-23 01:51:29.000000 UTC  3136    notepad.exe     1       4       r1      $a
a0 00 65 7e a4 01 00 00 03                      ..e~.....
# get VAD containing StaticCache.dat
$ vol -q -f Notepad_forensics.raw vadinfo --pid 2632 | grep "Fonts"
2632    notepad.exe     0xffffa2838056e820      0x1a400000000   0x1a40125ffff   Vad     PAGE_READONLY   0       0       0xffffa2837fc52400      \Windows\Fonts\StaticCache.dat  Disabled
```

The content pointer is inside the VAD starting at `0x1a47f160008`. Because the VAD containing *StaticCache.dat* ends at `0x1a40125ffff`, we can see the VADs are not next to each other. 

However, note the offset of the content pointer. It seems to be at offset 8 of the VAD again. We can verify this with *vadinfo*.

``` sh
$ vol -q -f Notepad_forensics.raw vadinfo --pid 2632 --address 0x1a47f160008
Volatility 3 Framework 2.27.0

PID     Process Offset  Start VPN       End VPN Tag     Protection      CommitCharge    PrivateMemory   Parent  File    File output

2632    notepad.exe     0xffffa2837f0f9e00      0x1a47f160000   0x1a47f25ffff   VadS    PAGE_READWRITE  1       1       0xffffa2837ecff820      N/A     Disabled
```

Also note the VAD attributes. "VadS" tag, read/write permissions, and private memory. This is the same for all *notepad.exe* processes in this memory dump. 

This should be enough to build our plugin for extracting the note content. 

## 3. plugin creation
``` python
def verify_address(content_pointer_VAD, layer):
    try:
        # offset 8 contains valid pointer
        content_pointer_bytes = layer.read(content_pointer_VAD+8, 8, pad=True)

        # offset 24 contains valid pointer and points to "Consolas block"
        consolas_pointer_bytes = layer.read(content_pointer_VAD+24, 8, pad=True)
        consolas_pointer = int.from_bytes(consolas_pointer_bytes, byteorder="little")
        consolas_bytes = layer.read(consolas_pointer, 96, pad=True)
        if consolas_bytes[64:96] != b"\x43\x00\x6f\x00\x6e\x00\x73\x00\x6f\x00\x6c\x00\x61\x00\x73\x00\x00\x00\x6e\x00\x73\x00\x6f\x00\x6c\x00\x65\x00\x00\x00\x00\x00":
            return False

        # offset 40 contains valid pointer and points to block starting with 0x64
        temp_pointer_bytes = layer.read(content_pointer_VAD+40, 8, pad=True)
        temp_pointer = int.from_bytes(temp_pointer_bytes, byteorder="little")
        temp_byte = layer.read(temp_pointer, 1, pad=True)
        if temp_byte != b'\x64':
            return False

        # offset 56 contains valid pointer and points to block starting with 0x90
        temp_pointer_bytes = layer.read(content_pointer_VAD+56, 8, pad=True)
        temp_pointer = int.from_bytes(temp_pointer_bytes, byteorder="little")
        temp_byte = layer.read(temp_pointer, 1, pad=True)
        if temp_byte != b'\x90':
            return False

        # offset 72 contains valid pointer and points to block starting with 0x1c
        temp_pointer_bytes = layer.read(content_pointer_VAD+72, 8, pad=True)
        temp_pointer = int.from_bytes(temp_pointer_bytes, byteorder="little")
        temp_byte = layer.read(temp_pointer, 1, pad=True)
        if temp_byte != b'\x1c':
            return False
    except:
        print(f"Address unavailable: {hex(content_pointer_VAD)}")
        return False

    content_virtual_address = int.from_bytes(content_pointer_bytes, byteorder="little")
    return content_virtual_address
```

Using our knowledge of where the pointers are within the VAD and what those pointers point to, we can create a function to perform a sanity check on the VAD. 

``` python
    def find_content_pointer_vad(self, vad_node):
        kernel = self.context.modules[self.config['kernel']]
        protect_vals = vadinfo.VadInfo.protect_values(
            self.context,
            kernel.layer_name,
            kernel.symbol_table_name
        )
        winnt_vals = vadinfo.winnt_protections

        if vad_node.get_tag() != "VadS":
            #print(f"{hex(vad_node.get_start())}: wrong tag!")
            return True

        if vad_node.get_protection(protect_vals, winnt_vals) != "PAGE_READWRITE":
            #print(f"{hex(vad_node.get_start())}: wrong protection!")
            return True

        if vad_node.get_private_memory() != 1:
            #print(f"{hex(vad_node.get_start())}: wrong private memory config!")
            return True

        # Keep everything else
        return False
```

Also, we can leverage our knowledge of content pointer VAD attributes. 

Using the plugin, we can extract the notes from all *notepad.exe* processes with just their PIDs.
``` sh
$ vol -f Notepad_forensics.raw windows.note_extractor --pid 3136
[REMOVED]
0x1920fff55b0   Original "test.txt" saved to disk!

modifying test.txt...is text is ap      4f 00 72 00 69 00 67 00 69 00 6e 00 61 00 6c 00 20 00 22 00 74 00 65 00 73 00 74 00 2e 00 74 00 78 00 74 00 22 00 20 00 73 00 61 00 76 00 65 00 64 00 20 00 74 00 6f 00 20 00 64 00 69 00 73 00 6b 00 21 00 0d 00 0a 00 0d 00 0a 00 6d 00 6f 00 64 00 69 00 66 00 79 00 69 00 6e 00 67 00 20 00 74 00 65 00 73 00 74 00 2e 00 74 00 78 00 74 00 2e 00 2e 00 2e 00 69 00 73 00 20 00 74 00 65 00 78 00 74 00 20 00 69 00 73 00 20 00 61 00 70 00
$ vol -f Notepad_forensics.raw windows.note_extractor --pid 996
[REMOVED]
$ vol -f Notepad_forensics.raw windows.note_extractor --pid 2632
0x1f85fc0e7d0   Window 2: new note, but not saved to disk...    57 00 69 00 6e 00 64 00 6f 00 77 00 20 00 32 00 3a 00 20 00 6e 00 65 00 77 00 20 00 6e 00 6f 00 74 00 65 00 2c 00 20 00 62 00 75 00 74 00 20 00 6e 00 6f 00 74 00 20 00 73 00 61 00 76 00 65 00 64 00 20 00 74 00 6f 00 20 00 64 00 69 00 73 00 6b 00 2e 00 2e 00 2e 00
0x1a47e6500a0   Window 5: Very long text being repeated over and over. Very long text being repeated over and over. Very long text being repeated over and over. Very long text being repeated over and over. Very long text being repeated over and over. Very long text being repeated over and over. Very long text being repeated over and over. Very long text being repeated over and over. [REMOVED]
```

#### *Note: This was performed on Windows 10 and Notepad version 10.0.19041.1865. Results may differ for other versions of Windows or Notepad.*
``` sh
$ vol -f Notepad_forensics.raw windows.info
Volatility 3 Framework 2.27.0
Progress:  100.00               PDB scanning finished
Variable        Value

Kernel Base     0xf80218600000
DTB     0x1aa000
Symbols file:[REDACTED]/volatility3/symbols/windows/ntkrnlmp.pdb/89284D0CA6ACC8274B9A44BD5AF9290B-1.json.xz
Is64Bit True
IsPAE   False
layer_name      0 WindowsIntel32e
memory_layer    1 Elf64Layer
base_layer      2 FileLayer
KdVersionBlock  0xf8021920f3a0
Major/Minor     15.19041
MachineType     34404
KeNumberProcessors      4
SystemTime      2026-04-23 01:59:26+00:00
NtSystemRoot    C:\Windows
NtProductType   NtProductWinNt
NtMajorVersion  10
NtMinorVersion  0
PE MajorOperatingSystemVersion  10
PE MinorOperatingSystemVersion  0
PE Machine      34404
PE TimeDateStamp        Fri May 20 08:24:42 2101
$ exiftool PE.0xa2837ed95080.3136.0x7ff601cc0000.dmp 
ExifTool Version Number         : 12.76
File Name                       : PE.0xa2837ed95080.3136.0x7ff601cc0000.dmp
Directory                       : .
File Size                       : 229 kB
File Modification Date/Time     : 2026:04:24 20:35:48+09:00
File Access Date/Time           : 2026:04:24 20:36:04+09:00
File Inode Change Date/Time     : 2026:04:24 20:35:48+09:00
File Permissions                : -rw-------
File Type                       : Win64 EXE
File Type Extension             : exe
MIME Type                       : application/octet-stream
Machine Type                    : AMD AMD64
Time Stamp                      : 2070:12:03 20:32:29+09:00
Image File Characteristics      : Executable, Large address aware
PE Type                         : PE32+
Linker Version                  : 14.20
Code Size                       : 149504
Initialized Data Size           : 57344
Uninitialized Data Size         : 0
Entry Point                     : 0x23f40
OS Version                      : 10.0
Image Version                   : 10.0
Subsystem Version               : 10.0
Subsystem                       : Windows GUI
File Version Number             : 10.0.19041.1865
Product Version Number          : 10.0.19041.1865
File Flags Mask                 : 0x003f
File Flags                      : (none)
File OS                         : Windows NT 32-bit
Object File Type                : Executable application
File Subtype                    : 0
Language Code                   : English (U.S.)
Character Set                   : Unicode
Company Name                    : Microsoft Corporation
File Description                : Notepad
File Version                    : 10.0.19041.1865 (WinBuild.160101.0800)
Internal Name                   : Notepad
Legal Copyright                 : © Microsoft Corporation. All rights reserved.
Original File Name              : NOTEPAD.EXE
Product Name                    : Microsoft® Windows® Operating System
Product Version                 : 10.0.19041.1865
```

## 4. Reflection
I learned how to make *Volatility* plugins and *Volshell* scripts through this scenario. Also, analyzing process dumps in this way was very interesting and I am looking forward to doing similar activities in the future. 

## 5. Tools used
Custom tooms I used can be found on my Github. 
- *get_byte_offset*:
https://github.com/DeceptiveRat/DF_tools

- *note_extractor* and *extract_heap*:
https://github.com/DeceptiveRat/Custom_Volatility_Utilities

- memory dump used:
https://drive.google.com/file/d/16L363gn32zn86pe4SRvcDqrHWcB4-jje/view?usp=sharing
