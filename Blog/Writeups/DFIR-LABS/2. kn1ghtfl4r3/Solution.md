## 1. solution

First let's check if there are any suspicious processes running with *pslist*. 

``` data
$ vol -f chall.raw -r pretty windows.pslist
Volatility 3 Framework 2.27.0
Formatting...0.00		PDB scanning finished
  |  PID | PPID |  ImageFileName |      Offset(V) | Threads | Handles | SessionId | Wow64 |                     CreateTime |                       ExitTime | File output
[REMOVED]
* | 7296 | 3160 | SearchProtocol | 0x9c84debb9080 |       5 |       - |         0 | False | 2024-10-26 17:34:56.000000 UTC |                            N/A |    Disabled
* | 8048 |  656 |    svchost.exe | 0x9c84df16f080 |       4 |       - |         0 | False | 2024-10-26 17:35:17.000000 UTC |                            N/A |    Disabled
* | 7492 |  772 |    dllhost.exe | 0x9c84dbed4080 |      12 |       - |         1 | False | 2024-10-26 17:35:43.000000 UTC |                            N/A |    Disabled
* | 1116 | 5016 | powershell.exe | 0x9c84de9f5080 |      12 |       - |         1 | False | 2024-10-26 17:35:45.000000 UTC |                            N/A |    Disabled
* | 3932 | 1116 |    conhost.exe | 0x9c84de6a8340 |       5 |       - |         1 | False | 2024-10-26 17:35:45.000000 UTC |                            N/A |    Disabled
* | 2672 | 5016 |    KeePass.exe | 0x9c84de6c40c0 |       8 |       - |         1 | False | 2024-10-26 17:36:39.000000 UTC |                            N/A |    Disabled
* | 4520 |  656 |    svchost.exe | 0x9c84dfa2c080 |      13 |       - |         0 | False | 2024-10-26 17:36:39.000000 UTC |                            N/A |    Disabled
* | 5464 |  656 |    svchost.exe | 0x9c84dbad3080 |       5 |       - |         1 | False | 2024-10-26 17:36:43.000000 UTC |                            N/A |    Disabled
* | 7452 |  772 | RuntimeBroker. | 0x9c84e3fae080 |       8 |       - |         1 | False | 2024-10-26 17:37:05.000000 UTC |                            N/A |    Disabled
* | 7128 | 5016 |    notepad.exe | 0x9c84de9f2080 |       6 |       - |         1 | False | 2024-10-26 17:37:07.000000 UTC |                            N/A |    Disabled
* | 3484 | 7128 |    notepad.exe | 0x9c84decee080 |       7 |       - |         1 | False | 2024-10-26 17:37:11.000000 UTC |                            N/A |    Disabled
* |  924 | 3484 |    notepad.exe | 0x9c84de30f080 |       6 |       - |         1 | False | 2024-10-26 17:37:20.000000 UTC |                            N/A |    Disabled
* | 3416 | 2112 |    audiodg.exe | 0x9c84dbee42c0 |       8 |       - |         0 | False | 2024-10-26 17:37:26.000000 UTC |                            N/A |    Disabled
* | 6588 | 5016 |     DumpIt.exe | 0x9c84de3af2c0 |       3 |       - |         1 |  True | 2024-10-26 17:37:27.000000 UTC |                            N/A |    Disabled
* | 7256 | 6588 |    conhost.exe | 0x9c84e03a52c0 |       7 |       - |         1 | False | 2024-10-26 17:37:27.000000 UTC |                            N/A |    Disabled
* | 7956 |  656 |    svchost.exe | 0x9c84df171080 |       6 |       - |         0 | False | 2024-10-26 17:37:29.000000 UTC |                            N/A |    Disabled
```

A few processes catch my eye; *powershell.exe*, *KeePass.exe*, and *notepad.exe*. *powershell.exe* and *notepad.exe* could contain interesting data. *KeePass.exe* hints at a *.kdbx* file somewhere in the system.

Also, I like to check the MFT entries to see if there are any interesting files.

``` data
$ vol -f chall.raw -r csv windows.mftscan.MFTScan > mft.csv
$ vol -f chall.raw windows.mftscan.ResidentData >mft_resident_data.txt
$ vim mft_resident_data.txt
[REMOVED]
0x101e5e520     FILE    91953   DATA    note.txt
54 6f 20 57 68 6f 6d 20 49 74 20 4d 61 79 20 43 To Whom It May C
6f 6e 63 65 72 6e 2c 0d 0a 0d 0a 49 e2 80 99 76 oncern,....I...v
65 20 6c 65 66 74 20 73 6f 6d 65 74 68 69 6e 67 e left something
20 76 61 6c 75 61 62 6c 65 20 68 69 64 64 65 6e  valuable hidden
20 77 69 74 68 69 6e 20 79 6f 75 72 20 73 79 73  within your sys
74 65 6d 2c 20 6c 6f 63 6b 65 64 20 61 77 61 79 tem, locked away
20 77 68 65 72 65 20 6f 6e 6c 79 20 74 68 65 20  where only the
63 6c 65 76 65 72 20 77 69 6c 6c 20 66 69 6e 64 clever will find
20 69 74 2e 20 59 6f 75 20 6d 69 67 68 74 20 77  it. You might w
61 6e 74 20 74 6f 20 63 68 65 63 6b 20 74 68 65 ant to check the
20 6c 6f 6f 73 65 20 65 6e 64 73 e2 80 94 74 68  loose ends...th
69 6e 67 73 20 79 6f 75 20 64 69 64 6e e2 80 99 ings you didn...
74 20 73 61 76 65 2c 20 6f 72 20 70 65 72 68 61 t save, or perha
70 73 20 66 69 6c 65 73 20 79 6f 75 20 74 68 6f ps files you tho
75 67 68 74 20 77 65 72 65 20 73 61 66 65 20 62 ught were safe b
75 74 20 77 65 72 65 6e e2 80 99 74 2e 0d 0a 0d ut weren...t....
0a 54 68 65 20 70 61 74 68 20 77 6f 6e e2 80 99 .The path won...
74 20 62 65 20 65 61 73 79 2e 20 53 6f 6d 65 20 t be easy. Some
64 6f 6f 72 73 20 61 72 65 20 73 74 69 6c 6c 20 doors are still
6c 6f 63 6b 65 64 2c 20 6f 74 68 65 72 73 20 62 locked, others b
75 72 69 65 64 20 62 65 6e 65 61 74 68 20 6c 61 uried beneath la
79 65 72 73 20 6f 66 20 70 72 6f 74 65 63 74 69 yers of protecti
6f 6e 2e 20 49 20 77 6f 6e 64 65 72 20 69 66 20 on. I wonder if
79 6f 75 e2 80 99 6c 6c 20 66 69 67 75 72 65 20 you...ll figure
6f 75 74 20 77 68 65 72 65 20 74 6f 20 73 74 61 out where to sta
72 74 2c 20 6f 72 20 77 69 6c 6c 20 74 68 65 20 rt, or will the
61 6e 73 77 65 72 20 73 6c 69 70 20 70 61 73 74 answer slip past
20 79 6f 75 20 61 73 20 74 68 65 20 63 6c 6f 63  you as the cloc
6b 20 74 69 63 6b 73 3f 0d 0a 0d 0a 47 6f 6f 64 k ticks?....Good
20 6c 75 63 6b 20 66 69 6e 64 69 6e 67 20 6d 65  luck finding me
2e                                              .
[REMOVED]
```

There weren't a lot of entries with resident data, but I did find an interesting note. 

Now that we have that out of the way, let's check the processes we found. Out of the 3, *notepad.exe* seems the most interesting so let's examine them next. 

``` data
$ vol -f chall.raw windows.memmap --pid 7128 --dump > PID_7128.map
$ vol -f chall.raw windows.memmap --pid 3484 --dump > PID_3484.map
$ vol -f chall.raw windows.memmap --pid 924 --dump > PID_924.map
$ strings -td -a pid.7128.dmp > strings_7128.txt
$ strings -td -el -a pid.7128.dmp >> strings_7128.txt
$ strings -td -a pid.924.dmp >strings_924.txt
$ strings -td -el -a pid.924.dmp >> strings_924.txt
$ strings -td -a pid.3484.dmp > strings_3484.txt
$ strings -td -el -a pid.3484.dmp >> strings_3484.txt
$ cat strings_7128.txt | grep -i -e "keepass"
[REMOVED]
434160784 "C:\Program Files\KeePass Password Safe 2\KeePass.exe" "C:\Users\kimsh\Downloads\Keylist.kdbx"
[REMOVED]
```

I am not sure why, but there are KeePass related text inside PID 7128 and we can find the name of the *.kdbx* file.

Let's see if we can find the *.kdbx* file on memory.

``` data
$ vol -f chall.raw windows.filescan > filescan.txt
$ cat filescan.txt | grep -F -i -e "download" -e "keypass"
[REMOVED]
0x9c84e3f0caa0	\Users\kimsh\Downloads\KeePass-2.53.1-Setup.exe
0x9c84e3f0cc30	\Users\kimsh\Downloads\VeraCrypt Setup 1.26.15.exe
0x9c84e3f0ed00	\Users\kimsh\Downloads\container.png
0x9c84e3f0ee90	\Users\kimsh\Downloads\note.txt
0x9c84e3f0fe30	\Users\kimsh\Downloads\vault.hc
[REMOVED]
```

Unfortunately, the *.kdbx* file doesn't seem to be on memory. However, in the `\Downloads` directory, we can see some interesting files. 

The version number of *KeePass* and *VeraCrypt* can be seen. Also, there is the strange *note.txt* file from earlier and *vault.hc* which turns out to be a *VeraCrypt* container file. 

Let's see if we can extract the *vault.hc* file. 

``` data
$ vol -f chall.raw windows.dumpfiles --virtaddr 0x9c84e3f0fe30
```

When we try to mount the file with *VeraCrypt*, we get a prompt for the password, as expected. 

We do not have the password right now, but the *note.txt* provides a hint as to where to search. 
> You might want to check the loose ends things you didn't save, or perhaps files you thought were safe but weren't. 

This seems to point to *notepad.exe*. 

Let's start from PID 7128 and work our way to PID 924. 

Since the user input should be in the heap, we first have to find the heap addresses.

``` data
$ volshell -w -f chall.raw --pid 7128
[REMOVED]
(layer_name_Process7128_2) >>> for entry in ps():
...     print(entry)
...     print(entry.UniqueProcessId)
[REMOVED]
<EPROCESS symbol_table_name1!_EPROCESS: layer_name @ 0x9c84de9f2080 #2624>
7128
[REMOVED]
(layer_name_Process7128_2) >>> dt("_EPROCESS", 0x9c84de9f2080)
[REMOVED]
  0x550 :   Peb                                    *symbol_table_name1!_PEB                                   0xbb4c76f000
[REMOVED]
(layer_name_Process7128_3) >>> dt("_PEB", 0xbb4c76f000)
[REMOVED]
   0x30 :   ProcessHeap                              *symbol_table_name1!void                             0x2a217e00000
[REMOVED]
   0xe8 :   NumberOfHeaps                            symbol_table_name1!unsigned long                     4
   0xec :   MaximumNumberOfHeaps                     symbol_table_name1!unsigned long                     16
   0xf0 :   ProcessHeaps                             **symbol_table_name1!void                            0x7ffea9a3ad40
[REMOVED]
(layer_name_Process7128_3) >>> dd(0x7ffea9a3ad40)
0x7ffea9a3ad40    17e00000 000002a2 17c60000 000002a2    .... .... .... ....
0x7ffea9a3ad50    17df0000 000002a2 197f0000 000002a2    .... .... .... ....
```

Now we have the heap addresses, we can dump them and extract their strings. From those strings, I manually extracted strings that could potentially be the password.

``` data
$ vol -f chall.raw -o PID_7128 windows.vadinfo --pid 7128 --dump
$ ll | grep -e 0x2a217e00000 -e 0x2a217c60000 -e 0x2a217df0000 -e 0x2a2197f0000 | awk '{print $NF}' | xargs -I {} strings -a {} > heap_strings
$ ll | grep -e 0x2a217e00000 -e 0x2a217c60000 -e 0x2a217df0000 -e 0x2a2197f0000 | awk '{print $NF}' | xargs -I {} strings  -el -a {} >> heap_strings
$ cat heap_strings | grep -F -v -i -e "\\" -e  " " -e "\;" -e "t0Pq" -e "system" -e "?pq" -e ".exe" -e "ext-ms-win-core" -e "Windows" -e ".dll" -e "S-1-15-3" -e "-"
```

Using the same method for the remaining 2 PIDs, I created a list of potential passwords. 

Passing the password list along with the *.hc* file we dumped earlier, we can crack the file. 

``` data
$ hashcat -a 0 -m 13711 -o cracked.txt file.0x9c84e3f0fe30.0x9c84dbc4e590.DataSectionObject.vault.hc.dat passwords.txt
$ hashcat -a 0 -m 13712 -o cracked.txt file.0x9c84e3f0fe30.0x9c84dbc4e590.DataSectionObject.vault.hc.dat passwords.txt
$ hashcat -a 0 -m 13713 -o cracked.txt file.0x9c84e3f0fe30.0x9c84dbc4e590.DataSectionObject.vault.hc.dat passwords.txt
$ hashcat -a 0 -m 13721 -o cracked.txt file.0x9c84e3f0fe30.0x9c84dbc4e590.DataSectionObject.vault.hc.dat passwords.txt
[REMOVED]
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
[REMOVED]
$ cat cracked.txt
file.0x9c84e3f0fe30.0x9c84dbc4e590.DataSectionObject.vault.hc.dat:Y454wE3kh7
```

The correct password, Y454wE3kh7, is found and we can now mount the *.hc* file with *VeraCrypt*.

Upon mounting the decrypted volume, we can see the following.

``` data
$ ll
total 8764
drwx------ 1 sansforensics sansforensics       0 Oct 26  2024 '$RECYCLE.BIN'/
drwx------ 1 sansforensics sansforensics    4096 Oct 26  2024  ./
drwxr-xr-x 4 root          root             4096 Apr 20 00:10  ../
drwx------ 1 sansforensics sansforensics       0 Oct 26  2024 'System Volume Information'/
-rwx------ 1 sansforensics sansforensics 8963778 Oct 22  2024  project.zip*
```

Let's try unzipping the *project.zip* file.

``` data
$ unzip project.zip
Archive:  project.zip
   creating: project/
[project.zip] project/Confidential password:
```

Unzipping requires another password and the previous one, Y454wE3kh7, does not work. Maybe we can try cracking it again with the password list from earlier?

``` data
$ fcrackzip -D -p potential_passwords.txt -v project.zip
'project/' is not encrypted, skipping
found file 'project/Confidential', (size cp/uc 320812/320800, flags 1, chk 1ebc)
found file 'project/enc.exe', (size cp/uc 8639848/8823677, flags 1, chk 92e1)
found file 'project/Keylist.kdbx', (size cp/uc   2522/  2510, flags 1, chk dd0b)
```

Unfortunately, this doesn't seem to be the correct method. 

After a few hours of trying out different plugins, I found something with the `envars` plugin.

``` data
$ vol -f chall.raw windows.envars > envars.txt
$ cat envars.txt | grep -F -i -e "notepad.exe" -e "powershell.exe" -e "keepass.exe"
[REMOVED]
1116	powershell.exe	0x24112d11c00	zippa4s5wo0rD	MP0Z\CRJws&4e\c':=3k
[REMOVED]
```

Using the newly found password, it is possible to unzip the *project.zip* file.

``` data
$ unzip project.zip
Archive:  project.zip
   creating: project/
[project.zip] project/Confidential password:
 extracting: project/Confidential
  inflating: project/enc.exe
 extracting: project/Keylist.kdbx
```

The *Confidential* file seems to be some obfuscated data. 

``` data
$ xxd -l 4096 Confidential
00000000: 8223 7839 5228 e77f 4707 a629 0755 b314  .#x9R(..G..).U..
00000010: d807 5d35 5b5a 98a1 9ce0 86ec c738 79ef  ..]5[Z.......8y.
00000020: 05d8 a293 6c09 4f52 4bc6 7472 79ca b04f  ....l.ORK.try..O
00000030: ceb7 090a 8707 0e68 4a4a f96d 32ab 4c64  .......hJJ.m2.Ld
00000040: 545a 9d03 0427 b697 2a87 73ca 060b c582  TZ...'..*.s.....
[REMOVED]
```

Let's see if we can find anything with *enc.exe*.

``` data
$ strings -n 10 enc.exe > long_strings.enc
$ strings -n 10 -el enc.exe >> long_strings.enc
$ vim long_strings.enc
[REMOVED]
Extraction path length exceeds maximum path length!
File already exists but should not: %s
Failed to create parent directory structure.
Failed to extract entry: %s.
Could not get __main__ module.
Could not get __main__ module's dict.
Failed to extract script from archive!
Absolute path to script exceeds PYI_PATH_MAX
Failed to unmarshal code object for %s
_pyi_main_co
Failed to execute script '%s' due to unhandled exception!
PYINSTALLER_RESET_ENVIRONMENT
_PYI_ARCHIVE_FILE
_PYI_APPLICATION_HOME_DIR
_PYI_PARENT_PROCESS_LEVEL
_PYI_SPLASH_IPC
Invalid value in _PYI_PARENT_PROCESS_LEVEL: %s
PYINSTALLER_STRICT_UNPACK_MODE
Failed to initialize security descriptor for temporary directory!
[REMOVED]
```

After a quick Google search I found out this meant *enc.exe* was created using *PyInstaller*. Also, it meant I could extract the original python code from the executable.

``` data
$ python3 pyinstxtractor.py project/enc.exe
[REMOVED]
[+] Successfully extracted pyinstaller archive: project/enc.exe
$ uncompyle6 enc.exe_extracted/enc.pyc
[REMOVED]
    cipher = Cipher((algorithms.AES(key)), (modes.CBC(iv)), backend=(default_backend()))
    encryptor = cipher.encryptor()
    padder = padding.PKCS7(128).padder()
[REMOVED]
key = b'{REDACTED}'
iv = bytes.fromhex("3038375330a3546850283a2d695b7e23")
original_file = "Confidential"
encrypt_file(original_file, key, iv)
[REMOVED]
```

You can see the encryption method, AES in CBC mode, and the IV that was used to create the *Confidential* file. Unfortunately however, the key was redacted and we will have to find that out. 

It seems likely the key is inside the *Keylist.kdbx* file that is located in the same directory. This file also requires a password, but a Google search for "KeePass memory dump" shows `CVE-2023-32784`.

> In KeePass 2.x before 2.54, it is possible to recover the cleartext master password from a memory dump, even when a workspace is locked or no longer running. The memory dump can be a KeePass process dump, swap file (pagefile.sys), hibernation file (hiberfil.sys), or RAM dump of the entire system. The first character cannot be recovered.

As we saw earlier, the *KeePass* version installed is 2.53.1, meaning the CVE can be exploited.

``` data
$ dotnet run --roll-forward LatestMajor /home/sansforensics/Downloads/pid.2672.dmp
1.:	●
2.:	0, Ú,  a, U, ), N, V, Á, O, S, I, 2, !, , 1, .,  , #, û, g, ¯,
3.:	r,
4.:	p,
5.:	_,
6.:	R,
7.:	X, ð,
8.:	_,
9.:	2,
10.:	4,
11.:	3,
12.:	1,
13.:	2,
Combined: ●{0, Ú,  a, U, ), N, V, Á, O, S, I, 2, !, , 1, .,  , #, û, g, ¯}rp_R{X, ð}_24312
```

Using *keepass-password-dumper*, I extracted the password. 

However, as it said in the CVE, the first character wasn't recovered and some other characters have multiple candidates. Using a simple python script, I created a list of potential passwords. 

*createlist.py*:
``` python
# characters for the first position (adjust as needed)
pos1 = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789!@#$%^&*()"

# your specific set for the second position
pos2 = ["0", "Ú", "a", "U", ")", "N", "V", "Á", "O", "S", "I", "2", "!", " ", "1", ".", "#", "û", "g", "¯"]
pos3 = "rp_R"
pos4 = ["X", "ð"]
pos5 = "_24312"

with open("wordlist.txt", "w", encoding="utf-8") as f:
    for c1 in pos1:
        for c2 in pos2:
            for c4 in pos4:
                f.write(f"{c1}{c2}{pos3}{c4}{pos5}\n")
```

With the password list, all we need are hash of *Keylist.kdbx* to crack. We can get the hash using *keepass2john*.

``` data
$ keepass2john Keylist.kdbx > keylist.hash
```

After cleaning the hash to fit the format, we can run hashcat to crack the password.

``` data
$ hashcat -a 0 -o keepass_cracked.txt -m 13400 Keylist_clean.hash wordlist.txt
[REMOVED]
Status...........: Cracked
[REMOVED]
$ cat keepass_cracked.txt
[REMOVED]c0rp_RX_24312
```

Upon opening *Keylist.kdbx*, in the recycle bin we can find the AES key, which is made apparent by the name, orgAES.

The key, ``bFFN5ss02~`76/8H2jHq#(~qWIB8d?p%``, is converted to hexadecimal `0x6246464e35737330327e6037362f3848326a487123287e7157494238643f7025` and used with other encryption parameters to decrypt *Confidential*.

``` data
$ openssl enc -aes-256-cbc -d -in Confidential -out Confidential.dec -K 6246464e35737330327e6037362f3848326a487123287e7157494238643f7025 -iv 3038375330a3546850283a2d695b7e23
$ file Confidential.dec
Confidential.dec: PDF document, version 1.5, 1 page(s)
```

The decrypted file seems to be a pdf file. Opening it, we can find some hidden text within the the file, revealing the flag.

![[Confidential_text.png]]

``` data
CONFIDENTIAL
Gotham's fate rests in your hands.
Hidden within your system is a code that holds the city's future.
Your objective: retrieve it before the shadows consume everything.
The code is: icc{15_17_d4rkkn1gh7_0r_4zr34lkn1gh7}
Trust nothing. The clues are buried deeper than you think. Time is running out.
```

flag: `icc{15_17_d4rkkn1gh7_0r_4zr34lkn1gh7}`

## 2. Reflection

This one was not easy for me to solve and took quite a long time for me to get right. 

However, I was able to learn many things that are very useful for digital forensics. Namely, extracting the original python code within a *PyInstaller* created executable and cracking passwords with *Hashcat*.
