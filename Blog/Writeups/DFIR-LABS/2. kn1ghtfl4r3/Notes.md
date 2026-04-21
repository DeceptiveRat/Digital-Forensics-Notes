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
$ keepass2john Keylist.kdbx > keylist.hash
$ python3 createlist.py 
$ head wordlist.txt 
a0rp_RX_24312
a0rp_Rð_24312
aÚrp_RX_24312
aÚrp_Rð_24312
aarp_RX_24312
aarp_Rð_24312
aUrp_RX_24312
aUrp_Rð_24312
a)rp_RX_24312
a)rp_Rð_24312
$ cut -d ":" -f 2- Keylist.hash > Keylist_clean.hash
$ hashcat -a 0 -o keepass_cracked.txt -m 13400 Keylist_clean.hash wordlist.txt
Status...........: Cracked
$ cat keepass_cracked.txt
[REMOVED]c0rp_RX_24312
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

- potential passwords found:
``` data
Y454wE3kh7
s_65
52C64B7E
Y24k8UPs
fFpPtTdDcCrRlL
bb6ea983fc583c3d9d71280b69d603640f2ca6c42b888e894ef5636292eca27e
BHKMOM
lasses5
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

### vault.hc
- mounted directory:
``` data
$ ll
total 8764
drwx------ 1 sansforensics sansforensics       0 Oct 26  2024 '$RECYCLE.BIN'/
drwx------ 1 sansforensics sansforensics    4096 Oct 26  2024  ./
drwxr-xr-x 4 root          root             4096 Apr 20 00:10  ../
drwx------ 1 sansforensics sansforensics       0 Oct 26  2024 'System Volume Information'/
-rwx------ 1 sansforensics sansforensics 8963778 Oct 22  2024  project.zip*
```

- unzip:
``` data
$ unzip project.zip
Archive:  project.zip
   creating: project/
[project.zip] project/Confidential password:
```
- requires password

- attempt password list:
``` data
$ fcrackzip -D -p potential_passwords.txt -v project.zip
'project/' is not encrypted, skipping
found file 'project/Confidential', (size cp/uc 320812/320800, flags 1, chk 1ebc)
found file 'project/enc.exe', (size cp/uc 8639848/8823677, flags 1, chk 92e1)
found file 'project/Keylist.kdbx', (size cp/uc   2522/  2510, flags 1, chk dd0b)
```

- attempt bruteforce:
``` data
$ fcrackzip -b -u -v project.zip
'project/' is not encrypted, skipping
found file 'project/Confidential', (size cp/uc 320812/320800, flags 1, chk 1ebc)
found file 'project/enc.exe', (size cp/uc 8639848/8823677, flags 1, chk 92e1)
found file 'project/Keylist.kdbx', (size cp/uc   2522/  2510, flags 1, chk dd0b)
^Cecking pw axpLN~
```

- use password found from powershell envar:
``` data
$ unzip project.zip
Archive:  project.zip
   creating: project/
[project.zip] project/Confidential password:
 extracting: project/Confidential
  inflating: project/enc.exe
 extracting: project/Keylist.kdbx
```

### files that seem safe
- MS edge
``` data
$ cat filescan.txt | grep -F "\\Users\\kimsh\\AppData\\Local\\Microsoft\\Edge\\User Data\\" | cut -f 2- | sort | uniq
[REMOVED]
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Autofill\4.0.1.5\autofill_bypass_cache_forms.json
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Autofill\4.0.1.5\edge_autofill_field_data.json
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Autofill\4.0.1.5\edge_autofill_global_block_list.json
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Autofill\4.0.1.5\manifest.fingerprint
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Autofill\4.0.1.5\manifest.json
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Autofill\4.0.1.5\v1FieldTypes.json
[REMOVED]
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\Cache\Cache_Data\data_0
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\Cache\Cache_Data\data_1
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\Cache\Cache_Data\data_2
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\Cache\Cache_Data\data_3
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\Cache\Cache_Data\index
[REMOVED]
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\History
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\History-journal
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\Local Storage\leveldb\000003.log
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\Local Storage\leveldb\CURRENT
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\Local Storage\leveldb\LOCK
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\Local Storage\leveldb\LOG
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\Local Storage\leveldb\MANIFEST-000001
[REMOVED]
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\Site Characteristics Database\000003.log
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\Site Characteristics Database\CURRENT
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\Site Characteristics Database\LOCK
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\Site Characteristics Database\LOG
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\Site Characteristics Database\MANIFEST-000001
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\Sync Data\LevelDB\000003.log
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\Sync Data\LevelDB\CURRENT
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\Sync Data\LevelDB\LOCK
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\Sync Data\LevelDB\LOG
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\Sync Data\LevelDB\MANIFEST-000001
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\Sync Data\Logs\sync_diagnostic.log
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\Top Sites
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\Visited Links
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\Web Data
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Default\Web Data-journal
[REMOVED]
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Edge Wallet\128.18169.18147.7\json\wallet\wallet-checkout-eligible-sites.json
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Edge Wallet\128.18169.18147.7\json\wallet\wallet-notification-config.json
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Edge Wallet\128.18169.18147.7\json\wallet\wallet-stable.json
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Edge Wallet\128.18169.18147.7\json\wallet\wallet-tokenization-config.json
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Edge Wallet\128.18169.18147.7\manifest.fingerprint
\Users\kimsh\AppData\Local\Microsoft\Edge\User Data\Edge Wallet\128.18169.18147.7\manifest.json
[REMOVED]
$ cat filescan.txt | grep -f edge_files | cut -f 1 | xargs -I {} vol -f chall.raw -o edge_userdata/ windows.dumpfiles --virtaddr {}
```

- filescan.txt:
``` data
$ cat filescan.txt | grep -v -F -i -e ".sys" -e ".dll" -e ".exe" -e ".ttf"| cut -f 2 | sort | uniq | less
[REMOVED]
\ProgramData\Microsoft\Network\Downloader\edb.chk
\ProgramData\Microsoft\Network\Downloader\edb.log
\ProgramData\Microsoft\Network\Downloader\edb00001.log
\ProgramData\Microsoft\Network\Downloader\qmgr.db
\ProgramData\Microsoft\Network\Downloader\qmgr.jfm
[REMOVED]
\Users\Public\Desktop\VeraCrypt.lnk
[REMOVED]
\Users\kimsh\AppData\Local\Microsoft\Credentials
\Users\kimsh\AppData\Local\Microsoft\Credentials\00380DB8A62A9469B936B7474D92F74D
\Users\kimsh\AppData\Local\Microsoft\Credentials\DFBE70A7E5CC19A398EBF1B96859CE5D
\Users\kimsh\AppData\Local\Microsoft\Credentials\E05DBE15D38053457F3523A375594044
[REMOVED]
\Users\kimsh\AppData\Roaming\KeePass\KeePass.config.xml
\Users\kimsh\AppData\Roaming\Microsoft\Credentials
\Users\kimsh\AppData\Roaming\Microsoft\Crypto\Keys\de7cf8a7901d2ad13e5c67c29e5d1662_2eb6cbae-3012-4ae0-bc82-1556eec145f2
[REMOVED]
\Users\kimsh\AppData\Roaming\Microsoft\Windows\Printer Shortcuts
[REMOVED]
\Users\kimsh\AppData\Roaming\Microsoft\Windows\SendTo\Bluetooth File Transfer.LNK
[REMOVED]
\Users\kimsh\Desktop\final.png
[REMOVED]
\Users\kimsh\OneDrive\Personal Vault.lnk
[REMOVED]
```

### lsadump
``` data
$ vol -f chall.raw windows.registry.lsadump
Volatility 3 Framework 2.27.0
Progress:  100.00		PDB scanning finished
Key	Secret	Hex

DefaultPassword
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ................
2b ac 9e d1 b1 45 e0 eb f9 37 a6 10 e3 94 fc c7 +....E...7......	00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 2b ac 9e d1 b1 45 e0 eb f9 37 a6 10 e3 94 fc c7
DPAPI_SYSTEM
2c 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ,...............
01 00 00 00 89 6c 21 ae 57 0c 7e 67 fc be ed d5 .....l!.W.~g....
bb 31 fc 5c e4 69 0f 7f 29 45 1d 89 72 c5 5b 6d .1.\.i..)E..r.[m
0e 76 d5 17 98 36 a8 2e bb 7b d7 6c 00 00 00 00 .v...6...{.l....	2c 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 01 00 00 00 89 6c 21 ae 57 0c 7e 67 fc be ed d5 bb 31 fc 5c e4 69 0f 7f 29 45 1d 89 72 c5 5b 6d 0e 76 d5 17 98 36 a8 2e bb 7b d7 6c 00 00 00 00
```

### envars
``` data
$ vol -f chall.raw windows.envars > envars.txt
$ cat envars.txt | grep -F -i -e "notepad.exe" -e "powershell.exe" -e "keepass.exe"
[REMOVED]
1116	powershell.exe	0x24112d11c00	zippa4s5wo0rD	MP0Z\CRJws&4e\c':=3k
[REMOVED]
```

- search for TMP tinkering
``` data
$ cat envars.txt | grep -e "TMP" | cut -f 4,5 | sort | uniq
CHROME_CRASHPAD_PIPE_NAME	\\.\pipe\crashpad_6540_FTMPKNJNXAPMMBDZ
TMP	%TEMP%\Packages\microsoft.windows.fontdrvhost\AC\Temp
TMP	C:\ProgramData\Microsoft\Search\Data\Temp\usgthrsvc
TMP	C:\Users\kimsh\AppData\Local\Packages\microsoft.windows.search_cw5n1h2txyewy\AC\Temp
TMP	C:\Users\kimsh\AppData\Local\Packages\microsoft.windows.startmenuexperiencehost_cw5n1h2txyewy\AC\Temp
TMP	C:\Users\kimsh\AppData\Local\Temp
TMP	C:\Windows\SERVIC~1\LOCALS~1\AppData\Local\Temp
TMP	C:\Windows\SERVIC~1\NETWOR~1\AppData\Local\Temp
TMP	C:\Windows\TEMP
```

### project directory
- Confidential:
``` data
$ xxd -l 4096 Confidential
00000000: 8223 7839 5228 e77f 4707 a629 0755 b314  .#x9R(..G..).U..
00000010: d807 5d35 5b5a 98a1 9ce0 86ec c738 79ef  ..]5[Z.......8y.
00000020: 05d8 a293 6c09 4f52 4bc6 7472 79ca b04f  ....l.ORK.try..O
00000030: ceb7 090a 8707 0e68 4a4a f96d 32ab 4c64  .......hJJ.m2.Ld
00000040: 545a 9d03 0427 b697 2a87 73ca 060b c582  TZ...'..*.s.....
[REMOVED]
```


### *base_library.zip*
``` data
$ cat strings.txt | grep -F -i -e "base_library.zip"
152974986 xbase_library.zipx
753301277 base_library.zip
152975003 xbase_library.zipxbitcoin.bmpxlock.bmpxlock.ico
$ cat strings.txt | grep -F -C 50 -i -e "base_library.zip"
[REMOVED]
152972492 TrojanDownloader:PowerShell/Bynoco.PSX!MTB
152972571 SCPT:TrojanDownloader:PowerShell/Bynoco.PSX1!MTB
152972620 SCPT:TrojanDownloader:PowerShell/Bynoco.PSX2!MTB
152972669 #ATTR_00002e12
152972684 SCPT:TrojanDownloader:PowerShell/Bynoco.PSX3!MTB
152972733 #ATTR_00002e13
152972748 SCPT:TrojanDownloader:PowerShell/Bynoco.PSX4!MTB
152972797 #ATTR_00002e14
152972812 SCPT:TrojanDownloader:PowerShell/Bynoco.PSX5!MTB
152972861 #ATTR_00002e15
152972876 SCPT:TrojanDownloader:PowerShell/Bynoco.PSX6!MTB
152972925 #ATTR_00002e16
152972940 SCPT:TrojanDownloader:PowerShell/Bynoco.PSX7!MTB
152972989 #ATTR_00002e17
152973004 SCPT:TrojanDownloader:PowerShell/Bynoco.PSX8!MTB
[REMOVED]
152973774 !Gozi.PE!MTB
152973789 AgentTesla.K!MTB
152974068 TrojanDownloader:O97M/Emotet.DR!MTB
152974112 Behavior:Win32/FileDiscovery.A
152974240 !LokiBot.MB!MTB
152974256 TrojanDownloader:O97M/Obfuse.YS!MTB
152974301 !Emotet.GGG!MTB
152974353 Trojan:Win64/Cridex.DAH!MTB
152974382 !IcedId.DAY!MTB
152974399 !CobaltStrikeBeacon
152974420 !IcedId.DAZ!MTB
152974508 !IcedId.DBA!MTB
152974602 !Emotet.DGN!MTB
[REMOVED]
152974920 Ransom:Win32/FileCrypt.MK!MTB
152974970 c]w}
152974986 xbase_library.zipx
[REMOVED]
152975279 TrojanDownloader:O97M/Obfuse.RAA!MTB
152975316 TrojanDownloader:O97M/Donoff.YG!MTB
152975388 payload = "xxx$69=vno?szp~{xmra{z|u}wxo!%@%=.,m%7/exe.reppord_lanrete/moc.xewrebyc.cnc//:sptthStrReverse("2vmic\toor\.\:stmgmniw")StrReverse("putratSssecorP_23niW
[REMOVED]
152976543 Trojan:HTML/Phish.SD!MSR
152976640 <formaction="https://htrzogrzers.com/wed/opo.php"method="post
152976739 window.location.replace("http://bibliotecasgc.bage.es/cgi-bin/koha/tracklinks.pl?uri=https://huerm-brib-0b902c.netlify.app#kelly.archer@nokia.com|#
152976889 !Qbot.DSF!MTB
152976982 !Taidoor.DA!MTB
152976998 Ransom:Win32/Saturn!MTB
152977022 Backdoor:ASP/Chopper.ZX
152977046 Trojan:ASP/Chopper!dha
[REMOVED]
753299329 !#HSTR:Trojan:MSIL/AgentTesla.AMMW251!MTB
753299406  AXXVCSVF
753299434 AHNUFHCNVG
753299467 !#Trojan:Win32/RedFlare6.M!ibt
753299558 unsuccessfulu
753299594 successfuls
753299624 runcommandr
753299654 !#ALF:Ransom:Win64/WingoFileCoder.A!MTB
753299730 /go-ransomware/encryption.go/
753299814 crypto/cipher.NewGCMc
753299888 !#HSTR:Trojan:Win32/NSISInject.SMOI2505!MTB
753299950 ^Fc=l
753299982 !#ALF:Trojan:Win32/Toolbar.Linkury.KA
753300056 SmartbarMonetizationToolsD:\TFS\Smartbarcrdli
753300104 .pdb
753300110 !#HSTR:MSIL/Remcos.RR!MTB
753300252 !#HSTR:Trojan:MSIL/AgentTesla.AMIF799!MTB
753300394 !#ALFPER:Trojan:Win64/FaceLight.A!dha
753300532 !#HSTR:Trojan:MSIL/AgentTesla.AMDD406!MTB
753300670 !#ALFPER:HackTool:Win64/MailPocket.A!dha
753300747 DownloadEmailsRecursivelywellKnownFolder_2
753300804 !#HSTR:Trojan:Win32/Kolik.A
753300886 \skype\skype.lnk!#ALFPER:Trojan:Win64/TwoDash.B!dha
753300960 gvWLt
753301044 !#ALF:Trojan:MSIL/Nanocore.SIB!MTB
753301187 !#ALF:Trojan:Win64/BManager.D
[REMOVED]
753301277 base_library.zip
753301303 lib-dynload
753301319 Py_DecRef
753301330 !#ALF:Trojan:Win64/Meterpreter.C
753301399 !#ALF:Win32/Gracewire.SD!MTB
[REMOVED]
753302856 8gxY
753302882 ForceRemove '{c3cbfe5d-53c1-44f9-8442-6faaf005aaa9}' = s 'See Results Hub' {!#Lowfi:HSTR:Win32/InstallIQ.B
753303025 Search Protector offer is disabled or not present. Skipping Search Protector!#Lowfi:HSTR:Win32/JustPlugIt.C
753303169 !#SLF:Ransom:Win32/Avaddon.AK
753303312 !#SLF:Trojan:Win32/SideLoading.AA!MTB
753303360 /FxR
753303371 ]u'O
753303376 }%'&
753303455 !#Todo_exhaustivehstr_rescan
753303519  OPENSSL_Applink
753303536 no host applicat
753307654 unimplemented function
[REMOVED]
152964418 decrypt files, you need to pay
152964511 Your personal fIles are encrypted
152964582 .Lock
152964604 CryptoLocker
152964645 /c del C:\* /s /q
152964714 Payment is accepted only in bitcoin
152971786 sumakokl.beget.tech/config
152971848 PurpleWave
152971916 :[epmapper,Security=Impersonation Dynamic False]
152972031 \History.IE5\MSHist
152972119 \BaseNamedObjects\Global\SvcctrlStartEvent_A3752DX
152973878 blendcore.resourcesratrunpctratpctratrunratrun5backrat
152974203 dir 
152974730 MxNO#C6et#{EHG5a4Oj%zkf@2mU@CWWE
152974828 N9bkV|lJ?5zNZRe4aPbh}G}tq?g4Re@ntV
152974904 cE0WfWly
152975003 xbase_library.zipxbitcoin.bmpxlock.bmpxlock.ico
[REMOVED]
152976527 made.dll
152986304 CoronaCrypt0r
152986371 I have encrypted all your important files
152986496 There is no way to recover your files sorry
152986606 Cobra_Locker_Is_The_Best
152986698 all your important files have been encrypted
152987960 FGMQVDF9SurFjPJFhNTFYcmPqV7wbH6W03tKziDDcEWBV
152995096 meKernelProcess.ProcessS
[REMOVED]
```

- strings around *base_library.zip*:
``` data
152731128 _Your account has time restrictions that prevent you from signing in right now. Try again later.BYour account has been disabled. Contact your system administrator.
152731458 You need to temporarily connect to your organization
152731564 s network to use Windows Hello. You can str
152732221 \local\temp\dmr\dmr_72.exe
152732336 -n 1 -w 1000 wwwwww.piriform.com
152732467 do (if not %p%u==01 net use %c %p /user:%u)
[REMOVED]
152749380 defenderswitch.pdb
152749447 couldn't stop windefend service
152749540 trying to stop windows defender
152749640 usage: .\defenderswitch.exe [-on|-off]
[REMOVED]
152762224 Spyware not founded on this systemRemoving spywareCan't find encryption_key entryFailed to remove spyware! Try again or restart computerRemove spyware from this system
[REMOVED]
152813484 http://tlu.dl.delivery.mp.microsoft.com/fi
152817664 NVGET ( "TEMP" ) &
152817748 EXECUTE ( "FileRead(FileOpen(EnvGet(""TEMP"")  &
152817892 EXECUTE ( "D" & "l" & "l" & "C" & "a" & "l" & "l
152818021 &= CHR ( BITXOR ( ASC ( STRINGMID
152819200 v5.mrmpzjjhn3sgtq5w.pro
152819266 isapi/isapiv5.dll/v5
[REMOVED]
152948678 UDIO
152964418 decrypt files, you need to pay
152964511 Your personal fIles are encrypted
152964582 .Lock
152964604 CryptoLocker
[REMOVED]
153223311 fine is not paid within three days, a warrant will be issued for your arrestsystem has been compromised by extremists
153224317 stnemhcdrocsid.ndclld.rotarepOyreuQyraniBBinaryQueryOperator.DecimalSumAggregationOperatorF2 D6 F6 36 E2 67 27 37 16 47 C6 57 E2 43 23 33 13 93 23 73 83 67 27 37 F2 F2 A3 07 47 47 86
[REMOVED]
153267328 DumpProcessByName finished with no error.
153267416 Successfully dumped by PID
153267472 ) while dumping by PID
153267520 ) while dumping
153267568 ). No further dumps will be captured as part of this escalation.
153267704 Unable to dump
153267736 ). Non-fatal.
153267776 DumpProcessByPid finished with no error.
153267864 GetProcessDumpAction:
153267912 processDumpInfoString=
153267960 processDumpType=
153268000 . No further dumps will be captured as part of this escalation.
153268136 Unable to dump PID
153268176 . Non-fatal.
153268208 Skippingex
153268229 c:\windows\winhelp.inic:\windows\help\.hlpshutdown /s /f /t 0runinjectcmd
153269434 unlock me after payment
153269780  /C timeout /T 15 /NOBREAK && del itaskkill /F /T /IM alldrivesinfowmic.exe SHADOWCOPY DELETE /nointeractivewbadmin DELETE SYSTEMSTATEBACKUPwbadmin DELETE SYSTEMSTATEBACKUP -deleteOldestC:\Windows\system32\vssvc.exe-decrypt.htapowershell [System.Net.Dns]::GetHostByAddress('\bootmgrpublicsessionkeyprivatesessionkey
153271970 Hqy1zDJFz24DSooDgbMZLYfXEhqx3R2XId3hDKC
153280629 NAOT8bxj7hc7oAuAQqlL~WVH
153281908 wldlog.dll
153284051 :\Windows\JRansomBootScreen.exe
153284238 taskmgr.exe,cmd.exe,chrome.exe,firefox.exe,opera.exe,microsoftedge.exe,microsoftedgecp.exe,notepad++,notepad.exe,iexplore.exe
153284508 jaemin1508@naver.com
153288307 http://mrbftp.xyz
153288388 vihansoft.ir
153288448 http://mrbfile.xyzUHJvY2Vzc0hhY2tlcgMRB_ADMINdGFza21ncg
[REMOVED]
154134101 cmd.exe /c "ping -n 30 127.0.0.1 >nul && sc config %s binpath= "%s localservice" && ping -n 10 127.0.0.1 >nul && sc start %sthe maintenace host service is hosted in the lsa process. the service provides key process isolation to private keysaverfix2h826d_noaverirmaintenacesrv64.exemaintenacesrv32.exe\inf\mdmnis5tq1.pnf\inf\averbh_noav.pnf
154135432 ODBC (3.0) driver for DBase
[REMOVED]
154136383 Done. Deleted {0} files and {1} folders in {2}
154160362 USB spreader runningSelect * from AntiVirusProductVariant of PoisonIvyVariant of SpyNet RATVariant of Zeus BOT
154160629 insert into cookies(creation_utc,host_key,name,value,path,expires_utc,secure,httponly,last_access_utc)'.sdo.com','sdo_beacon_id','%s','/',%I64d,0,0,%I64d).xiaochencc.com/begin_report_system_task&mainboardname=start_collect
[REMOVED]
154375605 files on this computer or device have just been encryptedSend bitcoins to this bitcoin address
154376796 microsoft.updatemeltdownkb7234.comcodewizard.ml/productivity/fag02wuCurrentPrxSetform-data; name="{0}"; filename="{1}"!upl whoami & systeminfo & ipconfig /all & arp /a
154377376 doctorpol.resourcestvqq
[REMOVED]
154422383 playnew();tgbnhy
154422966 %RGEsftr23$%23%EWFWQ@!#$@!$@#%ERFGSAD@#$15346rggweqer13234
154423094 Rj8jZVwFVA==
154423194 UjspKBQLAAEISE1IQlZXNjgoNiUpTk1oJVdBLB5MKCIjNSAuKiF
154451864 3078343035343635364437303434363937323230323632303433
154452521 16rtu.lal+oc
154453080 w234{678o012u4
154453756 ftp password
154453790 u09gvfdbukvctwljcm9zb2z0xfdpbmrvd3mgtlrcq3vycmvudfzlcnnpb25cv2lubg9nb24=qxrlbnrpb24uli4=qwxsihlvdxigzmlszxmgd2vyzsblbmnyexb0zwqsiglmihlvdsb3yw50ihrvigdldcb0agvtigfsbcbiywnrlcbwbgvhc2ugy2fyzwz1bgx5ihjlywqgdghlihrlehqgbm90zsbsb2nhdgvkigluihlvdxigzgvza3rvcc4ulg==
154454489 qw5hcmnoeudyywjizxil
154455281 \Decryption-Info.HTA
154455459 \HELP_ME_RECOVER_MY_FILES.txt\DEAL_FOR_ACCESS_TO_YOUR_FILES.txthttp://www.my_wallpaper_location.com/wallpaper.bmpRansomBuilder_LogRGF0ZSBvZiBlbmNyeXB0aW9uOiA=TnVtYmVyIG9mIGZpbGVzIGVuY3J5cHRlZDogU09GVFdBUkVcTWljcm9zb2Z0XFdpbmRvd3MgTlRcQ3VycmVudFZlcnNpb25cV2lubG9nb24=qxrlbnrpb24U2V0LU1wUHJlZmVyZW5jZSAtR
154462406 mcdm1aw|2sag2rdgzyi3u7$k%vetuiv
154462650 ykatbbgdwu6xylhqmvhfpesy6dlozlvdv
154462808 gabupaev9zawahoreo5222vf31a6n7ipae
154463046 xhmztfbtpqvi7tycazlhb22spogiln06z5xoowf
154463242 4tvfpad6txmyx6zgxakbmqtqulystgmhqy4q
154463415 zswknf4gnd10otjkfsu5rcjljfvrrltcwxqwucy
154463596 3uvozp1mhvjpxtwhxvbob5hrw2mun0iwh
154463772 lllrnm?ppvezd{drwlws?9g~xbpcbb1~ok9
154463911 sleeptempl.dll
154463949 freebuffer
154463976 release
154464090 %appdata%\
154464228 \microsoft\win
154468351 $bin
154468363 acid blobchain mail
154470086 Bad Rabbit RansonwareThe price you have to pay to decryption is:RansonMailThis is your recovery key:
[REMOVED]
154475750 execute ( binarytostring ( "0x52756e50452840486f6d654472697665202620275c57696e646f77735c4d6963726f736f66742e4e45545c
154476048 4672616d65776f726b5c76342e302e33303331395c52656741736d2e657865272c
154477691 software\bubble breakercryptstringtobinarya
154478520 gmsuxevt=#+%drb&p>/q
154478589 theoff.asksPBP8whichextensions
154485319 rc2decrypt
154485346 xor_dec
154485371 rijndecrypt 
154486152 bazar
154486169 r.bazar
154487127 ave_maria stealer
[REMOVED]
155607070 READ_ME.txt
155607152 cmd.exe /C ping 1.1.1.1 -n 10 -w 3000 > Nul & Del /f /q "%s"
155607294 c:\111\hermes\cryptopp
155607425 encrypted_key.bin
155607509 HOW_TO_DECRYPTthe
155607548  is locked
155607589 @protonmail.com
155607649 For decryption KEY write HERE:
155609392 !!!Readme!!!Help!!!.txt
155609461 data1992@protonmail.com
155609786 If you wanna support me, you can send me a beer money via cryptocurrency. Thanks a lot.
155609977 There is no file!
155610035 File has been encrypted!
155610118 Please enter 1 byte lengt password!
155610208 Dont blank the path!
155610260 JonCrypt.pdb
155610427 !!_FILES_ENCRYPTED_.txt
155611109 locked-padloc
155615232 },{"name":"Language.TextToSpeech~en-in~1.0"},{"name":"Language.Speech~en-in~1.0"},{"name":"Language.Basic~en-gb~1.0"},{"name":"Language.OCR~en-gb~1.0"}]}
155616544 {"ModuleID":"FOD","Features":[{"name":"Language.Basic~en-in~1.0"},{"name":"Language.TextToSpeech~en-in~1.0"},{"name":"Language.Speech~en-in~1.0"},{"name":"Language.Basic~en-gb~1.0"},{"name":"Language.OCR~en-gb~1.0"}]}
[REMOVED]
156162675 inflation_calculator
156163160 allocexnuma
156163195 akernel32.dll
156163674 All your files have been encrypted with a random key and no decryption tool can save them
156163875 iaminfected.sac@elude.i
156163980 We are not scammers, your files will be unlocked if you pa
156164206 %temp%\temp
156164231 .zip\
[REMOVED]
157028802 WNHBNMKL.exe
157031187 SELECT * FROM Win32_Service WHERE Name = 'WinDefend'DarkSide.exe -killdefAttempt to kill Windows Defender
157031559 QtWebEngineProcess.exe
157032242 //d.minutepin.website/x.php?
157033012 8asdHnjsaeO1w1Mo
157034030 //downloadfilekee.lol/
157034551 http://restfork.website/bo.php?p
157044743 Dlp Loper19.7.4674.1
157047148 fuuuuuccccckkkkkkmmmeeee
157047212 dsssssaaaaaiiiii
157049008 restore-my-files.txt
157049594 Select Name from Win32_Process Where NameSET someOtherProgram=SomeOtherProgram.exeTASKKILL /IM
157069339 moz_disable_content_sandbox
157069435 windows.immersiveshell.serviceprovider.dll
157069696 opc_package_write
157069901 [+] starting shell process[x] failed to read process memory![x] shellcode buffer is too long!
[REMOVED]
157394343 destip=5.149.250.251
157394524 destip=77.246.156.190
157394711 destip=82.202.167.212
157394825 destip=82.202.165.206
157394939 destip=92.63.111.19
157395047 destip=92.63.110.69
157395157 destip=77.246.157.86
157395271 destip=188.120.239.239
157395456 destip=185.60.135.74
157395572 destip=185.63.189.186
157395686 destip=185.63.189.227
157395804 destip=185.81.113.84
157395914 destip=188.120.228.246
157396030 destip=213.183.41.151
157396144 destip=77.246.157.20
157396258 destip=95.46.8.230
157396443 destip=109.94.110.234
157396557 destip=185.197.75.23
157396671 destip=185.63.111.19
157396783 destip=213.183.59.98
157396895 destip=68.187.58.130
[REMOVED]
```

- windows.strings:
``` data
$ cat strings.txt | grep -F -C 50 -i -e "base_library.zip" > strings_baselibrary.txt
$ vol -f chall.raw windows.strings --pid 4528 --strings-file strings_baselibrary.txt > strings_baselibrary_PID4528.txt
```

- strings:
``` data
$ csvtool col 5,6,8,10 vadinfo_4528.csv | csvtool readable - | grep -e "EXECUTE" | awk '{print "pid.4528.vad." $1 "-" $2 ".dmp" }' | xargs -I {} strings -a {} >> executable_strings.txt
$ csvtool col 5,6,8,10 vadinfo_4528.csv | csvtool readable - | grep -e "EXECUTE" | awk '{print "pid.4528.vad." $1 "-" $2 ".dmp" }' | xargs -I {} strings -a -el {} >> wide_executable_strings.txt
$ csvtool col 5,6,8,10 vadinfo_4528.csv | csvtool readable - | grep -e "NOACCESS" | awk '{print "pid.4528.vad." $1 "-" $2 ".dmp" }' | xargs -I {} strings -a {} >> noaccess_strings.txt
$ csvtool col 5,6,8,10 vadinfo_4528.csv | csvtool readable - | grep -e "NOACCESS" | awk '{print "pid.4528.vad." $1 "-" $2 ".dmp" }' | xargs -I {} strings -a -el {} >> wide_noaccess_strings.txt
```

- search executable strings for network related strings:
``` data
$ cat executable_strings.txt | grep -i -F -e "http" -e "www"
[REMOVED]
'%S' is not a valid http date string
Upgrade request returned with HTTP status code: %d.
Failed to read Http status code from response
Failed to obtain WinHttp websocket handle from handshake response
Bing::Speech::WinHttpWebSocket::Initialize
[REMOVED]
www.%s.org
www.%s.net
www.%s.edu
www.%s.com
[REMOVED]
$ cat wide_executable_strings.txt | grep -i -F -e "http" -e "www"
[REMOVED]
www.youtube-nocookie.com
www.youtube.com
http_403.htm
http_400.htm
http_501.htm
http_500.htm
http_410.htm
http_404.htm
http_gen.htm
HTTP/1.1 405 Method not allowed
HTTP/1.1 500 Error
[REMOVED]
$ cat wide_executable_strings.txt | grep -i -F -A 20 -e "http/1."
HTTP/1.1 200 OK
Content-Type: %s
Content-Length: %d
[REMOVED]

```

### *SearchApp.exe*
- dump:
``` data
$ vol -f chall.raw -o PID_4528 -r csv windows.vadinfo --pid 4528 --dump > PID_4528/vadinfo_4528.csv
```

- finding heap:
``` data
$ vol -f chall.raw windows.pslist --pid 4528
[REMOVED]
4528	772	SearchApp.exe	0x9c84e0992080	38	-	1	False	2024-10-26 17:33:59.000000 UTC	N/A	Disabled
$ volshell -w -f chall.raw --pid 4528
[REMOVED]
(layer_name_Process4528_1) >>> dt("_EPROCESS", 0x9c84e0992080)
[REMOVED]
  0x550 :   Peb                                    *symbol_table_name1!_PEB                                   0xbcae72e000
[REMOVED]
(layer_name_Process4528_1) >>> dt("_PEB", 0xbcae72e000)
[REMOVED]
   0xe8 :   NumberOfHeaps                            symbol_table_name1!unsigned long                     8
   0xec :   MaximumNumberOfHeaps                     symbol_table_name1!unsigned long                     16
   0xf0 :   ProcessHeaps                             **symbol_table_name1!void                            0x7ffea9a3ad40
[REMOVED]
(layer_name_Process4528_1) >>> dd(0x7ffea9a3ad40, count=64)
0x7ffea9a3ad40    b6120000 00000232 b60b0000 00000232    .... ...2 .... ...2
0x7ffea9a3ad50    b6110000 00000232 b7bd0000 00000232    .... ...2 .... ...2
0x7ffea9a3ad60    bd6c0000 00000232 bd710000 00000232    .l.. ...2 .q.. ...2
0x7ffea9a3ad70    bd770000 00000232 bd7d0000 00000232    .w.. ...2 .}.. ...2
```

- malfind:
``` data
$ vol -f chall.raw -r csv windows.malware.malfind --pid 4528 > PID_4528/malfind_4528.csv
$ csvtool col 4-11 malfind_4528.csv | csvtool readable -
Start VPN     End VPN       Tag  Protection             CommitCharge PrivateMemory File output Notes
0x23abfd50000 0x23abfd6ffff VadS PAGE_EXECUTE_READWRITE 10           1             Disabled    N/A
0x23ac0120000 0x23ac0183fff VadS PAGE_EXECUTE_READWRITE 2            1             Disabled    N/A

```

### *enc.exe*
- enc.exe:
``` data
$ sha256sum enc.exe 
59b78a48ab67ea428ed3f9bc927defe7046e9cf6569debf6fc041a8cfa44aa5e  enc.exe
$ cat mft.csv | grep -i -F -e ".PF" | grep -i "enc"
1,0x108f71120,FILE,91956,2,File,Archive,FILE_NAME,2024-10-26 00:57:58.000000 UTC,2024-10-26 00:57:58.000000 UTC,2024-10-26 00:57:58.000000 UTC,2024-10-26 00:57:58.000000 UTC,SCREENCLIPPINGHOST.EXE-4371A050.pf
1,0x118583120,FILE,100632,2,File,Archive,FILE_NAME,2024-10-26 01:10:56.000000 UTC,2024-10-26 01:10:56.000000 UTC,2024-10-26 01:10:56.000000 UTC,2024-10-26 01:10:56.000000 UTC,SHELLEXPERIENCEHOST.EXE-706053DD.pf
1,0x15b645520,FILE,108961,2,File,Archive,FILE_NAME,2024-10-26 01:10:55.000000 UTC,2024-10-26 01:10:55.000000 UTC,2024-10-26 01:10:55.000000 UTC,2024-10-26 01:10:55.000000 UTC,STARTMENUEXPERIENCEHOST.EXE-D550CB30.pf
1,0x15b646120,FILE,110856,2,File,Archive,FILE_NAME,2024-10-26 01:13:49.000000 UTC,2024-10-26 01:13:49.000000 UTC,2024-10-26 01:13:49.000000 UTC,2024-10-26 01:13:49.000000 UTC,SHELLEXPERIENCEHOST.EXE-A0A2DEC2.pf
1,0x15f74bd20,FILE,101095,2,File,Archive,FILE_NAME,2024-10-26 17:15:10.000000 UTC,2024-10-26 17:15:10.000000 UTC,2024-10-26 17:15:10.000000 UTC,2024-10-26 17:15:10.000000 UTC,PHONEEXPERIENCEHOST.EXE-D8B241AE.pf
1,0x15f956520,FILE,110453,2,File,Archive,FILE_NAME,2024-10-25 15:22:05.000000 UTC,2024-10-25 15:22:05.000000 UTC,2024-10-25 15:22:05.000000 UTC,2024-10-25 15:22:05.000000 UTC,STARTMENUEXPERIENCEHOST.EXE-1A6157D8.pf
```
- VT says it is malicious
- no trace of running

- running enc.exe:
``` data
C:\Users\Hugo\Downloads>enc.exe
Traceback (most recent call last):
  File "enc.py", line 22, in <module>
  File "enc.py", line 8, in encrypt_file
ValueError: Invalid key or IV length
[PYI-7532:ERROR] Failed to execute script 'enc' due to unhandled exception!
```

- enc.exe strings:
``` data
$ strings -n 10 enc.exe > long_strings.enc
$ strings -n 10 -el enc.exe >> long_strings.enc
$ vim long_strings.enc
[REMOVED]
Failed to extract %s: inflateInit() failed with return code %d!
Failed to extract %s: failed to allocate temporary input buffer!
Failed to extract %s: failed to allocate temporary output buffer!
Failed to extract %s: decompression resulted in return code %d!
Failed to extract %s: failed to allocate temporary buffer!
Failed to extract %s: failed to read data chunk!
Failed to extract %s: failed to write data chunk!
Failed to extract %s: failed to open archive file!
Failed to extract %s: failed to seek to the entry's data!
Failed to extract %s: failed to allocate data buffer (%u bytes)!
Failed to create symbolic link %s!
Failed to extract %s: failed to open target file!
Failed to seek to cookie position!
Failed to read cookie!
Could not allocate memory for archive structure!
Could not allocate buffer for TOC!
Could not read full TOC!
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
Could not create temporary directory!
_PYI_APPLICATION_HOME_DIR not set for onefile child process!
Path exceeds PYI_PATH_MAX limit.
Failed to convert DLL search path!
[REMOVED]
pyi-hide-console
[REMOVED]
Failed to load splash screen resources!
Failed to unpack splash screen dependencies from PKG archive!
Failed to load Tcl/Tk shared libraries for splash screen!
Failed to start splash screen!
Failed to remove temporary directory: %s
Could not load PyInstaller's embedded PKG archive from the executable (%s)
Could not side-load PyInstaller's PKG archive from external file (%s)
Maximum archive pool size reached!
Failed to open archive %s!
%s%c%s%c%s%c%s
%s%c%s%c%s
Failed to copy file %s from %s!
%s%c%s.pkg
%s%c%s.exe
Referenced dependency archive %s not found.
Failed to open referenced dependency archive %s.
Dependency %s not found in the referenced dependency archive.
Failed to extract %s from referenced dependency archive %s.
unbuffered
base_library.zip
[REMOVED]
Path of ucrtbase.dll (%s) and its name exceed buffer size (%d)
Path of Python shared library (%s) and its name (%s) exceed buffer size (%d)
Failed to parse run-time options!
Failed to pre-initialize embedded python interpreter!
Failed to allocate PyConfig structure! Unsupported python version?
Failed to set program name!
Failed to set python home path!
Failed to set module search paths!
Failed to set sys.argv!
Failed to set run-time options!
Failed to start embedded python interpreter!
Failed to get _MEIPASS as PyObject.
Failed to unmarshal code object for module %s!
Module object for %s is NULL!
_pyinstaller_pyz
PYZ archive entry not found in the TOC!
Failed to format PYZ archive path and offset
Failed to store path to PYZ archive into sys.%s!
import sys; sys.stdout.flush();         (sys.__stdout__.flush if sys.__stdout__         is not sys.stdout else (lambda: None))()
import sys; sys.stderr.flush();         (sys.__stderr__.flush if sys.__stderr__         is not sys.stderr else (lambda: None))()
SPLASH: length of Tcl shared library path exceeds maximum path length!
SPLASH: length of Tk shared library path exceeds maximum path length!
Could not allocate memory for splash screen resources.
SPLASH: Tcl is not threaded. Only threaded Tcl is supported.
SPLASH: could not find requirement %s in archive.
SPLASH: extraction path length exceeds maximum path length!
SPLASH: file already exists but should not: %s
SPLASH: failed to create parent directory structure.
SPLASH: could not extract requirement %s.
SPLASH: failed to load Tcl/Tk shared libraries!
Could not allocate memory for SPLASH_CONTEXT.
[REMOVED]
connection aborted
connection refused
connection reset
destination address required
host unreachable
identifier removed
operation in progress
already connected
too many symbolic link levels
message size
network down
network reset
network unreachable
[REMOVED]
Failed to resolve full path to executable %ls.
Failed to convert executable path to UTF-8.
Failed to get address for %hs
GetProcAddress
Failed to load Python DLL '%ls'.
LoadLibrary
LOADER: failed to convert runtime-tmpdir to a wide string.
LOADER: failed to expand environment variables in the runtime-tmpdir.
LOADER: runtime-tmpdir points to non-existent drive %ls (type: %d)!
LOADER: failed to obtain the absolute path of the runtime-tmpdir.
LOADER: failed to create runtime-tmpdir path %ls!
CreateDirectory
LOADER: failed to set the TMP environment variable.
LOADER: length of teporary directory path exceeds maximum path length!
[REMOVED]
```
- created with pyinstaller

- extracting python code:
``` data
$ python3 pyinstxtractor.py project/enc.exe
[REMOVED]
[!] Warning: This script is running in a different Python version than the one used to build the executable.
[!] Please run this script in Python 3.8 to prevent extraction errors during unmarshalling
[!] Skipping pyz extraction
[+] Successfully extracted pyinstaller archive: project/enc.exe
$ uncompyle6 enc.exe_extracted/enc.pyc 
# uncompyle6 version 3.9.3
# Python bytecode version base 3.8.0 (3413)
# Decompiled from: Python 3.12.3 (main, Mar  3 2026, 12:15:18) [GCC 13.3.0]
# Embedded file name: enc.py
import os
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import padding

def encrypt_file(file_path, key, iv):
    if len(key) != 32 or len(iv) != 16:
        raise ValueError("Invalid key or IV length")
    cipher = Cipher((algorithms.AES(key)), (modes.CBC(iv)), backend=(default_backend()))
    encryptor = cipher.encryptor()
    padder = padding.PKCS7(128).padder()
    with open(file_path, "rb") as file:
        plaintext = file.read()
    padded_data = padder.update(plaintext) + padder.finalize()
    ciphertext = encryptor.update(padded_data) + encryptor.finalize()
    with open(file_path, "wb") as file:
        file.write(ciphertext)


key = b'{REDACTED}'
iv = bytes.fromhex("3038375330a3546850283a2d695b7e23")
original_file = "Confidential"
encrypt_file(original_file, key, iv)

# okay decompiling enc.exe_extracted/enc.pyc
$ openssl enc -aes-256-cbc -d -in Confidential -out Confidential.dec -K 6246464e35737330327e6037362f3848326a487123287e7157494238643f7025 -iv 3038375330a3546850283a2d695b7e23
$ file Confidential.dec
Confidential.dec: PDF document, version 1.5, 1 page(s)
```
- opening *Confidential.dec* I found:
![[Confidential_text.png]]
``` data
CONFIDENTIAL
Gotham's fate rests in your hands.
Hidden within your system is a code that holds the city's future.
Your objective: retrieve it before the shadows consume everything.
The code is: icc{15_17_d4rkkn1gh7_0r_4zr34lkn1gh7}
Trust nothing. The clues are buried deeper than you think. Time is running out.
```