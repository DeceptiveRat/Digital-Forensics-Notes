In my last post, I analyzed *notepad.exe* memory on Windows 10 to create a plugin to extract notepad contents. I was initially planning to analyze deleted text that can be recovered with `ctrl z` as well, but cut it short because I didn't want the post to get too long. However, I noticed the text that was deleted (window 6, moretextmoretext...) wasn't actually deleted or moved in memory.

This made me think there must be the length of the content stored somewhere in memory to prevent deleted text from being displayed as well.

I will continue to analyze *notepad.exe* for deleted text in this post. The focus is on how `ctrl z` works on *notepad.exe*, to restore deleted text or delete newly added text. 

## 1. Scenario
#### Window 1 creation steps 

![[window1_state1.png]]
It starts out with 2 text.

![[window1_state2.png]]
Both texts are deleted. 

![[window1_state2_ctrlz.png]]
Using `ctrl z` we can see `state1` is restored.

#### Window 2 creation steps

![[window2_state1.png]]
Starts out same as the first window.

![[window2_state2.png]]
One word is deleted.

![[window2_state3.png]]
A shorter word is added.

![[window2_state3_ctrlz.png]]
`ctrl z` restores the deleted word, replacing the new word.

#### Memory capture state:
![[capture_state.png]]

Window 3 is created in the same way as window 1, except the capture happened with the deleted text restored.

## 2. Analysis
First, we need the PIDs of our *notepad.exe* processes.

``` sh
$ vol --save-config config.json -f notepad_deleted.raw windows.pslist | grep notepad
4040    3532    notepad.exe     0xa0843a3e2080  3       -       1       False   2026-05-03 02:20:12.000000 UTC  N/A     Disabled
4916    4040    notepad.exe     0xa0843af760c0  4       -       1       False   2026-05-03 02:20:38.000000 UTC  N/A     Disabled
3672    4916    notepad.exe     0xa0843aa15080  1       -       1       False   2026-05-03 02:23:42.000000 UTC  N/A     Disabled
```

With the PIDs, let's see if we can extract their contents with the plugin from the last post.

``` sh
$ vol -q -c config.json -r pretty -f notepad_deleted.raw windows.note_extractor --pid 4040
Volatility 3 Framework 2.28.0
Formatting...
  | Virtual Address |             Method |              Content |                                                                                                             Raw Content
* |   0x227372bfbe0 | 1 (StaticCacheVad) | window1. text1 text2 | 77 00 69 00 6e 00 64 00 6f 00 77 00 31 00 2e 00 20 00 74 00 65 00 78 00 74 00 31 00 20 00 74 00 65 00 78 00 74 00 32 00
$ vol -q -c config.json -r pretty -f notepad_deleted.raw windows.note_extractor --pid 4916
Volatility 3 Framework 2.28.0
Formatting...
  | Virtual Address |         Method |               Content |                                                                                                                   Raw Content
* |   0x1b6de5dbaa0 | 2 (BruteForce) | window2. text1 new12  | 77 00 69 00 6e 00 64 00 6f 00 77 00 32 00 2e 00 20 00 74 00 65 00 78 00 74 00 31 00 20 00 6e 00 65 00 77 00 31 00 32 00 20 00
$ vol -q -c config.json -r pretty -f notepad_deleted.raw windows.note_extractor --pid 3672
Volatility 3 Framework 2.28.0
Formatting...
  | Virtual Address |             Method |              Content |                                                                                                             Raw Content
* |   0x21a2277baa0 | 1 (StaticCacheVad) | window3. text1 text2 | 77 00 69 00 6e 00 64 00 6f 00 77 00 33 00 2e 00 20 00 74 00 65 00 78 00 74 00 31 00 20 00 74 00 65 00 78 00 74 00 32 00
```

The text is extracted correctly for all windows. We can also see the text within window 1 and window 3 is the same, even though the displayed text was different when the memory was captured. This is consistent with the behavior we saw in the previous post with window 6.

Also note the output for PID 4916. The method is bruteforce, not StaticCacheVad, which is interesting because short text was always found via StaticCacheVad until now. The content is also interesting. The text is neither "new1" nor "text2". It seems as if "new1" overwrote "text2". "text2" must be saved somewhere for PID 4916, since we are able to recover it via `ctrl z`.

Let's search the address space of PID 4916 to see if we can find "text2".

``` sh
$ vol -c config.json -f notepad_deleted.raw windows.vadyarascan --pid 4916 --yara-string "text2" --wide
Volatility 3 Framework 2.27.0
Progress:  100.00               PDB scanning finished
Offset  PID     CreateTime      PPID    ImageFileName   SessionId       Threads Rule    Component       Value

0x1b6de600680   4916    2026-05-03 02:20:38.000000 UTC  4040    notepad.exe     1       4       r1      $a
74 00 65 00 78 00 74 00 32 00                   t.e.x.t.2.
```

We can see that it is at VA `0x1b6de600680`. We can use *vadinfo* to see which VAD it is inside. 

``` sh
$ vol -q -c config.json -f notepad_deleted.raw windows.vadinfo --pid 4916 --address 0x1b6de600680 | grep notepad
4916    notepad.exe     0xffffa0843b1511f0      0x1b6de5a0000   0x1b6de69ffff   VadS    PAGE_READWRITE  172     1       0xffffa0843b023140      N/A     Disabled
```

It is worth noting that this is the same VAD as the one containing the text content, as we can see from the *note_extractor* output. 

Let's see the surrounding data to see if we can find anything interesting.

``` sh
$ xxd pid.4916.vad.0x1b6de5a0000-0x1b6de69ffff.dmp > xxd_4916_0x1b6de5a0000.txt
$ cat xxd_4916_0x1b6de5a0000.txt | grep -C 20 60680
[REMOVED]
00060650: 0000 0000 0000 0000 14ba f5fb 0014 0088  ................
00060660: d0f0 5fde b601 0000 0100 0000 0000 0000  .._.............
00060670: 0000 0000 0000 0000 16ba f7fb 0015 0092  ................
00060680: 7400 6500 7800 7400 3200 2000 0000 0000  t.e.x.t.2. .....
00060690: 0000 0000 0000 0000 18ba e9fb 0016 0088  ................
000606a0: 804b c06c ff7f 0000 7005 5bde b601 0000  .K.l....p.[.....
000606b0: 0100 0000 0100 0000 1aba ebfb 0017 0080  ................
[REMOVED]
```

This doesn't seem to be very useful, but upon closer examination, we can kind of start to see a pattern emerge. 

![[undo_text_surrounding_pattern.png]]

At first, I noticed numbers being incremented. Then looking closer, we can see a few more patterns.

Bytes 9~16 seem to have the following pattern:
- byte 10 is consistent among entries; `0xba`
- byte 12 is consistent among entries; `0xfb`
- byte 13 is `0x00`
- byte 14 is incremented by 1 for each entry
- byte 15 is `0x00`

It could also be that byte 13 and 14 are incremented together, but then it would have to be in big endian, which is not typical. 

We now have an idea of how "undo text" is stored, but we are far from being able to get it. The index of 15 is probably arbitrary, meaning we need a way to find exactly which index contains our "undo text".

There must be a pointer to the "undo text" so it can be restored with `ctrl z`, so let's look for that next. 

``` sh
$ vol -q -c config.json -f notepad_deleted.raw windows.vadyarascan --pid 4916 --yara-string "{ 80 06 60 de b6 01 }" | grep notepad
0x1b6de5fd9b8   4916    2026-05-03 02:20:38.000000 UTC  4040    notepad.exe     1       4       r1      $a
```

The pointer to the undo text is also in the same VAD as the content and undo text. Let's check the surrounding data.

``` sh
$ cat xxd_4916_0x1b6de5a0000.txt | grep -C 5 5d9b0
0005d960: 0b00 0000 0000 0000 f003 0000 af00 0000  ................
0005d970: f802 0600 0000 0000 ff01 0000 2a00 0000  ............*...
0005d980: 0000 0000 500e a001 0000 0000 0000 0200  ....P...........
0005d990: d03c 5ede b601 0000 0000 0000 0000 0000  .<^.............
0005d9a0: 0000 0000 0000 0000 e301 0000 0000 0000  ................
0005d9b0: 0300 0000 0000 0000 8006 60de b601 0000  ..........`.....
0005d9c0: 0f00 0000 0600 0000 0f00 0000 1300 0000  ................
0005d9d0: 2409 0a38 0000 0000 1700 0000 3000 0000  $..8........0...
0005d9e0: 0000 0000 0800 0000 1000 0000 0000 0000  ................
0005d9f0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
0005da00: 0000 0000 0000 0000 0000 0000 1200 0000  ................
```

There doesn't seem to be anything of note so far. 

Because it is difficult to analyze undo text using just 1 process, I created another *.raw* file. The following is a simple diagram showing how each file was created.

![[switch_state_images_creation.png]]

First, let's get the PID of the *notepad.exe* process and VAD used. 

``` sh
$ vol -q -f switch_state1.raw windows.pslist | grep notepad
3908    3404    notepad.exe     0x988fab911080  6       -       1       False   2026-05-04 00:31:13.000000 UTC  N/A     Disabled
$ vol -q -f switch_state1.raw windows.note_extractor --pid 3908
[REMOVED]
0x232b06b4cb0   1 (StaticCacheVad)      aaaaaaaaaaaaaaaa        61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00
```

PID is 3908 and address of content in VAD is `0x232b06b4cb0`. 

When `ctrl z` is pressed, 1 of 2 things happens. Either (1)the displayed text is overwritten with the undo text, or (2)the content pointer is altered to point to undo text, which becomes the new displayed text. 

If we compare that VAD in state 2 with state 2 after `ctrl z`, we will be able to find out which is the case. 

``` sh
# dump VAD containing content
$ vol -f switch_state2.raw windows.vadinfo --dump --pid 3908 --address 0x232b06b4cb0
$ vol -f switch_state2_ctrlz.raw windows.vadinfo --dump --pid 3908 --address 0x232b06b4cb0
$ xxd pid.3908.vad.0x232b06b0000-0x232b07affff.dmp xxd_3908_state2.txt
$ xxd pid.3908.vad.0x232b06b0000-0x232b07affff-1.dmp xxd_3908_state2_ctrlz.txt
# use diff to compare VADs
$ diff xxd_3908_state2.txt xxd_3908_state2_ctrlz.txt > diff_switch_state2_ctrlz.txt
$ cat diff_switch_state2_ctrlz.txt
[REMOVED]
1228,1229c1228,1229
< 00004cb0: 6200 6200 6200 6200 6200 6200 6200 6200  b.b.b.b.b.b.b.b.
< 00004cc0: 6200 6200 6200 6200 6100 6100 6100 6100  b.b.b.b.a.a.a.a.
---
> 00004cb0: 6100 6100 6100 6100 6100 6100 6100 6100  a.a.a.a.a.a.a.a.
> 00004cc0: 6100 6100 6100 6100 6100 6100 6100 6100  a.a.a.a.a.a.a.a.
[REMOVED]
```

When `ctrl z` is pressed, the displayed text is just replaced with the string of "a"s. We can confirm this is the displayed text with some simple math.

``` data
(gdb) p/x 0x232b06b0000 + 0x4cb0
$1 = 0x232b06b4cb0
```

We can see this is indeed the same VA that contained the displayed text in the *text_extractor* output.

This means there was a pointer to the string of "a"s before `ctrl z` was pressed, and it was accessed to get the string to overwrite the displayed text. Let's find the pointer. 

``` sh
$ grep -e "a.a.a.a.a.a.a.a." xxd_3908_state2.txt
0006b790: 6100 6100 6100 6100 6100 6100 6100 6100  a.a.a.a.a.a.a.a.
0006b7a0: 6100 6100 6100 6100 6100 6100 6100 6100  a.a.a.a.a.a.a.a.
[REMOVED]
(gdb) p/x 0x6b790 + 0x232b06b0000
$2 = 0x232b071b790
[REMOVED]
$ cat diff_switch_state2_ctrlz.txt | grep -e "90b7 71b0 3202"
< 0005bb00: 0300 0000 0000 0000 90b7 71b0 3202 0000  ..........q.2...
> 0005c8c0: 90b7 71b0 3202 0000 00f0 0000 e802 0000  ..q.2...........
```

The pointer to string of "a"s at `0x5bb00` is overwritten with the `ctrl z` operation.

Let's take a closer look at the addresses this change occurs. 

``` sh
$ cat diff_switch_state2_ctrlz.txt | grep -C 10 -e "0005bb00" -e "0005c8c0"
[REMOVED]
23472,23474c23472,23474
< 0005baf0: 0000 0000 0000 0000 7401 0000 0000 0000  ........t.......
< 0005bb00: 0300 0000 0000 0000 90b7 71b0 3202 0000  ..........q.2...
< 0005bb10: 0000 0000 1000 0000 0000 0000 0c00 0000  ................
---
> 0005baf0: 0000 0000 0000 0000 f001 0000 0000 0000  ................
> 0005bb00: 0300 0000 0000 0000 60ba 71b0 3202 0000  ........`.q.2...
> 0005bb10: 0000 0000 0c00 0000 0000 0000 1000 0000  ................
[REMOVED]
23693c23693
< 0005c8c0: c0ba 71b0 3202 0000 00f0 0000 e802 0000  ..q.2...........
---
> 0005c8c0: 90b7 71b0 3202 0000 00f0 0000 e802 0000  ..q.2...........
[REMOVED]
```

The address at `0x5c8c0` doesn't make a lot of sense, but the address at `0x5bb00` is interesting. It used to point to a string of "a"s, but it is overwritten with another address. 

``` sh
$ cat xxd_3908_state2_ctrlz.txt | grep -e "6ba60"
0006ba60: 6200 6200 6200 6200 6200 6200 6200 6200  b.b.b.b.b.b.b.b.
```

The address turns out to be the string of "b"s, which is the undo text. This means the offset `0x6ba60` contains the pointer to the undo text both before and after `ctrl z`. If we can find a reference to this, maybe we can build a pattern around it. 

``` sh
(gdb) p/x 0x232b06b0000 + 0x5bb00
$3 = 0x232b070bb00
[REMOVED]
$ vol -q -f switch_state2.raw windows.vadyarascan --yara-string "{ 08 bb 70 b0 32 02 }" --pid 3908
vol -q -f switch_state2.raw windows.vadyarascan --yara-string "{ 08 bb 70 b0 32 02 }" --pid 3908
Volatility 3 Framework 2.28.0

Offset  PID     CreateTime      PPID    ImageFileName   SessionId       Threads Rule    Component       Value
```

Unfortunately, I could not find a reference to the address of the undo text pointer. However, viewing the address at `0x5bb00`, we can see it resembles the address of the undo text pointer in PID 4916. 

``` sh
# PID 4916 undo text pointer
$ cat xxd_4916_0x1b6de5a0000.txt | grep 5d9b0
0005d9b0: 0300 0000 0000 0000 8006 60de b601 0000  ..........`.....
# PID 3908 undo text pointer
$ cat diff_switch_state2_ctrlz.txt | grep -e "0005bb00"
< 0005bb00: 0300 0000 0000 0000 90b7 71b0 3202 0000  ..........q.2...
> 0005bb00: 0300 0000 0000 0000 60ba 71b0 3202 0000  ........`.q.2...
```

Using this, perhaps we can find the undo text pointer?

The first 8 bytes are fixed, and the following 8 bytes is the VA of the undo text in little endian. Because the undo text is in the same VAD, we can hardcode the first 5 bytes of the VA.

``` sh
$ cat xxd_49160x1b6de5a0000.txt | grep -E "0300 0000 0000 0000 [0-9a-f]{4} [0-9a-f]{2}de b601 0000" -c
8
```

Unfortunately, there still seems to be too many false positives. We still have to filter some more out. 

I was thinking about how to achieve this when I remembered the pattern found around "text2" of PID 4916. 

If we can find the same pattern at undo text of state2 and state2 after `ctrl z`, that should be enough to filter out false positives. 

In state2, the displayed text is the "b string" which means "a string" should be the undo text, and the opposite for state2 after `ctrl z`. 

``` sh
$ grep -e "a.a.a.a.a.a.a.a." -C 1 xxd_3908_state2.txt
0006b780: 0000 0000 0000 0000 9994 32fd 0001 008e  ..........2.....
0006b790: 6100 6100 6100 6100 6100 6100 6100 6100  a.a.a.a.a.a.a.a.
0006b7a0: 6100 6100 6100 6100 6100 6100 6100 6100  a.a.a.a.a.a.a.a.
0006b7b0: 0000 0000 0000 0000 9a94 3ffd 0002 0080  ..........?.....
$ grep -e "b.b.b.b.b.b.b.b." -C 2 xxd_3908_state2_ctrlz.txt
0006ba40: 4300 6f00 6c00 2000 3700 0000 0000 0000  C.o.l. .7.......
0006ba50: 0000 0000 0000 0000 4494 01fd 0010 0096  ........D.......
0006ba60: 6200 6200 6200 6200 6200 6200 6200 6200  b.b.b.b.b.b.b.b.
0006ba70: 6200 6200 6200 6200 0000 0000 0000 0000  b.b.b.b.........
0006ba80: 0000 0000 0000 0000 4994 02fd 0011 0080  ........I.......
```

Fortunately, these undo text seem to follow the same pattern as "text2" from PID 4916. We can see byte 10 is fixed at `0x94` and byte 12 is fixed at `0xfd`, etc. 

Before creating the plugin, I summarized the discoveries in a simple diagram. 

![[undo_text_layout.png]]

## 3. plugin creation
Using the criteria from the previous post to find the VAD containing the content pointer, we can get the address of the VAD containing the displayed text. 

``` python
def get_vad_for_address(task, target_address):
    for vad in vadinfo.VadInfo.list_vads(task):
        start_addr = vad.get_start()
        end_addr = vad.get_end()

        if start_addr <= target_address <= end_addr:
            return start_addr, end_addr

    return None
```

Next we need to find the undo text pointer. The fixed format enables us to do this easily. 

``` python
    # get address of content VAD in little endian
    raw_address_bytes = start_address.to_bytes(8, byteorder='little')
    address_MS5bytes = raw_address_bytes[3:]
    escaped_address_bytes = re.escape(address_MS5bytes)

    # compile pointer pattern
    pattern_bytes = b'\x03\x00{7}.{3}' + escaped_address_bytes
    pattern = re.compile(pattern_bytes, re.DOTALL)
```

Once we find a candidate for undo text pointer, we can filter out false positives using the structure before and after the undo text. 

``` python
    # byte 2 should match
    if before_text[1] != after_text[1]:
        return False
    # byte 4 should match
    if before_text[3] != after_text[3]:
        return False
    # byte 5 should be 0
    if before_text[4] != 0 or after_text[4] != 0:
        return False
    # byte 6 should be incremented
    if before_text[5] + 1 != after_text[5]:
        return False
    # byte 7 should be 0
    if before_text[6] != 0 or after_text[6] != 0:
        return False
```

The plugin seems to function decently, though there are a few false positives occasionally. 

``` sh
$ vol -c config.json -f notepad_deleted.raw -q windows.note_extractor --pid 4916
Volatility 3 Framework 2.27.0

Virtual Address Content Type    Method  Content Raw Content

0x1b6de5dbaa0   content Bruteforce      window2. text1 new12    77 00 69 00 6e 00 64 00 6f 00 77 00 32 00 2e 00 20 00 74 00 65 00 78 00 74 00 31 00 20 00 6e 00 65 00 77 00 31 00 32 00 20 00
0x1b6de600680   undo text       -       text2   74 00 65 00 78 00 74 00 32 00 20 00
# false positive
0x1b6de636590   undo text       -       dummy   64 00 75 00 6d 00 6d 00 79 00
$ vol -q -f switch_state1.raw windows.note_extractor --pid 3908
[REMOVED]
# no undo text exists yet
0x232b06b4cb0   content StaticCacheVad  aaaaaaaaaaaaaaaa        61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00
$ vol -q -f switch_state2.raw windows.note_extractor --pid 3908
[REMOVED]
0x232b06b4cb0   content StaticCacheVad  bbbbbbbbbbbbaaaa        62 00 62 00 62 00 62 00 62 00 62 00 62 00 62 00 62 00 62 00 62 00 62 00 61 00 61 00 61 00 61 00
0x232b071b790   undo text       -       aaaaaaaaaaaaaaaa        61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00
$ vol -q -f switch_state2_ctrlz.raw windows.note_extractor --pid 3908
[REMOVED]
0x232b06b4cb0   content StaticCacheVad  aaaaaaaaaaaaaaaa        61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00 61 00
0x232b071ba60   undo text       -       bbbbbbbbbbbb    62 00 62 00 62 00 62 00 62 00 62 00 62 00 62 00 62 00 62 00 62 00 62 00
```

Upon further testing, there has been one instance of undo text being omitted. Thus, manual verification and correlation with output from other plugins is recommended. 

### *tested environment*
``` sh
# kernel version
$ vol -c config.json -f notepad_deleted.raw windows.info
[REMOVED]
Major/Minor     15.19041
MachineType     34404
[REMOVED]
# notepad version
$ exiftool 4040.notepad.exe.0x7ff62cf30000.dmp
[REMOVED]
Original File Name              : NOTEPAD.EXE
Product Name                    : Microsoft® Windows® Operating System
Product Version                 : 10.0.19041.1865
```

## 4. Reflection
I need to work on making my posts more readable and easy to understand. 

## 5. Tools used
Custom tooms I used can be found on my Github.

- *note_extractor* and *extract_heap*:
	- https://github.com/DeceptiveRat/Custom_Volatility_Utilities

- memory dumps used:
	- *switch_state1.raw*: https://drive.google.com/file/d/1sm1nB-_tedaczdSl7ZdbAD9GTMbIEn6o/view?usp=sharing
	- *switch_state2.raw*: https://drive.google.com/file/d/19zFMRalRJJ3jbAWj3v-RDNwNfo-LRRWe/view?usp=sharing
	- *switch_state2_ctrlz.raw*: https://drive.google.com/file/d/1DpjrOAa4fwHdt1-nQ_lB_bvOT6lYA5ga/view?usp=sharing
	- *notepad_deleted.raw*: https://drive.google.com/file/d/1vBSBbHFHUCCjFB5ZpBqHgT2-U9ozfk8v/view?usp=sharing
