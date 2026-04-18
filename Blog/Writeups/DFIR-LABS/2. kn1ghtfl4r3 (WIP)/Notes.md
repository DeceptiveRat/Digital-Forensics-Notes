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
