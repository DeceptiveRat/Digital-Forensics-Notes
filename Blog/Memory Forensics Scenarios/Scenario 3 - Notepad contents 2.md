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
$ strings -td -el -a notepad_deleted.raw > strings_long.txt
$ vol -c config.json -f notepad_deleted.raw windows.strings --pid 4916 --strings-file strings_long.txt > translated_strings.txt

