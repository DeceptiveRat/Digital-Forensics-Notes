## 1. *Graphical User Interface(GUI)*
- responsible for managing user input; e.g. mouse movements and keystrokes
- draws display surface; windows, buttons, menus, etc
- provides isoloation to support multiple concurrent users logged in via console, RDP and Fast User Switching
- GUI diagram:
	![[GUI_diagram.png]]
- majority of GUI-related APIs exposed to user mode are implemented in *user32.dll* and *gdi32.dll*; equivalent to *ntdll.dll*
- SSDT routes GUI calls into *win32k.sys* kernel module rather than NT executive module:
	![[native_GUI_API.png]]

## 2. session
- outermost container; represents user's logon environment
- have unique session IDs
- created upon logon via Fast-User Switching, RDP, or *Terminal Services(TS)*
- tied to a particular user; resources linked to session can be attributed back to actions by user
- resources include:
	- atom table (group of strings shared globally between applications in the session)
	- one or more window stations (named security boundaries)
	- handle table for USER objects (similar to executive objects, but managed by GUI subsystem)

## 3. window station
- applications that require user input run in an interactive window station (e.g. `WinSta0`)
- security boundaries for processes and desktops
- services that run in the background use noninteractive window stations
- contains:
	- atom table
	- clipboard
	- one or more desktops

## 4. desktop
- container for application windows and user interface objects; e.g. windows, menus, hooks, heap

## 5. window
- can be visible or invisible
- contains:
	- screen coordinates
	- window procedure (function that executes when window messages are received)
	- optional caption or title
	- associated class

## 6. GUI forensics plugins
- plugin table:

	| Plugin | Description |
	| --- | --- |
	| sessions | Lists details on user logon sessions |
	| wndscan | Enumerates window stations and their properties |
	| deskscan | Analyzes desktops and associated threads |
	| atomscan | Scans for atoms(globally shared strings) |
	| atoms | Prints session and window station atom tables |
	| messagehooks | Lists desktop and thread window message hooks |
	| eventhooks | Prints details on windows event hooks |
	| windows | Enumerates windows |
	| wintree | Prints Z-Order desktop windows trees |
	| gahti | Dumps the USER handle type information |
	| userhandles | Dumps the USER handle table objects |
	| gditimers | Examines the use of GDI timers |
	| screenshot | Saves a pseudo screenshot based on GDI windows |

## 7. session analysis objectives
- associate processes with RDP users
- detect hidden processes: per session process list can be leveraged to find processes unlinked from `PsActiveProcessHead`
- determine kernel drivers

## 8. session DS
- `_MM_SESSION_SPACE`:
	``` python
	>>> dt("_MM_SESSION_SPACE")
	'_MM_SESSION_SPACE' (8064 bytes)
	0x0 : ReferenceCount ['long']
	0x4 : u ['__unnamed_2145']
	0x8 : SessionId ['unsigned long']
	0xc : ProcessReferenceToSession ['long']
	0x10 : ProcessList ['_LIST_ENTRY']
	0x20 : LastProcessSwappedOutTime ['_LARGE_INTEGER']
	0x28 : SessionPageDirectoryIndex ['unsigned long long']
	0x30 : NonPagablePages ['unsigned long long']
	0x38 : CommittedPages ['unsigned long long']
	0x40 : PagedPoolStart ['pointer64', ['void']]
	0x48 : PagedPoolEnd ['pointer64', ['void']]
	0x50 : SessionObject ['pointer64', ['void']]
	0x58 : SessionObjectHandle ['pointer64', ['void']]
	0x64 : SessionPoolAllocationFailures ['array', 4, ['unsigned long']]
	0x78 : ImageList ['_LIST_ENTRY']
	0x88 : LocaleId ['unsigned long']
	0x8c : AttachCount ['unsigned long']
	0x90 : AttachGate ['_KGATE']
	0xa8 : WsListEntry ['_LIST_ENTRY']
	0xc0 : Lookaside ['array', 21, ['_GENERAL_LOOKASIDE']]
	0xb40 : Session ['_MMSESSION']
	0xb98 : PagedPoolInfo ['_MM_PAGED_POOL_INFO']
	0xc00 : Vm ['_MMSUPPORT']
	0xc88 : Wsle ['pointer64', ['_MMWSLE']]
	0xc90 : DriverUnload ['pointer64', ['void']]
	0xcc0 : PagedPool ['_POOL_DESCRIPTOR']
	0x1e00: PageDirectory ['_MMPTE']
	0x1e08: SessionVaLock ['_KGUARDED_MUTEX']
	0x1e40: DynamicVaBitMap ['_RTL_BITMAP']
	0x1e50: DynamicVaHint ['unsigned long']
	0x1e58: SpecialPool ['_MI_SPECIAL_POOL']
	0x1ea0: SessionPteLock ['_KGUARDED_MUTEX']
	0x1ed8: PoolBigEntriesInUse ['long']
	0x1edc: PagedPoolPdeCount ['unsigned long']
	0x1ee0: SpecialPoolPdeCount ['unsigned long']
	0x1ee4: DynamicSessionPdeCount ['unsigned long']
	0x1ee8: SystemPteInfo ['_MI_SYSTEM_PTE_TYPE']
	0x1f30: PoolTrackTableExpansion ['pointer64', ['void']]
	0x1f38: PoolTrackTableExpansionSize ['unsigned long long']
	0x1f40: PoolTrackBigPages ['pointer64', ['void']]
	0x1f48: PoolTrackBigPagesSize ['unsigned long long']
	[snip]
	```
	- `SessionId`: uniquely identifies session
	- `ProcessList`:
		- alternate process list
		- each process belongs to exactly one session, besides `System` process and *smss.exe*
	- `ImageList`: 
		- list of `_IMAGE_ENTRY_IN_SESSION` structures; one for each device driver mapped into session space
		- copy of *win32k.sys* exists in each session

## 9. RDP
- `sessions` plugin can be used to find RDP sessions:
	``` sh
	$ python vol.py -f rdp.mem --profile=Win2003SP2x86 sessions
	[snip]
	**************************************************
	Session(V): f79ff000 ID: 2 Processes: 10
	PagedPoolStart: bc000000 PagedPoolEnd bc3fffff
	Process: 7888 csrss.exe 2012-05-23 02:51:43
	Process: 3272 winlogon.exe 2012-05-23 02:51:43
	Process: 6772 rdpclip.exe 2012-05-23 02:52:00
	Process: 5132 explorer.exe 2012-05-23 02:52:00
	[snip]
	Image: 0x877d0478, Address bf9d3000, Name: dxg.sys
	Image: 0x8a1bdf38, Address bff60000, Name: RDPDD.dll
	Image: 0x8771a970, Address bfa1e000, Name: ATMFD.DLL
	```
	- *RDPDD.dll* is the RDP display driver
	- *rdpclip.exe* handles remote clipboard operations
- *Volatility* can extract command histories and entire screen buffers from RAM:
	``` sh
	$ python vol.py -f rdp.mem --profile=Win2003SP2x86 consoles
	Volatility Foundation Volatility Framework 2.4
	**************************************************
	ConsoleProcess: csrss.exe Pid: 7888
	Console: 0x4c2404 CommandHistorySize: 50
	HistoryBufferCount: 4 HistoryBufferMax: 4
	OriginalTitle: Command Prompt
	Title: Command Prompt
	AttachedProcess: cmd.exe Pid: 5544 Handle: 0x25c
	----
	CommandHistory: 0xf41610 Application: ftp.exe Flags: Reset
	CommandCount: 19 LastAdded: 18 LastDisplayed: 18
	FirstCommand: 0 CommandCountMax: 50
	ProcessHandle: 0x0
	Cmd #0 at 0xf43b58: xxxxxxxxxx
	Cmd #1 at 0xf41788: cd statistics
	Cmd #2 at 0xf43b78: cd logs
	Cmd #3 at 0xf43db0: dir
	Cmd #4 at 0x4c1eb8: cd xxxxxxxxxx
	Cmd #5 at 0xf43dc0: dir
	Cmd #6 at 0xf43de0: get xxxxxxxxxx.log
	Cmd #7 at 0xf43e30: get for /bin/ls.
	Cmd #8 at 0x4c2ac0: ge xxxxxxxxxx.log
	Cmd #9 at 0xf43dd0: bye
	[snip]
	----
	Screen 0x4c24b4 X:80 Y:3000
	Dump:
	Microsoft Windows [Version 5.2.3790]
	(C) Copyright 1985-2003 Microsoft Corp.
	C:\Documents and Settings\ xxxxxxxxxx >d:
	D:\>cd inetlogs
	D:\inetlogs>cd xxxxxxxxxx
	D:\inetlogs\ xxxxxxxxxx >type xxxxxxxxxx.log | find " xxxxxxxxxx " | \
		find "GET" 2012-05-23 02:51:19 W3SVC xxxxxxxxxx xxxxxxxxxx \
		GET xxxxxxxxxx xxxxxxxxxx - 80 – xxxxxxxxxx \
		Mozilla/4.0+(compatible;+MSIE+7.0;+Windows+NT+5.1;\
		+Trident/4.0) 200 0 0
	```

## 10. window stations analysis objectives
- detect clipboard snooping
- determine clipboard usage

## 11. window stations DS
- window stations and desktops are securable objects; allocated, managed, freed by executive object manager that handles processes, threads, etc
- have `_POOL_HEADER` and `_OBJECT_HEADER`; can locate easily using pool scanning
- if PDB symbols are available, can walk list of window stations via `win32k!grpWinStaList`
- `tagWINDOWSTATION`:
	``` python
	>>> dt("tagWINDOWSTATION")
	'tagWINDOWSTATION' (152 bytes)
	0x0 : dwSessionId ['unsigned long']
	0x8 : rpwinstaNext ['pointer64', ['tagWINDOWSTATION']]
	0x10 : rpdeskList ['pointer64', ['tagDESKTOP']]
	0x18 : pTerm ['pointer64', ['tagTERMINAL']]
	0x20 : dwWSF_Flags ['unsigned long']
	0x28 : spklList ['pointer64', ['tagKL']]
	0x30 : ptiClipLock ['pointer64', ['tagTHREADINFO']]
	0x38 : ptiDrawingClipboard ['pointer64', ['tagTHREADINFO']]
	0x40 : spwndClipOpen ['pointer64', ['tagWND']]
	0x48 : spwndClipViewer ['pointer64', ['tagWND']]
	0x50 : spwndClipOwner ['pointer64', ['tagWND']]
	0x58 : pClipBase ['pointer', ['array', <function <lambda> at 0x10195a848>, ['tagCLIP']]]
	0x60 : cNumClipFormats ['unsigned long']
	0x64 : iClipSerialNumber ['unsigned long']
	0x68 : iClipSequenceNumber ['unsigned long']
	0x70 : spwndClipboardListener ['pointer64', ['tagWND']]
	0x78 : pGlobalAtomTable ['pointer64', ['void']]
	0x80 : luidEndSession ['_LUID']
	0x88 : luidUser ['_LUID']
	0x90 : psidUser ['pointer64', ['void']]
	```
	- `dwSessionId`: associates window station to owning session
	- `rpwinstaNext`: doubly linked list enumerating all windows stations in same session
	- `rpdeskList`: pointer to window station's first desktop
	- `dwWSF_Flags`: tells whether window station is interactive
	- `pClibBase`:
		- pointer to array of `tagCLIP` structures that describe available clipboard formats and contain handles to clipboard objects
		- array size is taken from `cNumClipFormats`
	- `iClipSequenceNumber`: increments by 1 for each object copied to clipboard
	- `pGlobalAtomTable`: points to window station's atom table
	- several fields can tell which thread is viewing the clipboard, which thread owns the clipboard, and which thread may be listening or snooping clipboard operations

## 12. `wndscan` plugin
- example output:
	``` sh
	$ python vol.py -f rdp.mem --profile=Win2003SP2x86 wndscan
	Volatility Foundation Volatility Framework 2.4
	**************************************************
	WindowStation: 0x8581e40, Name: WinSta0, Next: 0x0
	SessionId: 2, AtomTable: 0xe7981648, Interactive: True
	Desktops: Default, Disconnect, Winlogon
	ptiDrawingClipboard: pid - tid -
	spwndClipOpen: 0x0, spwndClipViewer: 6772 rdpclip.exe
	cNumClipFormats: 4, iClipSerialNumber: 9
	pClipBase: 0xe6fe8ec8, Formats: CF_UNICODETEXT,CF_LOCALE,CF_TEXT,CF_OEMTEXT
	[snip]
	```
	- atom table at `0xe7981648`
	- contains 3 desktops
	- *rdpclip.exe* is currently viewing clipboard
	- 4 supported clipboard formats including Unicode and ASCII
	- 9 items have been copied to clipboard so far

## 13. clipboard snooping
- clipboard snooping can be done by:
	- hooking `SetClipboardData` and stealing data as it is placed in the clipboard; easily detected
	- calling `GetClipboardData` at regular intervals; because only one process can access clipboard at a time, could cause race condition

## 14. clipboard viewers and listeners
- MS recommended way to access data as soon as it is copied to clipboard is registering clipboard viewer (via `SetClipboardViewer`) or format listener (via `WM_DRAWCLIPBOARD` messages)
- applications using clipboard APIs become owners of `CLIPBRDWNDCLASS`; can be used to identify processes that can perform clipboard operations
	``` sh
	$ python vol.py -f memory.dmp --profile=Win7SP1x86 wintree | grep CLIPBRDWNDCLASS
	Volatility Foundation Volatility Framework 2.4
	.#10062 explorer.exe:372 CLIPBRDWNDCLASS
	.#100f0 explorer.exe:372 CLIPBRDWNDCLASS
	.#1011e vmtoolsd.exe:2224 CLIPBRDWNDCLASS
	.#1014a SnagIt32.exe:2300 CLIPBRDWNDCLASS
	```

## 15. desktops analysis objectives
- finding hidden desktops
- detecting ransomware: malicious desktops can lock users out of their system
- finding hidden threads: GUI subsystem defines `tagTHREADINFO` structure independent of `_ETHREAD`, containing information about thread

## 16. desktops DS
- `tagDESKTOP`:
	``` python
	>>> dt("tagDESKTOP")
	'tagDESKTOP' (224 bytes)
	0x0 : dwSessionId ['unsigned long']
	0x8 : pDeskInfo ['pointer64', ['tagDESKTOPINFO']]
	0x10 : pDispInfo ['pointer64', ['tagDISPLAYINFO']]
	0x18 : rpdeskNext ['pointer64', ['tagDESKTOP']]
	0x20 : rpwinstaParent ['pointer64', ['tagWINDOWSTATION']]
	0x28 : dwDTFlags ['unsigned long']
	0x30 : dwDesktopId ['unsigned long long']
	0x38 : spmenuSys ['pointer64', ['tagMENU']]
	0x40 : spmenuDialogSys ['pointer64', ['tagMENU']]
	0x48 : spmenuHScroll ['pointer64', ['tagMENU']]
	0x50 : spmenuVScroll ['pointer64', ['tagMENU']]
	0x58 : spwndForeground ['pointer64', ['tagWND']]
	0x60 : spwndTray ['pointer64', ['tagWND']]
	0x68 : spwndMessage ['pointer64', ['tagWND']]
	0x70 : spwndTooltip ['pointer64', ['tagWND']]
	0x78 : hsectionDesktop ['pointer64', ['void']]
	0x80 : pheapDesktop ['pointer64', ['tagWIN32HEAP']]
	0x88 : ulHeapSize ['unsigned long']
	0x90 : cciConsole ['_CONSOLE_CARET_INFO']
	0xa8 : PtiList ['_LIST_ENTRY']
	0xb8 : spwndTrack ['pointer64', ['tagWND']]
	0xc0 : htEx ['long']
	0xc4 : rcMouseHover ['tagRECT']
	0xd4 : dwMouseHoverTime ['unsigned long']
	0xd8 : pMagInputTransform ['pointer64', ['_MAGNIFICATION_INPUT_TRANSFORM']]
	```
	- `dwSessionId`: associates desktop to owning session
	- `pDeskInfo`: points to `tagDESKTOPINFO`, where information on desktop global hooks is stored
	- `rpdeskNext`: enumerates desktops in same window station
	- `rpwinstaParent`: identifies window station to which desktop belongs
	- `pheapDesktop`: points to desktop heap
	- `PtiList`: list of `tagTHREADINFO` structures, one for each thread attached to desktop

## 17. `deskscan` plugin
- example output:
	``` data
	$ python vol.py -f rdp.mem --profile=Win2003SP2x86 deskscan
	Volatility Foundation Volatility Framework 2.4
	**************************************************
	Desktop: 0x8001038, Name: WinSta0\Default, Next: 0x8737bc10
	SessionId: 2, DesktopInfo: 0xbc6f0650, fsHooks: 2128
	spwnd: 0xbc6f06e8, Windows: 238
	Heap: 0xbc6f0000, Size: 0x300000, Base: 0xbc6f0000, Limit: 0xbc9f0000
	7808 (notepad.exe 6236 parent 5544)
	7760 (csrss.exe 7888 parent 432)
	5116 (csrss.exe 7888 parent 432)
	8168 (PccNTMon.exe 5812 parent 5132)
	3040 (cmd.exe 5544 parent 5132)
	6600 (csrss.exe 7888 parent 432)
	7392 (explorer.exe 5132 parent 8120)
	5472 (explorer.exe 5132 parent 8120)
	548 (PccNTMon.exe 5812 parent 5132)
	6804 (mbamgui.exe 5220 parent 5132)
	2008 (ctfmon.exe 4576 parent 5132)
	3680 (PccNTMon.exe 5812 parent 5132)
	2988 (VMwareTray.exe 3552 parent 5132)
	1120 (explorer.exe 5132 parent 8120)
	4500 (explorer.exe 5132 parent 8120)
	7732 (explorer.exe 5132 parent 8120)
	6836 (explorer.exe 5132 parent 8120)
	7680 (winlogon.exe 3272 parent 432)
	7128 (rdpclip.exe 6772 parent 3272)
	5308 (rdpclip.exe 6772 parent 3272)
	**************************************************
	Desktop: 0x737bc10, Name: WinSta0\Disconnect, Next: 0x8a2f2068
	SessionId: 2, DesktopInfo: 0xbc6e0650, fsHooks: 0
	spwnd: 0xbc6e06e8, Windows: 25
	Heap: 0xbc6e0000, Size: 0x10000, Base: 0xbc6e0000, Limit: 0xbc6f0000
	**************************************************
	Desktop: 0xa2f2068, Name: WinSta0\Winlogon, Next: 0x0
	SessionId: 2, DesktopInfo: 0xbc6c0650, fsHooks: 0
	spwnd: 0xbc6c06e8, Windows: 6
	Heap: 0xbc6c0000, Size: 0x20000, Base: 0xbc6c0000, Limit: 0xbc6e0000
	6912 (winlogon.exe 3272 parent 432)
	1188 (winlogon.exe 3272 parent 432)
	8172 (winlogon.exe 3272 parent 432)
	**************************************************
	[snip]
	```
	- `Winlogon` presents login prompt; swithces to `Default` if successful
	- *explorer.exe* runs in `Default` as expected
	- `Default` heap size much larger to allow creation of many windows and other objects
	- only threads in `Winlogon` belong to *winlogon.exe*; other threads may indicate attempt to steal login credentials
	- `Default` has non-zero `fsHooks`; threads within desktop can intercept window messages (e.g. keystrokes) before they reach target window

## 18. alternate desktop
- windows and window messages do not cross desktop boundaries; running from alternate desktop hides process from other desktops
- *Internet Explorer* running on different desktop:
	![[IE_alternate_desktop.png]]
	- not shown in *Winlister* output
- switching desktops in Windows is not easy; involves writing C program that calls `SwitchDesktop` with name of desktop

## 19. Tigger case study
- steps:
	1. create new desktop named `system_temp_`
	2. create temporary filename to capture redirected output of console commands
	3. set `STARTUPINFO.lpDesktop` to `WinSta0\system_temp_`
	4. set `dwFlags` to indicate `STARTUPINFO` structure has preferences for window visibility, standard output, and standard error handles for process to be created
	5. create new process; path to process `szCmd` is passed as an argument (originally taken from attacker over network)
	6. wait for process to complete
	7. close `system_temp_` desktop
	8. read process' output
- code:
	``` C
	// can be found in book
	```
- allows Tigger to execute any application without alerting AVs that monitor only `Default` desktop

## 20. atom
- strings that can be shared between processes in same session
- added by passing string to function such as `AddAtom` or `GlobalAddAtom`; returns integer identifier that can be used to retrieve string
- atom table is a hash bucket that contains integer to string mappings
- many API functions create atoms implicitly; malware authors are not aware and do not cover up tracks

## 21. atom table analysis objectives
- detect window class names: when applications use `RegisterClassEx` to register a new window class, class name is added to global atom table
- detect window message names: when applications register window messages via `RegisterWindowMessage`, name is added to global atom table
- identify injected DLLs: `SetWindowsHookEx` and `SetWinEventHook` add path to DLL containing hook function to global atom table
- detect system presence marking: some malware families use atoms to mark system as infected

## 22. atom table DS
- atom tables are represented as `_RTL_ATOM_TABLE` structures
- structure format is different for user mode and kernel mode
- when analyzing session atom table (`win32k!UserAtomTableHandle`) and window station atom table (`tagWINDOWSTATION.pGlobalAtomTable`), use kernel mode definition
- `_RTL_ATOM_TABLE`: 
	``` python
	>>> dt("_RTL_ATOM_TABLE")
	'_RTL_ATOM_TABLE' (112 bytes)
	0x0 : Signature ['unsigned long']
	0x8 : CriticalSection ['_RTL_CRITICAL_SECTION']
	0x18 : NumBuckets ['unsigned long']
	0x20 : Buckets ['array', <function>, ['pointer', ['_RTL_ATOM_TABLE_ENTRY']]]
	```
	- `Signature`: 
		- `0x6d6f7441`, Atom
		- structures exist in pools with tag "AtmT"
	- `NumBuckets`: specifies number of `_RTL_ATOM_TABLE_ENTRY` structures in `Buckets` array

- `_RTL_ATOM_TABLE_ENTRY`:
	``` python
	>>> dt("_RTL_ATOM_TABLE_ENTRY")
	'_RTL_ATOM_TABLE_ENTRY' (24 bytes)
	0x0 : HashLink ['pointer64', ['_RTL_ATOM_TABLE_ENTRY']]
	0x8 : HandleIndex ['unsigned short']
	0xa : Atom ['unsigned short']
	0xc : ReferenceCount ['unsigned short']
	0xe : Flags ['unsigned char']
	0xf : NameLength ['unsigned char']
	0x10 : Name ['String', {'length': <function>, 'encoding': 'utf16'}]
	```
	- `HashLink`: used to enumerate all atom entries in bucket
	- `Atom`:
		- integer identifier for atom table entry
		- value returned by functions such as `AddAtom` and `FindAtom`
		- serves as index into atom table to particular atom
	- `ReferenceCount`: incremented when specified string is added to atom table, decremented when atom is deleted
	- `Name`: 
		- string name of atom
		- primary analysis point

## 23. `atomscan` plugin
- scans memory and reports in order found
- can sort output by atom ID with `--sort-by=atom`
- can sort output by reference cout with `--sort-by=refcount`
- example output:
	``` data
	$ python vol.py -f win7x64.dd --profile=Win7SP1x64 atomscan --sort-by=refcount
	Volatility Foundation Volatility Framework 2.4
	AtomOfs(V) Atom Refs Pinned Name
	------------------ ------------ ------ ------ ----
	0xfffff8a007b22b80 0xc125 8 0 Micro[snip].IEXPLORER_EXITING
	0xfffff8a005dba520 0xc0be 8 0 Net Resource
	0xfffff8a007b1b480 0xc124 8 0 Micro[snip].SET_CONNECTOID_NAME
	0xfffff8a007b21440 0xc123 8 0 Micro[snip].WINSOCK_ACTIVITY
	0xfffff8a0098594b0 0xc15b 8 0 ReaderModeCtl
	0xfffff8a007a62c50 0xc102 8 0 C:\Windows\system32\MsftEdit.dll
	0xfffff8a0076f5fe0 0xc0e2 10 0 WorkerW
	0xfffff8a005d51250 0xc0c9 10 0 UniformResourceLocatorW
	0xfffff8a0073a6d60 0xc0e9 10 0 CLIPBRDWNDCLASS
	0xfffff8a0081bd640 0xc1af 11 0 WM_HTML_GETOBJECT
	0xfffff8a007baa5d0 0xc135 11 0 C:\Windows\system32\fxsst.dll
	0xfffff8a001ae9200 0xc164 11 0 CMBIgnoreNextDeselect
	0xfffff8a0071eb9b0 0xc1d1 12 0 C:\Windows\[snip]\ShFusRes.dll
	0xfffff8a006a79850 0xc08b 12 0 FileNameMapW
	0xfffff8a0083155a0 0xc1d9 13 0 C:\Program[snip] \windbg.exe
	0xfffff8a0087cb880 0xc1f6 16 0 text/css
	[snip]
	```
	- DLL paths, registered window messages (`WM_HTML_GETOBJECT`), class names, data types, etc 

## 24. atom table usage
- detecting empty window class names:
	- using `WNDCLASSEX` passed when calling `RegisterClassEx`, *win32k.sys* creates an atom from the `lpszClassName` member of this structure; malware may pass blank names to be stealty:
		![[empty_class_name.png]]
	- `atomscan` can show blank atom names which are suspicious:
		``` data
		$ python vol.py -f mutihack.vmem --profile=WinXPSP3x86 atomscan
		Volatility Foundation Volatility Framework 2.4
		AtomOfs(V) Atom Refs Pinned Name
		---------- ---------- ------ ------ ----
		[snip]
		0xe17c34b8 0xc0c4 2 0 UnityAppbarWindowClass
		0xe17c7678 0xc006 1 1 FileName
		0xe17d40a0 0xc0ff 2 0
		0xe17d4128 0xc027 1 1 SysCH
		[snip]
		```
- detecting registered window messages:
	- `lpString` parameter passed to `RegisterWindowMessage` is used to create an atom
	- Clod malware registers `WM_HTML_GETOBJECT` window message to obtain `IHTMLDocument2` interface from an `HWND` (handle to window):
		![[Clod_windows_message.png]]
- identifying infected systems:
	- malware may use atoms instead of mutexes to mark presence on system
	- Tigger uses atoms to mark presence:
		![[Tigger_atom_presence.png]]
	- Tigger uses atoms for synchronization:
		![[Tigger_atom_sync.png]]

## 25. windows
