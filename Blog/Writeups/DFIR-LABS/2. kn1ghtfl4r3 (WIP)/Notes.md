### pslist
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

### notepad.exe dump and strings
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
```

- 7128 strings:
``` data
$ cat strings_7128.txt | grep -i -e "keepass"
[REMOVED]
203354788 [{"application":"{6D809377-6AF0-444B-8957-A3773F02200E}\\KeePass Password Safe 2\\KeePass.exe","platform":"windows_win32"},{"application":"{7C5A40EF-A0FB-4BFC-874A-C0F2E0B9FA8E}\\KeePass Password Safe 2\\KeePass.exe","platform":"windows_win32"},{"application":"{6D809377-6AF0-444B-8957-A3773F02200E}\\KeePass Password Safe 2\\KeePass.exe","platform":"packageId"},{"application":"","platform":"alternateId"}]6MYBtTFFGOjLzXy4t88PskM+uG9H5+ahA9mrDwevVoM=ECB32AF3-1440-4086-94E3-5311F97F89C4\{Local Downloads}\Keylist.kdbx
[REMOVED]
337991658 qpackageid{6d809377-6af0-444b-8957-a3773f02200e}\keepass password safe 2\keepass.exegD
[REMOVED]
345098381 c:\users\kimsh\appdata\local\temp\is-09q7j.tmp\keepass-2.53.1-setup.tmpx_exe_path
[REMOVED]
289482674 When checked, the trigger will initially be on when KeePass starts.
289599776 In order to avoid data loss, it is recommended to install/use the latest version of KeePass.
289600098 KeePass Data Editor
289600139 If you lose the database file or any of the master key components (or forget the composition), all data stored in the database is lost. KeePass does not have any built-in file backup functionality. There is no backdoor and no universal key that can open your database.
289600677 KeePass Data Viewer
289603308 A KeePass emergency sheet contains all important information that is required to open your database. It should be printed, filled out and 
[REMOVED]
434160784 "C:\Program Files\KeePass Password Safe 2\KeePass.exe" "C:\Users\kimsh\Downloads\Keylist.kdbx"
434210896 KeePass Password Safe 2KEEPAS~1
[REMOVED]
```

### keepass
``` data
$ cat strings.txt | grep -C 4 -i -e "keepass" >strings_keepass.txt
[REMOVED]
131741914 <.accountName=%s&password=%s&passwordConfirm=%s&secretQuestionId=0&secretQuestionAnswer=test&submit.x=80&submit.y=18signup.worldofwarcraft.com/accountname.htmlfirstName=%s&lastName=%s&addressLine1=%d+Broadway&addressLine2=&city=New+York&state=NY&postalCode=10023&countryId=USA&email=%s@hotmail.com&AutoUpdate.exeEMailClient.LogRename = "ssm.exe"!#ALF:Y2V:FireEye/HackTool_MSIL_SharPersist_2
131742314 6RXxf
131742340 tSharPersist.libSharPersist.exeERROR: Invalid hotkey location option given.E
131742503 ERROR: Invalid hotkey given.E
131742587 ERROR: Keepass configuration file not found.E
131742719 ERROR: Keepass configuration file was not found.E
131742863 ERROR: That value already exists in:E
131742971 ERROR: Failed to delete hidden registry key.E
131743103 \SharPersist\\SharPersist.pdb!#HSTR:Win32/Guloader.KSST123!MTB
[REMOVED]
141620184 C:\Windows\system32\svchost.exe -k LocalSystemNetworkRestricted -p -s PcaSvc
141620348 p\Device\HarddiskVolume2\Program Files\KeePass Password Safe 2\KeePass.exe
141620424 "C:\Program Files\KeePass Password Safe 2\KeePass.exe" "C:\Users\kimsh\Downloads\Keylist.kdbx"
141620604 p\Device\HarddiskVolume2\Windows\System32\svchost.exe
141620659 C:\Windows\system32\svchost.exe -k netsvcs -p -s lfsvc
[REMOVED]
194266240 grab_passwords_chrome()from logins where blacklisted_by_user = 0\default\login data.bak
194266328 !Trickbot.O
194266366 km9o
194266375 O[reflection.assembly]::loadfile("
194266411  \keepass.exe")
194266428 MTIzNA==; cXdlcg==;
194266448 !TrickbotVP.A!MTB
194266501 nvpnDll build %s %s startedv
194266580 VPN bridge failureV
[REMOVED]
```

- CVE abuse:
``` data
$ dotnet run --roll-forward LatestMajor /home/sansforensics/Downloads/2672.KeePass.exe.0x4f0000.dmp
[REMOVED]
Password candidates (character positions):
Unknown characters are displayed as "●"
1.:	●
Combined: ●
$ python3 keepass_dump.py -f 2672.KeePass.exe.0x4f0000.dmp 
[*] Searching for masterkey characters
[-] Couldn't find jump points in file. Scanning with slower method.
[*] 0:	{UNKNOWN}
[-] couldn't find any characters
```

- finding heap:
``` data
$ vol -f chall.raw -o PID_2672 -r csv windows.vadinfo --pid 2672 --dump > PID_2672/vadinfo_2672.csv
[REMOVED]
(layer_name_Process2672_1) >>> dt("_EPROCESS", 0x9c84de6c40c0)
[REMOVED]
  0x550 :   Peb                                    *symbol_table_name1!_PEB                                   0xa2f000
[REMOVED]
(layer_name_Process2672_1) >>> dt("_PEB", 0xa2f000)
[REMOVED]
   0xe8 :   NumberOfHeaps                            symbol_table_name1!unsigned long                     11
   0xec :   MaximumNumberOfHeaps                     symbol_table_name1!unsigned long                     16
   0xf0 :   ProcessHeaps                             **symbol_table_name1!void                            0x7ffea9a3ad40
[REMOVED]
(layer_name_Process2672_1) >>> dd(0x7ffea9a3ad40, count=88)
0x7ffea9a3ad40    00de0000 00000000 00810000 00000000    .... .... .... ....
0x7ffea9a3ad50    009f0000 00000000 01040000 00000000    .... .... .... ....
0x7ffea9a3ad60    029d0000 00000000 02930000 00000000    .... .... .... ....
0x7ffea9a3ad70    029c0000 00000000 1b480000 00000000    .... .... .H.. ....
0x7ffea9a3ad80    1b670000 00000000 1b370000 00000000    .g.. .... .7.. ....
0x7ffea9a3ad90    227b0000 00000000                      "{.. ....
```
- 11 heap addresses found:
	- 0xde0000
	- 0x810000
	- 0x9f0000
	- 0x1040000
	- 0x29d0000
	- 0x2930000
	- 0x29c0000
	- 0x1b480000
	- 0x1b670000
	- 0x1b370000
	- 0x227b0000

- viewing heap:
``` data
$ cat heap_addresses.txt 
0xde0000
0x810000
0x9f0000
0x1040000
0x29d0000
0x2930000
0x29c0000
0x1b480000
0x1b670000
0x1b370000
0x227b0000
$ ll | grep -f heap_addresses.txt | awk '{print $NF}' | xargs -I {} strings -a {} > heap_strings
$ ll | grep -f heap_addresses.txt | awk '{print $NF}' | xargs -I {} strings -el -a {} >> heap_strings
$ cat heap_strings
[REMOVED]
0${"
P~{"
C:\Program Files\KeePass Password Safe 2\KeePass.exe
C:\Users\kimsh\Downloads\Keylist.kdbx
ALLUSERSPROFILE=C:\ProgramData
        APPDATA=C:\Users\kimsh\AppData\Roaming
[REMOVED]
://www.sajatypeworks.com
[REMOVED]
http://www.monotype.com/html/mtname/
[REMOVED]
```

- mft entries
``` data
$ csvtool col 3,4,6,8,13 mft.csv | csvtool readable - | grep -F -i -e "keylist" -e "Attribute"
Record Type Record Number MFT Type  Attribute Type       Filename
FILE        91950         File      FILE_NAME            Keylist.kdbx
FILE        318           File      FILE_NAME            Keylist.kdbx.lnk
FILE        318           File      FILE_NAME            Keylist.kdbx.lnk
FILE        318           File      FILE_NAME            Keylist.kdbx.lnk
```

### mft
``` data
$ vol -f chall.raw -r csv windows.mftscan.MFTScan > mft.csv
$ csvtool col 3,4,6,8,13 mft.csv | csvtool readable - | grep -F -i -e ".pf" | grep -F -v "~" | xsel -ib
FILE        46456         File      FILE_NAME            MSEDGE.EXE-78F14B87.pf
[REMOVED]
FILE        110607        File      FILE_NAME            BACKGROUNDTRANSFERHOST.EXE-CC9DA465.pf
FILE        101095        File      FILE_NAME            PHONEEXPERIENCEHOST.EXE-D8B241AE.pf
FILE        46410         File      FILE_NAME            SPPSVC.EXE-B0F8131B.pf
FILE        109260        File      FILE_NAME            FILESYNCCONFIG.EXE-8E0E69E8.pf
FILE        111506        File      FILE_NAME            KEEPASS-2.53.1-SETUP.TMP-58F68041.pf
FILE        112028        File      FILE_NAME            KEEPASS.EXE-CD8A3C32.pf
FILE        108947        File      FILE_NAME            LOCALBRIDGE.EXE-BF9EA5D6.pf
FILE        106586        File      FILE_NAME            SVCHOST.EXE-056162C1.pf
FILE        106587        File      FILE_NAME            MICROSOFTEDGEUPDATE.EXE-C4317749.pf
FILE        107521        File      FILE_NAME            SETUP.EXE-0A6B6F28.pf
FILE        107522        File      FILE_NAME            MICROSOFTEDGE_X64_130.0.2849.-9E0EDFD1.pf
FILE        96910         File      FILE_NAME            MPCMDRUN.EXE-F401FBB4.pf
FILE        106877        File      FILE_NAME            NGENTASK.EXE-4F8BD802.pf
[REMOVED]
```

- lnk files:
``` data
$ csvtool col 3,4,6,8,13 mft.csv | csvtool readable - | grep -F -i -e "lnk" -e "Attribute" | grep -v -e "~"
Record Type Record Number MFT Type  Attribute Type       Filename
FILE        46600         File      FILE_NAME            Windows PowerShell.lnk
FILE        46601         File      FILE_NAME            Windows PowerShell (x86).lnk
FILE        46602         File      FILE_NAME            Run.lnk
FILE        46603         File      FILE_NAME            File Explorer.lnk
FILE        109421        File      FILE_NAME            Bluetooth File Transfer.LNK
FILE        104871        File      FILE_NAME            Microsoft Edge.lnk
FILE        112032        File      FILE_NAME            VeraCryptExpander.lnk
FILE        112034        File      FILE_NAME            VeraCrypt.lnk
FILE        265           File      FILE_NAME            ms-gamingoverlay--kglcheck-.lnk
FILE        103832        File      FILE_NAME            Windows PowerShell.lnk
FILE        318           File      FILE_NAME            Keylist.kdbx.lnk
FILE        319           File      FILE_NAME            Downloads.lnk
FILE        33336         File      FILE_NAME            Notepad.lnk
FILE        33336         File      FILE_NAME            Notepad.lnk
FILE        33336         File      FILE_NAME            Notepad.lnk
FILE        33337         File      FILE_NAME            Registry Editor.lnk
FILE        33337         File      FILE_NAME            Registry Editor.lnk
FILE        33337         File      FILE_NAME            Registry Editor.lnk
FILE        108094        File      FILE_NAME            Desktop.lnk
FILE        108095        File      FILE_NAME            Downloads.lnk
FILE        318           File      FILE_NAME            KEYLISͱ1.LNK
FILE        318           File      FILE_NAME            Keylist.kdbx.lnk
FILE        46627         File      FILE_NAME            10 - AppsAndFeatures.lnk
[REMOVED]
FILE        112031        File      FILE_NAME            VeraCrypt.lnk
FILE        110008        File      FILE_NAME            Personal Vault.lnk
FILE        107935        File      FILE_NAME            Internet Explorer.lnk
[REMOVED]
```

- resident data:
``` data
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

### note.txt analysis
- mft entry
``` data
$ cat mft.csv | grep note.txt
1,0x4e23dd70,FILE,91953,1,File,Archive,FILE_NAME,2024-10-26 17:34:50.000000 UTC,2024-10-26 17:34:50.000000 UTC,2024-10-26 17:34:50.000000 UTC,2024-10-26 17:34:50.000000 UTC,note.txt
1,0x101e5e4b0,FILE,91953,1,File,Archive,FILE_NAME,2024-10-26 17:34:50.000000 UTC,2024-10-26 17:34:50.000000 UTC,2024-10-26 17:34:50.000000 UTC,2024-10-26 17:34:50.000000 UTC,note.txt
```
- `2024-10-26 17:34:50.000000 UTC` created

- search string
``` data
$ cat strings.txt | grep -F -e "To Whom It May Concern" > notetxt_strings.txt
$ vol -f chall.raw windows.strings --strings-file notetxt_strings.txt > notetxt_translated.txt
$ cat notetxt_strings.txt 
1219676976 To Whom It May Concern,
1438777343 ETo Whom It May Concern,
4326810912 To Whom It May Concern,
336586184 To Whom It May Concern,
1482438327 To Whom It May Concern,
1652887324 To Whom It May Concern,
6021313722 To Whom It May Concern,
6021648716 To Whom It May Concern,
6021773633 To Whom It May Concern,
```

### `2024-10-26 17:34:50.000000 UTC`
``` data
$ cat mft.csv | grep -F -e "2024-10-26 17:34" | cut -d, -f 13 | sort | uniq
[REMOVED]
CACHED~1
CACHED~1.JPG
CACHE_~1
CONTAI~1.PNG
[REMOVED]
CachedImage_1600_1200_POS4.jpg
[REMOVED]
LOG
LOG.old
LOG.old~RF5bf9e.TMP
LOG.old~RF5bfbd.TMP
LOG.old~RF5bffc.TMP
LOG.old~RF5c02b.TMP
LOG.old~RF5c03a.TMP
LOG.old~RF5c115.TMP
LOG.old~RF5c153.TMP
LOG.old~RF5c26d.TMP
LOG.old~RF5e9ac.TMP
LOGOLD~1.TMP
Local State~RF5c03a.TMP
Local State~RF5c173.TMP
Local State~RF5c182.TMP
[REMOVED]
TH1Y4X~1.JPG
TH5UHK~1.JPG
THEAVS~1.JPG
THIKVJ~1.JPG
THJZK9~1.JPG
THK0OT~1.JPG
THK3FB~1.JPG
THNWP8~1.JPG
THOLVJ~1.JPG
THP6YD~1.JPG
THPAE4~1.JPG
THRLNY~1.JPG
THTL34~1.JPG
THV4LB~1.JPG
THWPA7~1.JPG
THXNXC~1.JPG
TRUSTE~1.PB
TRUSTE~1.TMP
UPA0EE~1.LOG
[REMOVED]
container.png
d44c20a7be687d8f.temp.ini
data.dat
data.dat.tmp
data_0
data_1
data_2
data_3
[REMOVED]
th1Y4XL3AT.jpg
th5UHKOVZV.jpg
thEAVSXNNN.jpg
thIKVJJ3PR.jpg
thJZK94U8C.jpg
thK0OTRXOW.jpg
thK3FBD7ZE.jpg
thNWP82PXU.jpg
thOLVJAYCD.jpg
thP6YDUDY1.jpg
thPAE429DZ.jpg
thRLNYA6FK.jpg
thTL346ZNV.jpg
thV4LBENLO.jpg
thWPA7Q220.jpg
thXNXC0UNN.jpg
[REMOVED]
```

### file dump
``` data
$ vol -f chall.raw windows.filescan > filescan.txt
$ cat filescan.txt
[REMOVED]
0x9c84db84c5c0  \Windows\System32\winevt\Logs\System.evtx
[REMOVED]
0x9c84db84ca70  \Windows\System32\winevt\Logs\Application.evtx
[REMOVED]
0x9c84db84d6f0  \Windows\System32\winevt\Logs\Security.evtx
[REMOVED]
$ cat filescan.txt | grep -F -i -e "download" -e "keypass"
[REMOVED]
0x9c84e2ac60c0	\Users\kimsh\Downloads\desktop.ini
0x9c84e3f0caa0	\Users\kimsh\Downloads\KeePass-2.53.1-Setup.exe
0x9c84e3f0cc30	\Users\kimsh\Downloads\VeraCrypt Setup 1.26.15.exe
0x9c84e3f0ed00	\Users\kimsh\Downloads\container.png
0x9c84e3f0ee90	\Users\kimsh\Downloads\note.txt
0x9c84e3f0fe30	\Users\kimsh\Downloads\vault.hc
[REMOVED]
0x9c84e4adfb00	\Users\kimsh\AppData\Local\Microsoft\OneDrive\settings\Personal\downloads3.txt
[REMOVED]
```
- evtx files can be valuable
- *.hc* extension is used for *VeraCrypt* encrypted volumes
``` data
$ cat filescan.txt | grep -F -i -e "\\Users\\kimsh" | tr \\ " " | awk '{print $(NF)}' | sort | uniq
[REMOVED]
4BpQ1bD8vX1mXuJObN-gg9RqkyQ.br[1].js
4QHCYosFl0R2IKZXnGsA_ZG3bOk.br[1].js
4Z8eGLcJkgpNRfm3HRHk0Y0wjt0.br[1].js
[REMOVED]
AppCache133744364083753205.txt
[REMOVED]
History-journal
[REMOVED]
OneDrive.lnk
[REMOVED]
PowerShell.lnk
[REMOVED]
flag.jpg
[REMOVED]
hermes.dll
hint.gif
[REMOVED]
keys.json
[REMOVED]
```

### flag.jpg
``` data
$ cat filescan.txt | grep -F -i -e "\\Users\\kimsh" | grep -e "flag\.jpg"
0x9c84e3f21c20	\Users\kimsh\Documents\flag.jpg
$ vol -f chall.raw windows.dumpfiles --virtaddr 0x9c84e3f21c20
Volatility 3 Framework 2.27.0
Progress:  100.00		PDB scanning finished                        
Cache	FileObject	FileName	Result

DataSectionObject	0x9c84e3f21c20	flag.jpg	file.0x9c84e3f21c20.0x9c84db8c29b0.DataSectionObject.flag.jpg.dat
$ open file.0x9c84e3f21c20.0x9c84db8c29b0.DataSectionObject.flag.jpg.dat
```
- nothing interesting

### hint.gif
``` data
$ cat filescan.txt | grep -F -i -e "\\Users\\kimsh" | grep -e "hint"
0x9c84e3f21db0	\Users\kimsh\Documents\hint.gif
$ vol -f chall.raw windows.dumpfiles --virtaddr 0x9c84e3f21db0
Volatility 3 Framework 2.27.0
Progress:  100.00		PDB scanning finished
Cache	FileObject	FileName	Result

DataSectionObject	0x9c84e3f21db0	hint.gif	file.0x9c84e3f21db0.0x9c84db8c3270.DataSectionObject.hint.gif.dat
$ open file.0x9c84e3f21db0.0x9c84db8c3270.DataSectionObject.hint.gif.dat
```

### Downloads.lnk
``` data
$ vol -f chall.raw windows.dumpfiles --virtaddr 0x9c84e0653950 --virtaddr 0x9c84e0657640 --virtaddr 0x9c84e3f3ac20 --virtaddr 0x9c84e5286370
Volatility 3 Framework 2.27.0
Progress:  100.00		PDB scanning finished
Cache	FileObject	FileName	Result

DataSectionObject	0x9c84e5286370	Downloads.lnk	file.0x9c84e5286370.0x9c84db8d92f0.DataSectionObject.Downloads.lnk.dat
```

### History-journal
``` data
$ cat filescan.txt | grep -F -i -e "\\Users\\kimsh" | grep -e "journal"
0x9c84de9cdd60	\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\Network\Reporting and NEL-journal
0x9c84e0677d00	\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\Web Data-journal
0x9c84e0678e30	\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\Network\Cookies-journal
0x9c84e3f260e0	\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\first_party_sets.db-journal
0x9c84e52711f0	\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\History-journal
0x9c84e5293b10	\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\History-journal
$ vol -f chall.raw windows.dumpfiles --virtaddr 0x9c84e52711f0
Volatility 3 Framework 2.27.0
Progress:  100.00		PDB scanning finished
Cache	FileObject	FileName	Result

DataSectionObject	0x9c84e52711f0	History-journal	Error dumping file
SharedCacheMap	0x9c84e52711f0	History-journal	file.0x9c84e52711f0.0x9c84e04eb4e0.SharedCacheMap.History-journal.vacb
$ vol -f chall.raw windows.dumpfiles --virtaddr 0x9c84e5293b10
Volatility 3 Framework 2.27.0
Progress:  100.00		PDB scanning finished
Cache	FileObject	FileName	Result

DataSectionObject	0x9c84e5293b10	History-journal	Error dumping file
SharedCacheMap	0x9c84e5293b10	History-journal	file.0x9c84e5293b10.0x9c84e04eb4e0.SharedCacheMap.History-journal.vacb
$ strings file.0x9c84e52711f0.0x9c84e04eb4e0.SharedCacheMap.History-journal.vacb
typed_url_model_type_state
last_compatible_version
version
#	mmap_status]
3typed_url_model_type_state
4DGI4pW5a+qClpE+CGQY5g==2
0003BFFD7BF4AF89H
last_compatible_version16
version69
mmap_status-1]
$ strings file.0x9c84e5293b10.0x9c84e04eb4e0.SharedCacheMap.History-journal.vacb
typed_url_model_type_state
last_compatible_version
version
#	mmap_status]
3typed_url_model_type_state
4DGI4pW5a+qClpE+CGQY5g==2
0003BFFD7BF4AF89H
last_compatible_version16
version69
mmap_status-1]
```

### keys.json
``` data
$ cat filescan.txt | grep -F -i -e "\\Users\\kimsh" | grep -e "keys\.json"
0x9c84e3f16870	\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\TrustTokenKeyCommitments\2024.10.11.1\keys.json
(vol) sansforensics@siftworkstation: ~/Downloads
$ vol -f chall.raw windows.dumpfiles --virtaddr 0x9c84e3f16870
Volatility 3 Framework 2.27.0
Progress:  100.00		PDB scanning finished
Cache	FileObject	FileName	Result

DataSectionObject	0x9c84e3f16870	keys.json	file.0x9c84e3f16870.0x9c84db8c06b0.DataSectionObject.keys.json.dat
$ cat file.0x9c84e3f16870.0x9c84db8c06b0.DataSectionObject.keys.json.dat | xsel -ib
```

### vault.hc
``` data
$ cat mft.csv | grep "vault\.hc"
1,0x8e44b0,FILE,93057,1,File,Archive,FILE_NAME,2024-10-26 17:34:50.000000 UTC,2024-10-26 17:34:50.000000 UTC,2024-10-26 17:34:50.000000 UTC,2024-10-26 17:34:50.000000 UTC,vault.hc
1,0x4e240d70,FILE,93057,1,File,Archive,FILE_NAME,2024-10-26 17:34:50.000000 UTC,2024-10-26 17:34:50.000000 UTC,2024-10-26 17:34:50.000000 UTC,2024-10-26 17:34:50.000000 UTC,vault.hc
```
- same time as *note.txt*

### process 7128
``` handles
$ vol -f chall.raw -r csv windows.handles --pid 7128 > handles_7128.csv
$ csvtool col 4,5,6,7,8 handles_7128.csv | csvtool readable -
[REMOVED]
0xd8856700b080 0x444       Key                  0x20019       MACHINE\\SOFTWARE\\MICROSOFT\\INTERNET EXPLORER\\SECURITY
[REMOVED]
0xd885670095f0 0x454       Key                  0x20019       USER\\S-1-5-21-3188986842-248894163-1689600432-1001\\SOFTWARE\\MICROSOFT\\INTERNET EXPLORER\\MAIN
0xd8856700b7f0 0x458       Key                  0x20019       MACHINE\\SOFTWARE\\MICROSOFT\\INTERNET EXPLORER\\MAIN
0xd88567009e70 0x45c       Key                  0x20019       USER\\S-1-5-21-3188986842-248894163-1689600432-1001\\SOFTWARE\\MICROSOFT\\INTERNET EXPLORER\\SECURITY
[REMOVED]
0xd88567fd58e0 0x4b0       Key                  0x20019       MACHINE\\SOFTWARE\\POLICIES\\MICROSOFT\\WINDOWS\\CURRENTVERSION\\INTERNET SETTINGS
0xd88567fd65a0 0x4b4       Key                  0x20019       USER\\S-1-5-21-3188986842-248894163-1689600432-1001\\SOFTWARE\\POLICIES\\MICROSOFT\\WINDOWS\\CURRENTVERSION\\INTERNET SETTINGS
0xd88567fd66b0 0x4b8       Key                  0x20019       USER\\S-1-5-21-3188986842-248894163-1689600432-1001\\SOFTWARE\\MICROSOFT\\WINDOWS\\CURRENTVERSION\\INTERNET SETTINGS
0xd88567fd67c0 0x4bc       Key                  0x20019       MACHINE\\SOFTWARE\\MICROSOFT\\WINDOWS\\CURRENTVERSION\\INTERNET SETTINGS
0xd88567fd68d0 0x4c0       Key                  0x1           MACHINE\\SOFTWARE\\MICROSOFT\\INTERNET EXPLORER\\MAIN\\FEATURECONTROL
0xd88567fd5c10 0x4c4       Key                  0x1           USER\\S-1-5-21-3188986842-248894163-1689600432-1001\\SOFTWARE\\MICROSOFT\\INTERNET EXPLORER\\MAIN\\FEATURECONTROL
0xd88567fd69e0 0x4c8       Key                  0x20019       MACHINE\\SOFTWARE\\POLICIES
0xd88567fd6c00 0x4cc       Key                  0x20019       USER\\S-1-5-21-3188986842-248894163-1689600432-1001\\SOFTWARE\\POLICIES
0xd88567fd55b0 0x4d0       Key                  0x20019       USER\\S-1-5-21-3188986842-248894163-1689600432-1001\\SOFTWARE
0xd88567fd59f0 0x4d4       Key                  0x20019       MACHINE\\SOFTWARE
0xd88567fd5390 0x4d8       Key                  0x20019       MACHINE\\SOFTWARE\\WOW6432NODE
0xd88567fd4f50 0x4dc       Key                  0x2001f       USER\\S-1-5-21-3188986842-248894163-1689600432-1001\\SOFTWARE\\MICROSOFT\\WINDOWS\\CURRENTVERSION\\INTERNET SETTINGS\\ZONEMAP
0xd88567fd5280 0x4e0       Key                  0x20019       MACHINE\\SOFTWARE\\MICROSOFT\\WINDOWS\\CURRENTVERSION\\INTERNET SETTINGS\\ZONEMAP
[REMOVED]
```

- vad:
``` data
$ vol -f chall.raw -r csv windows.vadinfo --pid 7128 > vadinfo_7128.csv
$ csvtool col 4-12 vadinfo_7128.csv | csvtool readable - | grep -e "VPN" -e "PAGE_READWRITE"
Offset             Start VPN      End VPN        Tag  Protection             CommitCharge PrivateMemory Parent             File
0xffff9c84db94bf10 0x2a217e00000  0x2a217efffff  VadS PAGE_READWRITE         180          1             0xffff9c84dfb91a20 N/A
0xffff9c84db94c500 0xbb4c880000   0xbb4c8fffff   VadS PAGE_READWRITE         20           1             0xffff9c84dfb91ca0 N/A
0xffff9c84db94ccd0 0x17c70000     0x17c70fff     VadS PAGE_READWRITE         1            1             0xffff9c84db94c0f0 N/A
0xffff9c84db94cf00 0x17c60000     0x17c60fff     VadS PAGE_READWRITE         1            1             0xffff9c84db94ccd0 N/A
0xffff9c84db94c050 0xbb4c600000   0xbb4c7fffff   VadS PAGE_READWRITE         13           1             0xffff9c84db94c0f0 N/A
0xffff9c84db94c0a0 0xbb4c550000   0xbb4c5cffff   VadS PAGE_READWRITE         20           1             0xffff9c84db94c050 N/A
0xffff9c84db94c730 0xbb4c800000   0xbb4c87ffff   VadS PAGE_READWRITE         20           1             0xffff9c84db94c050 N/A
0xffff9c84e3f6c160 0x2a217c60000  0x2a217c6ffff  Vad  PAGE_READWRITE         0            0             0xffff9c84db94c500 N/A
0xffff9c84db94ce60 0xbb4c980000   0xbb4c9fffff   VadS PAGE_READWRITE         20           1             0xffff9c84e3f6c160 N/A
0xffff9c84db94d1d0 0xbb4c900000   0xbb4c97ffff   VadS PAGE_READWRITE         20           1             0xffff9c84db94ce60 N/A
0xffff9c84db94cb90 0xbb4ca00000   0xbb4ca7ffff   VadS PAGE_READWRITE         20           1             0xffff9c84db94ce60 N/A
0xffff9c84db94c5f0 0x2a217cc0000  0x2a217cc1fff  VadS PAGE_READWRITE         2            1             0xffff9c84e3f6d7e0 N/A
0xffff9c84db94c7d0 0x2a217dc0000  0x2a217dccfff  VadS PAGE_READWRITE         2            1             0xffff9c84e3f6ca20 N/A
0xffff9c84db94bfb0 0x2a217df0000  0x2a217dfffff  VadS PAGE_READWRITE         15           1             0xffff9c84dfb93780 N/A
0xffff9c84db94ca00 0x2a2197f0000  0x2a2197fffff  VadS PAGE_READWRITE         10           1             0xffff9c84db94bf10 N/A
0xffff9c84db94caa0 0x2a219750000  0x2a219750fff  VadS PAGE_READWRITE         1            1             0xffff9c84db94ca00 N/A
0xffff9c84db94bf60 0x2a2196a0000  0x2a2196acfff  VadS PAGE_READWRITE         2            1             0xffff9c84dfb958a0 N/A
0xffff9c84db94cd70 0x2a219720000  0x2a219720fff  VadS PAGE_READWRITE         1            1             0xffff9c84dfb958a0 N/A
0xffff9c84db94cb40 0x2a219790000  0x2a21979cfff  VadS PAGE_READWRITE         1            1             0xffff9c84db94caa0 N/A
0xffff9c84db94d180 0x2a219770000  0x2a219770fff  VadS PAGE_READWRITE         1            1             0xffff9c84db94cb40 N/A
0xffff9c84dfb9ce20 0x2a219760000  0x2a219760fff  Vad  PAGE_READWRITE         0            0             0xffff9c84db94d180 N/A
0xffff9c84db947960 0x2a21cd10000  0x2a21cd10fff  VadS PAGE_READWRITE         1            1             0xffff9c84db94ca00 N/A
0xffff9c84db359190 0x2a21c000000  0x2a21c3e4fff  Vad  PAGE_READWRITE         0            0             0xffff9c84db947960 N/A
0xffff9c84db94ceb0 0x2a21bf00000  0x2a21bffffff  VadS PAGE_READWRITE         1            1             0xffff9c84e3f730a0 N/A
0xffff9c84db94d770 0x2a21cbf0000  0x2a21cceffff  VadS PAGE_READWRITE         5            1             0xffff9c84db359190 N/A
0xffff9c84dee1ead0 0x2a21ccf0000  0x2a21ccf0fff  Vad  PAGE_READWRITE         0            0             0xffff9c84db94d770 N/A
0xffff9c84db359f50 0x2a21ce20000  0x2a21ce20fff  Vad  PAGE_READWRITE         0            0             0xffff9c84dfb91f20 N/A
```

- data dump:
``` data
$ vol -f chall.raw -o PID_7128 windows.vadinfo --pid 7128 --dump
$ csvtool col 4-12 vadinfo_7128.csv | csvtool readable - | grep -e "PAGE_READWRITE" | awk '{print "pid.7128.vad."$2"-"$3".dmp"}' | xargs -I {} strings {} >> total_strings
strings: 'pid.7128.vad.0x7df4d0970000-0x7df5d098ffff.dmp': No such file
$ cat total_strings 
[REMOVED]
kern
mark
mkmk
dist
o@)l
B	C	SimSun-ExtB
N	M	`	r	a	b
e	f	g	h
%	Y
?	&	W
/C:\
Users
kimsh
Desktop
/C:\
Users
kimsh
Music
/C:\
Users
kimsh
Videos
[REMOVED]
C:\Users\kimsh\AppData\Local\Microsoft\OneDrive\OneDrive.exe,1
[REMOVED]
$ csvtool col 4-12 vadinfo_7128.csv | csvtool readable - | grep -e "PAGE_READWRITE" | awk '{print "pid.7128.vad."$2"-"$3".dmp"}' | xargs -I {} strings -el {} >> long_total_strings
strings: 'pid.7128.vad.0x7df4d0970000-0x7df5d098ffff.dmp': No such file
$ cat long_total_strings
[REMOVED]
Cannot open the %% file.
Make sure a disk is in the drive you specified.
Cannot find the %% file.
Do you want to create a new file?
The text in the %% file has changed.
Do you want to save the changes?
Untitled
Not enough memory available to complete this operation. Quit one or more applications to increase available memory, and then try again.
Cannot find "%%"
%1%2 - Notepad
The %% file is too large for Notepad.
Use another editor to edit the file.
Notepad
Failed to initialize file dialogs. Change the file name and try again.
Failed to initialize print dialogs. Make sure that your printer is connected properly and use Control Panel to verify that the printer is configured properly.
Cannot print the %% file. Be sure that your printer is connected properly and use Control Panel to verify that the printer is configured properly.
Not a valid file name.
Cannot create the %% file.
Make sure that the path and file name are correct.
Cannot carry out the Word Wrap command because there is too much text in the file.
notepad.hlp
Text Documents (*.txt)
All Files
Open
Save As
You cannot shut down or log off Windows because
the Save As dialog box in Notepad is open. Switch to
Notepad, close this dialog box, and then try shutting
down or logging off Windows again.
Cannot access your printer.
Be sure that your printer is connected properly and use Control Panel to verify that the printer is configured properly.
You do not have permission to open this file.  See the owner of the file or an administrator to obtain permission.
 This file contains characters in Unicode format which will be lost if you save this file as an ANSI encoded text file. To keep the Unicode information, click Cancel below and then select one of the Unicode options from the Encoding drop down list. Continue?
Common Dialog error (0x%04x)
Page too small to print one line.
Try printing using smaller font.
Notepad - Goto Line
The line number is beyond the total number of lines
Auto-Detect
[REMOVED]
DESKTOP-9BHKMOM
10.0.2.15
[REMOVED]
ISUN0407.EXE
\??\C:\Users\kimsh
open
Start menu cache
C:\Users\kimsh
GUESTMODEMSG.EXE
UNINSTAL.EXE
[REMOVED]
WUAPP.EXE
[REMOVED]
JAVAW.EXE
Startup
*.exe
ST5UNST.EXE
[REMOVED]
ADOR.EXE
ALAR.EXE
DFSVC.EXE
EAUNINSTALL.EXE
MODEMSG.EXE
HPZSCR01.EXE
HPZSCR40.EX
STALL.EXE
ISUN0407.EXE
ISUNINST.EXE
002.EXE
LNKSTUB.EXE
MSIEXEC.EXE
MSOO
SETUP.EXE
ST5UNST.EXE
UNINS000.EX
INS001.EXE
UNINS002.EXE
UNINST.EXE
TAL.EXE
UNINSTALL.EXE
UNINSTALLER.EX
[REMOVED]
```

- finding heap:
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
- found 4 heaps:
	- 0x2a217e00000
	- 0x2a217c60000
	- 0x2a217df0000
	- 0x2a2197f0000

- extracting heap strings:
``` data
$ ll | grep -e 0x2a217e00000 -e 0x2a217c60000 -e 0x2a217df0000 -e 0x2a2197f0000 | awk '{print $NF}' | xargs -I {} strings -a {} > heap_strings
$ ll | grep -e 0x2a217e00000 -e 0x2a217c60000 -e 0x2a217df0000 -e 0x2a2197f0000 | awk '{print $NF}' | xargs -I {} strings  -el -a {} >> heap_strings
$ cat heap_strings | grep -F -v -i -e "\\" -e  " " -e "\;" -e "t0Pq" -e "system" -e "?pq" -e ".exe" -e "ext-ms-win-core" -e "Windows" -e ".dll" -e "S-1-15-3" -e "-"
```

- potential passwords:
``` data
?{8s
%)yAI
=1W)
J]/i2
MP{9
MqXM
n30'
gWk&c[?
h8H$
lUz9
$/A*
bfiJi
C2#l
FtNM
(GX&
$^+8
n30'
gWk&c[?
|:0Po
[B9J
$zO*/;
h`%}
S/3L
GFS]A=
QF)P,
52C64B7E
Y24k8UPs
fFpPtTdDcCrRlL
```

- password cracking
``` data
$ hashcat -a 0 -m 13711 file.0x9c84e3f0fe30.0x9c84dbc4e590.DataSectionObject.vault.hc.dat passwords.txt
$ hashcat -a 0 -m 13712 -o cracked.txt file.0x9c84e3f0fe30.0x9c84dbc4e590.DataSectionObject.vault.hc.dat passwords.txt
$ hashcat -a 0 -m 13713 -o cracked.txt file.0x9c84e3f0fe30.0x9c84dbc4e590.DataSectionObject.vault.hc.dat passwords.txt
$ hashcat -a 0 -m 13721 -o cracked.txt file.0x9c84e3f0fe30.0x9c84dbc4e590.DataSectionObject.vault.hc.dat passwords.txt
$ hashcat -a 0 -m 13731 -o cracked.txt file.0x9c84e3f0fe30.0x9c84dbc4e590.DataSectionObject.vault.hc.dat passwords.txt
```
- nothing found

- viewing heap:
``` data
$ volshell -w -f chall.raw --pid 7128
[REMOVED]
(layer_name_Process7128_1) >>> dt("_HEAP", 0x2a217e00000)
[REMOVED]
   0x38 :   NumberOfPages                      symbol_table_name1!unsigned long                   255
   0x40 :   FirstEntry                         *symbol_table_name1!_HEAP_ENTRY                    0x2a217e00740
   0x48 :   LastValidEntry                     *symbol_table_name1!_HEAP_ENTRY                    0x2a217eff000 (unreadable pointer)
   0x50 :   NumberOfUnCommittedPages           symbol_table_name1!unsigned long                   75
   0x54 :   NumberOfUnCommittedRanges          symbol_table_name1!unsigned long                   1
[REMOVED]
(layer_name_Process7128_1) >>> dt("_HEAP_ENTRY", 0x2a217e00740)
[REMOVED]
  0x8 :   Size                         symbol_table_name1!unsigned short           30439
[REMOVED]
  0xa :   Flags                        symbol_table_name1!unsigned char            150
[REMOVED]
```

### PID 924
``` data
$ vol -f chall.raw -o PID_924 -r csv windows.vadinfo --pid 924 --dump > PID_924/vadinfo_924.csv
$ csvtool col 4-12 vadinfo_924.csv | csvtool readable - | grep -e "PAGE_READWRITE" | awk '{print "pid.924.vad."$2"-"$3".dmp"}' |     xargs -I {} strings {} >> total_strings
strings: 'pid.924.vad.0x7df4f4020000-0x7df5f403ffff.dmp': No such file
$ csvtool col 4-12 vadinfo_924.csv | csvtool readable - | grep -e "PAGE_READWRITE" | awk '{print "pid.924.vad."$2"-"$3".dmp"}' | xargs -I {} strings -el {} >> long_total_strings
strings: 'pid.924.vad.0x7df4f4020000-0x7df5f403ffff.dmp': No such file
$ cat long_total_strings
[REMOVED]
```

- finding heap:
``` data
$ volshell -f chall.raw -w --pid 924
[REMOVED]
(layer_name_Process924_1) >>> dt("_EPROCESS", 0x9c84de30f080)
[REMOVED]
  0x550 :   Peb                                    *symbol_table_name1!_PEB                                   0xc1186f4000
[REMOVED]
(layer_name_Process924_1) >>> dt("_PEB", 0xc1186f4000)
[REMOVED]
   0xe8 :   NumberOfHeaps                            symbol_table_name1!unsigned long                     4
   0xec :   MaximumNumberOfHeaps                     symbol_table_name1!unsigned long                     16
   0xf0 :   ProcessHeaps                             **symbol_table_name1!void                            0x7ffea9a3ad40
[REMOVED]
(layer_name_Process924_1) >>> dd(0x7ffea9a3ad40, count=32)
0x7ffea9a3ad40    04a90000 0000026d 04860000 0000026d    .... ...m .... ...m
0x7ffea9a3ad50    063f0000 0000026d 063b0000 0000026d    .?.. ...m .;.. ...m
[REMOVED]
```

- viewing heap data:
``` data
$ ll | grep -e 0x26d04a90000 -e 0x26d04860000 -e 0x26d063f0000 -e 0x26d063b0000
-rw-------  1 sansforensics sansforensics    65536 Apr 19 01:54 pid.924.vad.0x26d04860000-0x26d0486ffff.dmp
-rw-------  1 sansforensics sansforensics  1048576 Apr 19 01:54 pid.924.vad.0x26d04a90000-0x26d04b8ffff.dmp
-rw-------  1 sansforensics sansforensics    65536 Apr 19 01:54 pid.924.vad.0x26d063b0000-0x26d063bffff.dmp
-rw-------  1 sansforensics sansforensics    65536 Apr 19 01:54 pid.924.vad.0x26d063f0000-0x26d063fffff.dmp
$ ll | grep -e 0x26d04a90000 -e 0x26d04860000 -e 0x26d063f0000 -e 0x26d063b0000 | awk '{print $NF}' | xargs -I {} strings -el {} > heap_wide_strings.txt
$ cat heap_wide_strings.txt | grep -F -v -e "\\" -e  " "
```

- potential passwords found:
``` data
Y454wE3kh7
s_65
Wind
eppi
ROCE
APPDA
veCo
Y24k8UPs
Tahoma
Ebrima
Gadugi
Leelawadee
OLEB5266094BAFF215EBFF4BBDF1071
07eb
4da9
Y454wE3kh7
%)0:@FLRX^djpv|
&*1;AGMSY_ekqw}
!,37=CIOU[agmsy
$/69?EKQW]ciou{
(Rbr
>4nrvz~
edWi
```
- password cracking
``` data
$ hashcat -a 0 -m 13711 -o cracked.txt file.0x9c84e3f0fe30.0x9c84dbc4e590.DataSectionObject.vault.hc.dat passwords_924.txt
$ hashcat -a 0 -m 13712 -o cracked.txt file.0x9c84e3f0fe30.0x9c84dbc4e590.DataSectionObject.vault.hc.dat passwords_924.txt
$ hashcat -a 0 -m 13713 -o cracked.txt file.0x9c84e3f0fe30.0x9c84dbc4e590.DataSectionObject.vault.hc.dat passwords_924.txt
$ hashcat -a 0 -m 13721 -o cracked.txt file.0x9c84e3f0fe30.0x9c84dbc4e590.DataSectionObject.vault.hc.dat passwords_924.txt
[REMOVED]
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
[REMOVED]
$ cat cracked.txt
file.0x9c84e3f0fe30.0x9c84dbc4e590.DataSectionObject.vault.hc.dat:Y454wE3kh7
```
- password found: Y454wE3kh7

### PID 3484
``` data
$ vol -f chall.raw -o PID_3484 -r csv windows.vadinfo --pid 3484 --dump > PID_3484/vadinfo_3484.csv
$ csvtool col 4-12 vadinfo_3484.csv | csvtool readable - | grep -e "PAGE_READWRITE" | awk '{print "pid.3484.vad."$2"-"$3".dmp"}' | xargs -I {} strings {} >> total_strings
strings: 'pid.3484.vad.0x7df468c60000-0x7df568c7ffff.dmp': No such file
$ csvtool col 4-12 vadinfo_3484.csv | csvtool readable - | grep -e "PAGE_READWRITE" | awk '{print "pid.3484.vad."$2"-"$3".dmp"}' | xargs -I {} strings -el {} >> long_total_strings
$ cat long_total_strings
[REMOVED]
Cannot open the %% file.
Make sure a disk is in the drive you specified.
Cannot find the %% file.
Do you want to create a new file?
The text in the %% file has changed.
Do you want to save the changes?
Untitled
Not enough memory available to complete this operation. Quit one or more applications to increase available memory, and then try again.
Cannot find "%%"
%1%2 - Notepad
The %% file is too large for Notepad.
Use another editor to edit the file.
Notepad
Failed to initialize file dialogs. Change the file name and try again.
Failed to initialize print dialogs. Make sure that your printer is connected properly and use Control Panel to verify that the printer is configured properly.
Cannot print the %% file. Be sure that your printer is connected properly and use Control Panel to verify that the printer is configured properly.
Not a valid file name.
Cannot create the %% file.
Make sure that the path and file name are correct.
Cannot carry out the Word Wrap command because there is too much text in the file.
notepad.hlp
Text Documents (*.txt)
All Files
Open
Save As
You cannot shut down or log off Windows because
the Save As dialog box in Notepad is open. Switch to
Notepad, close this dialog box, and then try shutting
down or logging off Windows again.
Cannot access your printer.
Be sure that your printer is connected properly and use Control Panel to verify that the printer is configured properly.
You do not have permission to open this file.  See the owner of the file or an administrator to obtain permission.
 This file contains characters in Unicode format which will be lost if you save this file as an ANSI encoded text file. To keep the Unicode information, click Cancel below and then select one of the Unicode options from the Encoding drop down list. Continue?
Common Dialog error (0x%04x)
Page too small to print one line.
Try printing using smaller font.
Notepad - Goto Line
The line number is beyond the total number of lines
[REMOVED]
Page &p
 Ln %d, Col %d
 Compressed,
 Encrypted,
 Hidden,
 Offline,
 ReadOnly,
 System,
 File
fFpPtTdDcCrRlL
&Encoding:
Notepad was running in a transaction which has completed.
Would you like to save the %% file non-transactionally?
Text Editor
Status Bar
We can
t open this file
Either your organization doesn
t allow it, or there
s a problem with the file
s encryption.
 Windows (CRLF)
 Unix (LF)
 Macintosh (CR)
 Found next from the bottom
 Found next from the top
 %d%%
[REMOVED]
gotiate
NegoExtender
Kerberos
NTLM
TSSSP
pku2u
Schannel
[REMOVED]
DESKTOP-9BHKMOM
10.0.2.15
[REMOVED]
C:\Users\kimsh\AppData\Roaming\Microsoft\Windows\Recent
C:\Users\kimsh\AppData\Roaming\Microsoft\Windows\Recent
TabGroupingPreference.LaunchingWindow
C:\Users\kimsh\AppData\Local\Microsoft\Windows\INetCache
Software\Microsoft\Windows NT\CurrentVersion\ProfileList
C:\Users\kimsh\AppData\Local\Microsoft\Windows\INetCookies
[REMOVED]
```

### cmdscan
``` data
$ vol -f chall.raw -r csv windows.cmdscan > cmdscan.csv
$ csvtool col 3,4,5,6,7 cmdscan.csv | csvtool readable -
Process     ConsoleInfo   Property                         Address       Data
conhost.exe 0x1c642ad7860 _COMMAND_HISTORY                 0x1c642ad7860 None
conhost.exe 0x1c642ad7860 _COMMAND_HISTORY.Application     0x1c642ad7890 powershell.exe
conhost.exe 0x1c642ad7860 _COMMAND_HISTORY.ProcessHandle   0x1c640b85b30 0x130
conhost.exe 0x1c642ad7860 _COMMAND_HISTORY.CommandCount    N/A           0
conhost.exe 0x1c642ad7860 _COMMAND_HISTORY.LastDisplayed   0x1c642ad78bc -1
conhost.exe 0x1c642ad7860 _COMMAND_HISTORY.CommandCountMax 0x1c642ad7888 50
conhost.exe 0x1c642ad7860 _COMMAND_HISTORY.CommandBucket   0x1c642ad7870 
conhost.exe 0x1c1a6ecac50 _COMMAND_HISTORY                 0x1c1a6ecac50 None
conhost.exe 0x1c1a6ecac50 _COMMAND_HISTORY.Application     0x1c1a6ecac80 DumpIt.exe
conhost.exe 0x1c1a6ecac50 _COMMAND_HISTORY.ProcessHandle   0x1c1a4de5380 0x124
conhost.exe 0x1c1a6ecac50 _COMMAND_HISTORY.CommandCount    N/A           0
conhost.exe 0x1c1a6ecac50 _COMMAND_HISTORY.LastDisplayed   0x1c1a6ecacac -1
conhost.exe 0x1c1a6ecac50 _COMMAND_HISTORY.CommandCountMax 0x1c1a6ecac78 50
conhost.exe 0x1c1a6ecac50 _COMMAND_HISTORY.CommandBucket   0x1c1a6ecac60
```

### netstat
``` data
$ vol -f chall.raw -r pretty windows.netstat
Volatility 3 Framework 2.27.0
Formatting...0.00		PDB scanning finished
  |         Offset | Proto |                 LocalAddr | LocalPort |    ForeignAddr | ForeignPort |       State |  PID |          Owner |                        Created
* | 0x9c84def11b20 | TCPv4 |                 10.0.2.15 |     49708 | 23.215.215.227 |         443 | ESTABLISHED | 4528 |  SearchApp.exe | 2024-10-26 17:34:04.000000 UTC
* | 0x9c84d9890b00 | TCPv4 |                 10.0.2.15 |     49760 |   40.99.34.226 |         443 | ESTABLISHED | 4528 |  SearchApp.exe | 2024-10-26 17:37:06.000000 UTC
* | 0x9c84e56eaa20 | TCPv4 |                 10.0.2.15 |     49723 |    4.188.4.136 |         443 | ESTABLISHED |  636 | smartscreen.ex | 2024-10-26 17:34:32.000000 UTC
* | 0x9c84dbde8a20 | TCPv4 |                 10.0.2.15 |     49765 | 204.79.197.222 |         443 | ESTABLISHED | 4528 |  SearchApp.exe | 2024-10-26 17:37:12.000000 UTC
* | 0x9c84dfab8010 | TCPv4 |                 10.0.2.15 |     49701 | 20.198.119.143 |         443 | ESTABLISHED | 2912 |    svchost.exe | 2024-10-26 17:32:22.000000 UTC
* | 0x9c84e0d17a20 | TCPv4 |                 10.0.2.15 |     49762 | 13.107.246.254 |         443 | ESTABLISHED | 4528 |  SearchApp.exe | 2024-10-26 17:37:09.000000 UTC
* | 0x9c84df1d8590 | TCPv4 |                 10.0.2.15 |     49758 | 204.79.197.239 |         443 | ESTABLISHED | 6872 |     msedge.exe | 2024-10-26 17:36:11.000000 UTC
* | 0x9c84dbfeab00 | TCPv4 |                 10.0.2.15 |     49764 |   20.217.24.74 |         443 | ESTABLISHED | 4528 |  SearchApp.exe | 2024-10-26 17:37:10.000000 UTC
* | 0x9c84e03b0b50 | TCPv4 |                 10.0.2.15 |     49763 | 13.107.246.254 |         443 | ESTABLISHED | 4528 |  SearchApp.exe | 2024-10-26 17:37:10.000000 UTC
* | 0x9c84dbe66a60 | TCPv4 |                 10.0.2.15 |     49744 | 20.198.119.143 |         443 | ESTABLISHED | 6580 |   OneDrive.exe | 2024-10-26 17:35:10.000000 UTC
* | 0x9c84df9d4010 | TCPv4 |                 10.0.2.15 |     49703 |     23.36.7.22 |         443 | ESTABLISHED | 4936 |    svchost.exe | 2024-10-26 17:33:46.000000 UTC
* | 0x9c84e0272570 | TCPv4 |                 10.0.2.15 |     49761 |  52.138.229.66 |         443 | ESTABLISHED | 4528 |  SearchApp.exe | 2024-10-26 17:37:06.000000 UTC
* | 0x9c84dfb633d0 | TCPv4 |                 10.0.2.15 |     49752 | 23.200.238.233 |         443 | ESTABLISHED | 5016 |   explorer.exe | 2024-10-26 17:35:28.000000 UTC
[REMOVED]
```

### device tree
``` data
$ vol -r csv -f chall.raw windows.devicetree.DeviceTree > devicetree.csv
$ csvtool readable devicetree.csv 
TreeDepth Offset         Type DriverName            DeviceName                                         DriverNameOfAttDevice     DeviceType
[REMOVED]
0         0x9c84d98b6870 DRV  Ntfs                  N/A                                                N/A                       N/A
1         0x9c84d98b6870 DEV  Ntfs                  -                                                  N/A                       FILE_DEVICE_DISK_FILE_SYSTEM
2         0x9c84d98b6870 ATT  Ntfs                  -                                                  \\FileSystem\\FltMgr      FILE_DEVICE_DISK_FILE_SYSTEM
[REMOVED]
```

### PID 5016 explorer.exe
- strings:
``` data
$ vol -f chall.raw -o PID_5016/ -r csv windows.vadinfo --pid 5016 --dump > PID_5016/vadinfo_5016.csv
$ csvtool col 4-12 vadinfo_5016.csv | csvtool readable - | grep -e "PAGE_READWRITE" | awk '{print "pid.5016.vad."$2"-"$3".dmp"}' | xargs -I {} strings {} >> total_strings
$ csvtool col 4-12 vadinfo_5016.csv | csvtool readable - | grep -e "PAGE_READWRITE" | awk '{print "pid.5016.vad."$2"-"$3".dmp"}' | xargs -I {} strings -el {} >> total_strings
$ cat total_strings
HXC138~1.PNG
[REMOVED]
USERPI~1
CACHED~1.JPG
[REMOVED]
www.digicert.com1 0
DigiCert Global Root G3
[REMOVED]
 200 OK
Content-Type: image/png
Access-Control-Allow-Origin: *
Access-Control-Allow-Headers: *
Access-Control-Allow-Methods: GET, POST, OPTIONS
Timing-Allow-Origin: *
Report-To: {"group":"network-errors","max_age":604800,"endpoints":[{"url":"https://aefd.nelreports.net/api/report?cat=bingth&ndcParam=QWthbWFp"}]}
NEL: {"report_to":"network-errors","max_age":604800,"success_fraction":0.001,"failure_fraction":1.0}
Content-Length: 8291
Alt-Svc: h3=":443"; ma=93600
[REMOVED]

```
