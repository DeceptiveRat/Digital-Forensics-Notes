## 1. strings
- ASCII and Unicode are encodings in which Windows APIs expect to receive arguments

## 2. string analysis objectives
- extract strings from memory dump
- translate strings: volatility can map physical offsets of strings to VA in memory dump; linking them to specific processes or modules
- leverage strings in unallocated or freed storage
- identify shared pages:
	- possible for multiple processes to map view of same physical page
	- can determine processes involved in same activity

## 3. extracting strings
- accepted formats by volatility:
	``` data
	<decimal_offset>:<string>
	<decimal_offset> <string>
	```
- ASCII and Unicode strings both should be extracted
- Windows tools:
	- *strings.exe*:
		``` cmd
		C:\Users\Jake\Tools> strings.exe –q –o memory.dmp > strings.txt
		```
		- `-o` displays decimal offsets
		- `-q` suppresses banner
- Linux tools:
	- *strings*:
		``` sh
		$ strings -td -a memory.dmp > strings.txt
		$ strings -td -el -a memory.dmp >> strings.txt
		```
		- `-td` specifies decimal-based offsets
		- `-el` sets encoding to little-endian 16-bit (Unicode)
		- `-a` covers entire file, not just non-executable sections
- Mac OS X tools:
	- *strings.exe* via *Wine*

## 4. translating strings
- `strings` plugin translates offsets in physical memory to virtual memory:
	- traverses page tables of all processes in active process list, including `System`
	- determines which processes had access to specified strings
- `strings` plugin output:
	``` data
	$ python vol.py strings –s strings.txt –f memory.dmp --profile=Win7SP0x64 > translated.txt
	$ cat translated.txt
	[snip]
	470696013 [ntoskrnl.exe:804d704d] !This program cannot be run in DOS
	470696799 [ntoskrnl.exe:804d735f] `PAGESPECC
	470696919 [ntoskrnl.exe:804d73d7] @PAGEDATAX
	470697040 [ntoskrnl.exe:804d7450] PAGEVRFCI4
	470697079 [ntoskrnl.exe:804d7477] @PAGEVRFDH
	470697816 [ntoskrnl.exe:804d7758] \REGISTRY\MACHINE\SYSTEM\DISK
	470697848 [ntoskrnl.exe:804d7778] \Device\Harddisk%d\Partition%d
	470697880 [ntoskrnl.exe:804d7798] 2600.xpsp.080413-2111
	470699252 [ntoskrnl.exe:804d7cf4] _nextafter
	[snip]
	507653344 [1024:75261ce0] rundll32.exe
	507653360 [1024:75261cf0] Software\Microsoft\Windows\CurrentVersion\RunOnce
	507653412 [1024:75261d24] explorer.exe
	507653428 [1024:75261d34] iernonce.dll
	507653444 [1024:75261d44] InstallOCX: End %1
	```
- `linux-strings` and `mac-strings` plugins can translate strings found in Linux and Mac OS X memory dumps

## 5. string-based analysis
- can reduce noise by setting minimum string length
- finding prefetch files:
	``` data
	$ grep "\.pf" translated.txt | grep ' [A-Z]\.EXE'
	50711138 [kernel:c15c2a62] R.EXE-19834F9B.pf-0
	55875810 [kernel:c15c08e2] G.EXE-24E91AA8.pfDA
	55892778 [kernel:c15c1b2a] W.EXE-0A1E603F.pf5B
	122417914 [kernel:e15c22fa] G.EXE-24E91AA8.pfG.EXE-24E91AA8.PF
	225133922 [kernel:e0ac4562] R.EXE-19834F9B.pf
	278414074 [kernel:e106e2fa] P.EXE-04500029.pfP.EXE-04500029.PF
	332995290 [kernel:e190aada] W.EXE-0A1E603F.pfW.EXE-0A1E603F.PF
	404921698 [kernel:e0ac0d62] W.EXE-0A1E603F.pf
	420774242 [kernel:e0ac1162] G.EXE-24E91AA8.pf
	455987554 [kernel:e0ac2162] P.EXE-04500029.pf
	```
	- presence of prefetch file indicates execution
- spatial proximities with IoCs:
	- related artifacts are likely to be near IoCs
	- useful *grep* options:
		- `-A NUM`: print `NUM` lines after string
		- `-B NUM`: print `NUM` lines before string
		- `-C NUM`: print `NUM` lines before and after string
	- example:
		``` data
		$ grep -A 30 "66.32.119.38" translated.txt
		361990424 [kernel:c75bf918] open 66.32.119.38
		361990443 [kernel:c75bf92b] jack
		361990449 [kernel:c75bf931] 2awes0me
		361990459 [kernel:c75bf93b] lcd c \WINDOWS\System32\systems
		361990492 [kernel:c75bf95c] cd/home/jack
		361990508 [kernel:c75bf96c] binary
		361990516 [kernel:c75bf974] mput "*.txt"
		361990530 [kernel:c75bf982] disconnect
		[snip]
		```
		- reveals many strings associated with IoC
- strings in free memory:
	- `strings` plugin uses `FREE_MEMORY` indicator for strings in unallocated storage:
		``` data
		209952762 [FREE MEMORY] mciFre\\?\globalroot\Device\Scsi\vmscsi1\revxtepo\revxtepo\tdlwsp.dll
		```
	- dumping only unallocated strings:
		``` sh
		$ grep "FREE MEMORY" translated.txt > unallocated.txt
		$ wc -l translated.txt
		881286 translated.txt
		$ wc -l unallocated.txt
		551218 unallocated.txt
		```
	- AV and *Host Intrusion Preventation System(HIPS)* scan only allocated memory; even kernel mode drivers can't access freed pages unless they directly map them
- detecting shared pages:
	- reasons for shared pages:
		- shared libraries: system DLLs such as *kernel32.dll* and *ntdll.dll* are loaded by every process
		- shared file mappings: processes may need to access same file
		- named shared memory: form of *Inter-Process Communication(IPC)*
	- `strings` plugin shows processes with access:
		``` data
		$ python vol.py -f case24888.dmp --profile=Win2003SP2x86 strings -s strings.txt
		[snip]
		1674e03b [1140:1000603b] 1http://www.microsoft-REDACTED-info.com/mls/shrt4.gif
		168291e4 [1140:00ece1e4] C:\Documents and Settings\Default User.WINDOWS\Local Settings\Temporary Internet Files\Content.IE5\G9E7C5ER\shrt4[2].gif
		16949bc0 [1140:00ecdbc0] C:\Documents and Settings\Default User.WINDOWS\Local Settings\Temporary Internet Files\Content.IE5\G9E7C5ER\shrt4[2].gif
		169a17e8 [1140:00f267e8 980:046a67e8] http://www.microsoft-REDACTED-info.com/mls/shrt4.gif
		169a181c [1140:00f2681c 980:046a681c] shrt4[2].gif
		169a1968 [1140:00f26968 980:046a6968] http://www.microsoft-REDACTED-info.com/mls/shrt4.gif
		169a199c [1140:00f2699c 980:046a699c] shrt4[2].gif
		169a1ae8 [1140:00f26ae8 980:046a6ae8] http://www.microsoft-REDACTED-info.com/mls/shrt4.gif
		169a1b1c [1140:00f26b1c 980:046a6b1c] shrt4[2].gif
		```
		- strings are in shared memory between PID 1140 and 980
	- check if string is in dll with `dlllist` plugin:
		``` data
		$ python vol.py -f case24888.dmp --profile=Win2003SP2x86 dlllist -p 1140
		Volatility Foundation Volatility Framework 2.4
		************************************************************************
		spoolsv.exe pid: 1140
		Command line : C:\WINDOWS\system32\spoolsv.exe
		Service Pack 2
		Base Size LoadCount Path
		---------- ---------- ---------- ----
		[snip]
		0x10000000 0xa000 0x1 C:\WINDOWS\system32\winpugtr.dll
		[snip]
		```
		- *winpugtr.dll* contains first malicious string
		- no dlls occupy space of other strings
	- dump dll with `dlldump` to confirm:
		``` data
		$ python vol.py -f case24888.dmp --profile=Win2003SP2x86 dlldump -p 1140 -b 0x10000000 -D OUTDIR
		Volatility Foundation Volatility Framework 2.4
		Process(V) Name Module Base Module Name Result
		---------- -------------------- ----------- -------------------- ------
		0x89841d88 spoolsv.exe 0x010000000 winpugtr.dll OK: module.1140.9841d88.10000000.dll
		$ strings -a OUTDIR/module.1140.9841d88.10000000.dll
		[snip]
		1http://www.microsoft-REDACTED-info.com/mls/shrt4.gif
		[snip]
		```
		- PID 1140 hosting malicious DLL confirmed
	- examine Internet history with `iehistory` plugin:
		``` data
		$ python vol.py -f case24888.dmp --profile=Win2003SP2x86 iehistory -p 1140,980
		[snip]
		**************************************************
		Process: 980 svchost.exe
		Cache type "URL " at 0x46a6780
		Record length: 0x180
		Location: http://www.microsoft-REDACTED-info.com/mls/shrt4.gif
		Last modified: 2010-07-14 01:05:16 UTC+0000
		Last accessed: 2010-09-21 03:03:10 UTC+0000
		File Offset: 0x180, Data Offset: 0x9c, Data Length: 0xac
		File: shrt4[2].gif
		Data: HTTP/1.1 200 OK^M
		ETag: "1c8428-5ac-48b4e9424eb00"^M
		Content-Length: 1452^M
		Content-Type: image/gif^M
		^M
		~U:system^M
		**************************************************
		Process: 980 svchost.exe
		Cache type "URL " at 0x46a6900
		Record length: 0x180
		Location: http://www.microsoft-REDACTED-info.com/mls/shrt4.gif
		Last modified: 2010-07-14 01:05:16 UTC+0000
		Last accessed: 2010-10-23 20:25:29 UTC+0000
		File Offset: 0x180, Data Offset: 0x9c, Data Length: 0xac
		File: shrt4[2].gif
		Data: HTTP/1.1 200 OK^M
		ETag: "1c8428-5ac-48b4e9424eb00"^M
		Content-Length: 1452^M
		Content-Type: image/gif^M
		^M
		~U:system^M
		```
		- strings were in cached IE history file (*index.dat*); does not mean PID 980 is involved with download of *shrt4.gif*

## 6. command history
- *cmd.exe* does not have capability to log commands to history file

## 7. command history analysis objectives
- recover commands from terminated shells
- extract full console input and output buffers
- enumerate and translate aliases
- reconstruct user activities

## 8. Windows command architecture
- *cmd.exe* is a console application, but still needs to engage in GUI activities; e.g. minimizing window, copy-paste requests, scrolling
- starting with Windows 7, console host process (*conhost.exe*) brokers necessary GUI functionality
- *conhost.exe* runs with permissions of user who started command shell
- commands entered into *cmd.exe* are processed by *conhost.exe*

## 9. important console module functions
- `SrvAllocConsole`: 
	- creates new console
	- each console can have multiple screens, command histories, and aliases
- `AllocateCommandHistory`: 
	- creates new command history buffer
- `AddCommand`: 
	- add command to history buffer
	- by default uses `FindMatchingCommand` and `RemoveCommand` to avoid storing duplicate commands
- `FindCommandHistory`: given process handle, finds command history
- `SrvAddConsoleAlias`: add command alias to console

## 10. console module DS
- `_gConsoleInformation` symbol in *conhost.exe* points to array of `_CONSOLE_INFORMATION` structures
- `_CONSOLE_INFORMATION` contains pointers to doubly linked list of history buffers (`_COMMAND_HISTORY`) and singly linked list of screens (`_SCREEN_INFORMATION`)
- `_SCREEN_INFORMATION` contains screen contents, including input and output
- diagram:
	![[console_module_DS_diagram.png]]
- `_CONSOLE_INFORMATION`:
	``` python
	>>> dt(“_CONSOLE_INFORMATION”)
	‘_CONSOLE_INFORMATION’
	0x28 : ProcessList [‘_LIST_ENTRY’]
	0xe0 : CurrentScreenBuffer [‘pointer’, [‘_SCREEN_INFORMATION’]]
	0xe8 : ScreenBuffer [‘pointer’, [‘_SCREEN_INFORMATION’]]
	0x148 : HistoryList [‘_LIST_ENTRY’]
	0x158 : ExeAliasList [‘_LIST_ENTRY’]
	0x168 : HistoryBufferCount [‘unsigned short’]
	0x16a : HistoryBufferMax [‘unsigned short’]
	0x16c : CommandHistorySize [‘unsigned short’]
	0x170 : OriginalTitle [‘pointer’, [‘String’, {‘length’: 256, ‘encoding’: ‘utf16’}]]
	0x178 : Title [‘pointer’, [‘String’, {‘length’: 256, ‘encoding’: ‘utf16’}]]
	```
	- `ProcessList`: doubly linked list of `_CONSOLE_PROCESS`, one for each process attached to console
	- `CurrentScreenBuffer`: screen that's currently displayed
	- `ScreenBuffer`: singly linked list of all available screens
	- `HistoryList`: doubly linked list of `_COMMAND_HISTORY`
	- `ExeAliasList`: doubly linked list of all executable aliases added to console
	- `HistoryBufferCount`: number of command elements in `HistoryList`
	- `CommandHistorySize`: maximum number of commands that canbe saved in `_COMMAND_HISTORY`; default 50
	- `Title`:
		- title for console window
		- typically path to *cmd.exe*
- `_SCREEN_INFORMATION`:
	``` python
	>>> dt(“_SCREEN_INFORMATION”)
	‘_SCREEN_INFORMATION’
	0x8 : ScreenX [‘short’]
	0xa : ScreenY [‘short’]
	0x48 : Rows [‘pointer’, [‘array’, lambda x: x.ScreenY, [‘_ROW’]]]
	0x128 : Next [‘pointer’, [‘_SCREEN_INFORMATION’]]
	```
	- `ScreenX`: 
		- console width in characters
		- default 80
	- `ScreenY`: 
		- console height in rows
		- default 300
	- `Rows`: array of `_ROW`, which store content displayed within screen buffer
- `_ROW`:
	``` python
	>>> dt(“_ROW”)
	‘_ROW’
	0x8 : Chars [‘pointer’, [‘String’, {‘length’: 256, ‘encoding’: ‘utf16’}]]
	```
- `_COMMAND_HISTORY`:
	``` python
	>>> dt(“_COMMAND_HISTORY”)
	‘_COMMAND_HISTORY’
	0x0 : ListEntry [‘_LIST_ENTRY’]
	0x10 : Flags [‘Flags’, {‘bitmap’: {‘Reset’: 1, ‘Allocated’: 0}}]
	0x18 : Application [‘pointer’, [‘String’, {‘length’: 256, ‘encoding’: ‘utf16’}]]
	0x20 : CommandCount [‘short’]
	0x22 : LastAdded [‘short’]
	0x24 : LastDisplayed [‘short’]
	0x26 : FirstCommand [‘short’]
	0x28 : CommandCountMax [‘short’]
	0x30 : ProcessHandle [‘address’]
	0x38 : PopupList [‘_LIST_ENTRY’]
	0x48 : CommandBucket [‘array’, lambda x: x.CommandCount, [‘pointer’, [‘_COMMAND’]]]
	```
	- `ListEntry`: doubly linked list connecting command history structures
	- `Flags`: set to `Reset` when structure marked for deletion
	- `Application`: Unicode name of application connected to console
	- `CommandCount`: size of history buffer
	- `CommandCountMax`: 
		- max size of history buffer
		- default 50
		- should match `_CONSOLE_INFORMATION.CommandHistorySize`
	- `ProcessHandle`: points to `CSR_PROCESS` from which you can derive `_EPROCESS` address
	- `CommandBucket`: contains `CommandCount` number of `_COMMAND`, one for each command user typed
- `_COMMAND`:
	``` python
	>>> dt(“_COMMAND”)
	‘_COMMAND’
	0x0 : CmdLength [‘unsigned short’]
	0x2 : Cmd [‘String’, {‘length’: lambda x: x.CmdLength, ‘encoding’: ‘utf16’}]
	```

## 11. console window default settings
- `HKEY_CURRENT_USER\Console` registry is authoritative source of most settings
- default settings changing:
	- resetting registry values:
		![[registry_console_settings.png]]
	- Properties -> Options -> Layout in *cmd.exe*:
		![[cmd_console_settings.png]]
	
## 12. `CmdScan` plugin
- finds all `_COMMAND_HISTORY` in pages of memory owend by *conhost.exe* 
- cycles through `CommandBucket` to find entries that contain valid commands
- example output:
	``` data
	$ python vol.py -f iis_server.mem --profile=Win2003SP2x86 cmdscan
	Volatility Foundation Volatility Framework 2.4
	**************************************************
	CommandProcess: csrss.exe Pid: 484
	CommandHistory: 0x4e4ed8 Application: CNTAoSMgr.exe Flags: Allocated
	CommandCount: 0 LastAdded: -1 LastDisplayed: -1
	FirstCommand: 0 CommandCountMax: 50
	ProcessHandle: 0xf24
	**************************************************
	CommandProcess: csrss.exe Pid: 7888
	CommandHistory: 0x4c2c30 Application: cmd.exe Flags: Allocated
	CommandCount: 12 LastAdded: 11 LastDisplayed: 11
	FirstCommand: 0 CommandCountMax: 50
	ProcessHandle: 0x25c
	Cmd #0 @ 0x4c1f90: d:
	Cmd #1 @ 0xf41280: cd inetlogs
	Cmd #2 @ 0xf412e8: cd w*46
	Cmd #3 @ 0xf41340: type ex<REDACTED>.log | find "<REDACTED>.jpg" | find "GET"
	Cmd #4 @ 0xf41b10: c:
	Cmd #5 @ 0xf412a0: cd\windows\system32\<REDACTED>\sample
	Cmd #6 @ 0xf41b20: ftp <REDACTED>.com
	Cmd #7 @ 0xf41948: notepad ex<REDACTED>.log
	Cmd #8 @ 0x4c2388: notepad ex<REDACTED>.log
	Cmd #9 @ 0xf43e70: ftp <REDACTED>.com
	Cmd #10 @ 0xf43fb0: dir
	Cmd #11 @ 0xf41550: notepad ex<REDACTED>.log
	```
	- PID 484 opened command shell, but didn't use
	- PID 7888 shows activity from attacker

## 13. `Consoles` plugin
- looks for `_CONSOLE_INFORMATION`
- has access to screen buffers that contain input and output of console window
- example output:
	``` data
	$ python vol.py -f iis_server.mem --profile=Win2003SP2x86 consoles
	Volatility Foundation Volatility Framework 2.4
	[snip]
	**************************************************
	ConsoleProcess: csrss.exe Pid: 7888
	Console: 0x4c2404 CommandHistorySize: 50
	HistoryBufferCount: 4 HistoryBufferMax: 4
	OriginalTitle: Command Prompt
	Title: Command Prompt
	AttachedProcess: cmd.exe Pid: 5544 Handle: 0x25c
	----
	CommandHistory: 0x4c2c30 Application: cmd.exe Flags: Allocated, Reset
	CommandCount: 12 LastAdded: 11 LastDisplayed: 11
	FirstCommand: 0 CommandCountMax: 50
	ProcessHandle: 0x25c
	Cmd #0 at 0x4c1f90: d:
	Cmd #1 at 0xf41280: cd inetlogs
	Cmd #2 at 0xf412e8: cd w*46
	Cmd #3 at 0xf41340: type <REDACTED>.log | find "<REDACTED>.jpg" | find "GET"
	Cmd #4 at 0xf41b10: c:
	Cmd #5 at 0xf412a0: cd\windows\system32\<REDACTED>\sample
	Cmd #6 at 0xf41b20: ftp <REDACTED>.com
	Cmd #7 at 0xf41948: notepad <REDACTED>.log
	Cmd #8 at 0x4c2388: notepad <REDACTED>.log
	Cmd #9 at 0xf43e70: ftp <REDACTED>.com
	Cmd #10 at 0xf43fb0: dir
	Cmd #11 at 0xf41550: notepad <REDACTED>.log
	----
	Screen 0x4c2b10 X:80 Y:3000
	Dump:
	Microsoft Windows [Version 5.2.3790]
	(C) Copyright 1985-2003 Microsoft Corp.
	C:\Documents and Settings\Administrator.<REDACTED>>d:
	D:\>cd inetlogs
	D:\inetlogs>cd w*46
	D:\inetlogs\<REDACTED>>type <REDACTED>.log | find "<REDACTED>.jpg" | find "GET" 2012-05-23 02:51:19 W3SVC481486246 X.X.83.22 GET <REDACTED>.jpg - 80 – X.X.110.161 Mozilla/4.0+(compatible;+MSIE+7.0;+Windows+NT+5.1;+Trident/4.0) 200 0 0
	D:\inetlogs\<REDACTED>>c
	[snip]
	```
