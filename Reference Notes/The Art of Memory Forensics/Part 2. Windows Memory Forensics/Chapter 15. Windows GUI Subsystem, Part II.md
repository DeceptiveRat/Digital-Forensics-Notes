## 1. window message hooks
- message hooks can intercept the message before it reaches the target window procedure
- e.g. attackers can log `WM_KEYDOWN` events then pass them on 
- nonhooked and hooked message systems:
	![[nonhooked_window_message.png]]
	![[hooked_window_message.png]]

## 2. message hook installation
- `SetWindowsHookEx` function prototype:
	``` C
	HHOOK WINAPI SetWindowsHookEx(
	_In_ int idHook,
	_In_ HOOKPROC lpfn,
	_In_ HINSTANCE hMod,
	_In_ DWORD dwThreadId
	);
	```
	- `idHook`: `WH_` constants specifying type of message
	- `lpfn`: address of hook procedure
	- `hMod`: 
		- handle for DLL containing hook procedure
		- loads into address space of thread that owns target window
	- `dwThreadId`:
		- thread ID for hook
		- specific thread ID or 0 to affect all threads in current desktop

## 3. message hook analysis objectives
- detect global hooks: most malicious message hooks are global
- DLL attribution: trace hook back to owning DLL on disk
- hook function analysis

## 4. message hook DS
- `tagHOOK`:
	``` python
	>>> dt("tagHOOK")
	'tagHOOK' (96 bytes)
	0x0 : head ['_THRDESKHEAD']
	0x28 : phkNext ['pointer64', ['tagHOOK']]
	0x30 : iHook ['long']
	0x38 : offPfn ['unsigned long long']
	0x40 : flags ['Flags', {'bitmap': {'HF_INCHECKWHF': 8, 'HF_HOOKFAULTED': 4, 'HF_WX86KNOWNDLL': 6, 'HF_HUNG': 3, 'HF_FREED': 9, 'HF_ANSI': 1, 'HF_GLOBAL': 0, 'HF_DESTROYED': 7}}]
	0x44 : ihmod ['long']
	0x48 : ptiHooked ['pointer64', ['tagTHREADINFO']]
	0x50 : rpdesk ['pointer64', ['tagDESKTOP']]
	0x58 : fLastHookHung ['BitField', {'end_bit': 8, 'start_bit': 7, 'native_type': 'long'}]
	0x58 : nTimeout ['BitField', {'end_bit': 7, 'start_bit': 0, 'native_type': 'unsigned long'}]
	```
	- `head`:
		- header for USER objects
		- can help identify owning process/thread
	- `phkNext`: 
		- pointer to next hook in chain
		- used when `CallNextHookEx` is called
	- `offPfn`: RVA to hook procedure
	- `ihmod`:
		- -1 if procedure in same module as code calling `SetWindowsHookEx`
		- index into array of atoms in `win32k!_aatomSysLoaded` if procedure in DLL
		- translate `ihmod` into atom and obtain atom name to determine DLL name
	- `ptiHooked`: used to identify hooked thread
	- `rpdesk`: identifies desktop in which hook is set

## 5. `messagehooks` plugin
- enumerates global hooks by finding all desktops and accessing `tagDESKTOP.pDeskInfo.aphkStart`, an array of `tagHOOK` structures whose positions indicate type of message
- `tagDESKTOP.pDeskInfo.fsHooks` is bitmap showing which elements in array used
- for each thread, scan for local hooks by looking at `tagTHREADINFO.aphkStart` and `tagTHREADINFO.fsHooks`
- output example:
	``` data
	$ python vol.py -f laqma.vmem --profile=WinXPSP3x86 messagehooks --output=block
	Volatility Foundation Volatility Framework 2.4
	Offset(V) : 0xbc693988
	Session : 0
	Desktop : WinSta0\Default
	Thread : <any>
	Filter : WH_GETMESSAGE
	Flags : HF_ANSI, HF_GLOBAL
	Procedure : 0x1fd9
	ihmod : 1
	Module : C:\WINDOWS\system32\Dll.dll
	Offset(V) : 0xbc693988
	Session : 0
	Desktop : WinSta0\Default
	Thread : 1584 (explorer.exe 1624)
	Filter : WH_GETMESSAGE
	Flags : HF_ANSI, HF_GLOBAL
	Procedure : 0x1fd9
	ihmod : 1
	Module : C:\WINDOWS\system32\Dll.dll
	Offset(V) : 0xbc693988
	Session : 0
	Desktop : WinSta0\Default
	Thread : 252 (VMwareUser.exe 1768)
	Filter : WH_GETMESSAGE
	Flags : HF_ANSI, HF_GLOBAL
	Procedure : 0x1fd9
	ihmod : 1
	Module : C:\WINDOWS\system32\Dll.dll
	[snip]
	```
	- hook procedure disassembly:
		``` python
		$ python vol.py -f laqma.vmem --profile=WinXPSP3x86 dlllist -p 1624 | grep Dll.dll
		Volatility Foundation Volatility Framework 2.4
		0x00ac0000 0x8000 C:\Documents and Settings\Mal Ware\Desktop\Dll.dll
		$ python vol.py -f laqma.vmem --profile=WinXPSP3x86 volshell
		Volatility Foundation Volatility Framework 2.4
		Current context: process System, pid=4, ppid=0 DTB=0x31a000
		Welcome to volshell!
		To get help, type 'hh()'
		>>> cc(pid = 1624)
		Current context: process explorer.exe, pid=1624, ppid=1592 DTB=0x80001c0
		>>> dis(0x00ac0000 + 0x00001fd9)
		0xac1fd9 ff74240c PUSH DWORD [ESP+0xc]
		0xac1fdd ff74240c PUSH DWORD [ESP+0xc]
		0xac1fe1 ff74240c PUSH DWORD [ESP+0xc]
		0xac1fe5 ff350060ac00 PUSH DWORD [0xac6000]
		0xac1feb ff157c40ac00 CALL DWORD [0xac407c] ; CallNextHookEx
		0xac1ff1 c20c00 RET 0xc
		```
		- calls next hook rightaway; just used to load DLL
	
## 6. USER handle
- references to objects in the GUI subsystem
- USER handle tables provide an alternate method of finding evidence
- there is one USER handle table per session, and all processes within share it; does not mean any process can access all objects

## 7. USER handle analysis objectives
- verification and cross-reference

## 8. USER handle DS
- `win32k!_gahti` is an array of `tagHANDLETYPEINFO` structures, one for each object type
- `tagHANDLETYPEINFO`:
	``` python
	>>> dt("tagHANDLETYPEINFO")
	'tagHANDLETYPEINFO' (16 bytes)
	0x0 : fnDestroy ['pointer', ['void']]
	0x8 : dwAllocTag ['String', {'length': 4}]
	0xc : bObjectCreateFlags ['Flags', {'target': 'unsigned char', 'bitmap': {'OCF_VARIABLESIZE': 7, 'OCF_DESKTOPHEAP': 4, 'OCF_THREADOWNED': 0, 'OCF_SHAREDHEAP': 6, 'OCF_USEPOOLIFNODESKTOP': 5, 'OCF_USEPOOLQUOTA': 3, 'OCF_MARKPROCESS': 2, 'OCF_PROCESSOWNED': 1}}
	```
	- `fnDestroy`: pointer to default deallocation function for object type
	- `dwAllocTag`:
		- 4 ASCII characters that directly precede objects in memory
		- can be used to identify allocations
	- `bObjectCreateFlags`: tell whether an object is thread-owned or process-owned, whether it is allocated from shared or desktop heap, etc

## 9. `gahti` plugin
- *Global Array of Handle Table Information(GAHTI)*
- parses `win32k!_gahti` to show types of objects on OS
- first entry of GAHTI is always `TYPE_FREE`:
	- members are all 0
	- handles no longer in use are set to `TYPE_FREE`, instead of being immediately removed
- example output:
	``` data
	$ python vol.py -f win7x64cmd.dd --profile=Win7SP1x64 gahti
	Volatility Foundation Volatility Framework 2.4
	Session Type Tag fnDestroy Flags
	-------- -------------------- -------- ------------------ -----
	0 TYPE_FREE 0x0000000000000000
	0 TYPE_WINDOW Uswd 0xfffff9600014f660 OCF_DESKTOPHEAP, OCF_THREADOWNED, OCF_USEPOOLIFNODESKTOP, OCF_USEPOOLQUOTA
	0 TYPE_MENU 0xfffff960001515ac OCF_DESKTOPHEAP, OCF_PROCESSOWNED
	0 TYPE_CURSOR Uscu 0xfffff960001541a0 OCF_MARKPROCESS, OCF_PROCESSOWNED, OCF_USEPOOLQUOTA
	0 TYPE_SETWINDOWPOS Ussw 0xfffff960001192b4 OCF_THREADOWNED, OCF_USEPOOLQUOTA
	0 TYPE_HOOK 0xfffff9600018e5c8 OCF_DESKTOPHEAP, OCF_THREADOWNED
	0 TYPE_CLIPDATA Uscb 0xfffff9600017c5ac
	[snip]
	0 TYPE_WINEVENTHOOK Uswe 0xfffff9600018f148 OCF_THREADOWNED
	0 TYPE_TIMER Ustm 0xfffff960001046dc OCF_PROCESSOWNED
	0 TYPE_INPUTCONTEXT Usim 0xfffff9600014c660 OCF_DESKTOPHEAP, OCF_THREADOWNED
	0 TYPE_HIDDATA Usha 0xfffff960001d2a34 OCF_THREADOWNED
	```

## 10. `tagSHAREDINFO` DS
- `win32k!_gSharedInfo` symbol:
	- points to `tagSHAREDINFO` structure, which identifies location of session's USER handle table, a map to all USER objects in use
	- located in *win32k.sys* kernel module data section:
		![[gSharedInfo_variable.png]]
- `tagSHAREDINFO`:
	``` python
	>>> dt("tagSHAREDINFO")
	'tagSHAREDINFO' (568 bytes)
	0x0 : psi ['pointer64', ['tagSERVERINFO']]
	0x8 : aheList ['pointer64', ['_HANDLEENTRY']]
	0x10 : HeEntrySize ['unsigned long']
	0x18 : pDispInfo ['pointer64', ['tagDISPLAYINFO']]
	0x20 : ulSharedDelta ['unsigned long long']
	0x28 : awmControl ['array', 31, ['_WNDMSG']]
	0x218 : DefWindowMsgs ['_WNDMSG']
	0x228 : DefWindowSpecMsgs ['_WNDMSG']
	```
	- `psi`: points to valid `tagSERVERINFO` structure
	- `aheList`:
		- points to an array of `_HANDLEENTRY` structures, one for each handle in table
		- number of handles can be found at `tagSHAREDINFO.psi.cHandleEntries`
	- `HeEntrySize`: size of `_HANDLEENTRY` for current platform
	- `ulSharedDelta`: delta user-mode processes can use to determine location of USER objects in kernel memory

## 11. finding `win32k!_gSharedInfo`
1. determine base address of *sinewk.sys* mapped into session space
2. locate data PE section:
	- if PE header is corrupt/paged, brute force search full length of *win32k.sys*
3. iterate over data on 4-byte boundary:
	- instantiate `tagSHAREDINFO` object at each address
	- call `is_valid` method for sanity checks

## 12. handle table entries
- `_HANDLEENTRY`:
	``` python
	>>> dt("_HANDLEENTRY")
	'_HANDLEENTRY' (24 bytes)
	0x0 : phead ['pointer64', ['_HEAD']]
	0x8 : pOwner ['pointer64', ['void']]
	0x10 : bType ['Enumeration', {'target': 'unsigned char', 'choices': {0: 'TYPE_FREE', 1: 'TYPE_WINDOW', 2: 'TYPE_MENU', 3: 'TYPE_CURSOR', 4: 'TYPE_SETWINDOWPOS', 5: [snip]
	0x11 : bFlags ['unsigned char']
	0x12 : wUniq ['unsigned short']
	```
	- `phead`: pointer to common header
	- `bType`: identifies type of object
- `_HEAD`: 
	``` python
	>>> dt("_HEAD")
	'_HEAD' (16 bytes)
	0x0 : h ['pointer64', ['void']]
	0x8 : cLockObj ['unsigned long']
	>>> dt("_THRDESKHEAD")
	'_THRDESKHEAD' (40 bytes)
	0x0 : h ['pointer64', ['void']]
	0x8 : cLockObj ['unsigned long']
	0x10 : pti ['pointer64', ['tagTHREADINFO']]
	0x18 : rpdesk ['pointer64', ['tagDESKTOP']]
	0x20 : pSelf ['pointer64', ['unsigned char']]
	>>> dt("_PROCDESKHEAD")
	'_PROCDESKHEAD' (40 bytes)
	0x0 : h ['pointer64', ['void']]
	0x8 : cLockObj ['unsigned long']
	0x10 : hTaskWow ['unsigned long']
	0x18 : rpdesk ['pointer64', ['tagDESKTOP']]
	0x20 : pSelf ['pointer64', ['unsigned char']]
	```
	- `_THRDESKHEAD.pti.pEThread` which points to `_ETHREAD` can be used to identify owning thread
	- `_HANDLEENTRY.pOwner` which points to owning `tagPROCESSINFO`, where `Process` member can be used to identify `_EPROCESS`

## 13. `userhandles` plugin
- locates shared information structure, walks handle table
- example output:
	``` data
	$ python vol.py -f win7x64.dd --profile=Win7SP1x64 userhandles
	Volatility Foundation Volatility Framework 2.4
	**************************************************
	SharedInfo: 0xfffff9600035d300, SessionId: 0
	aheList: 0xfffff900c0400000, Table size: 0x2000, Entry size: 0x18
	Object(V) Handle bType Flags Thread Process
	------------------ ------------ --------------- -------- -------- -------
	0xfffff900c05824b0 0x10001 TYPE_MONITOR 0 -------- -
	0xfffff900c01bad20 0x10002 TYPE_WINDOW 64 432 316
	0xfffff900c00b6730 0x10003 TYPE_CURSOR 0 -------- 316
	0xfffff900c0390b90 0x10004 TYPE_WINDOW 0 432 316
	0xfffff900c00d7ab0 0x10005 TYPE_CURSOR 0 -------- 316
	0xfffff900c0390e60 0x10006 TYPE_WINDOW 0 432 316
	0xfffff900c00d7640 0x10007 TYPE_CURSOR 0 -------- 316
	[snip]
	```

## 14. event hook
- used to receive notification when UI-related events occur
- e.g. *Windows Explorer* fires events when soudns play or scroll operation begins
- can be used to generically  load a DLL into any process

## 15. event hook analysis objectives
- determine event hook scope: RAM artifacts tell which threads/processes are affected and specific event being monitored
- analyze intent: disassemble hook functions

## 16. event hook DS
- `tagEVENTHOOK`:
	``` python
	>>> dt("tagEVENTHOOK")
	'tagEVENTHOOK' (None bytes)
	0x18 : phkNext ['pointer', ['tagEVENTHOOK']]
	0x20 : eventMin ['Enumeration', {'target': 'unsigned long', 'choices': {1: 'EVENT_MIN', 2: 'EVENT_SYSTEM_ALERT', [snip]
	0x24 : eventMax ['Enumeration', {'target': 'unsigned long', 'choices': {1: 'EVENT_MIN', 2: 'EVENT_SYSTEM_ALERT', [snip]
	0x28 : dwFlags ['unsigned long']
	0x2c : idProcess ['unsigned long']
	0x30 : idThread ['unsigned long']
	0x40 : offPfn ['unsigned long long']
	0x48 : ihmod ['long']
	```
	- `phkNext`: pointer to next hook
	- `eventMin`: lowest system event hook applies to 
	- `eventMax`: highst system event hook applies to 
	- `dwFlags`: 
		- tells whether process generating event will load DLL containing hook procedure (`WINEVENT_INCONTEXT`)
		- tells whether thread installing hook wants to be exempt from hook (`WINEVENT_SKIPOWNPROCESS` and `WINEVENT_SKIPOWNTHREAD`)
	- `idProcess`: PID of target, or 0 for all processes
	- `idThread`: TID of target, or 0 for all threads
	- `offPfn`: RVA into hook procedure in DLL
	- `ihmod`: index into `win32k!_aatomSysLoaded` array, which can be used to identify full path to DLL containing hook
- event hooks are installed using `SetWinEventHook`:
	``` C
	HWINEVENTHOOK WINAPI SetWinEventHook(
	_In_ UINT eventMin,
	_In_ UINT eventMax,
	_In_ HMODULE hmodWinEventProc,
	_In_ WINEVENTPROC lpfnWinEventProc,
	_In_ DWORD idProcess,
	_In_ DWORD idThread,
	_In_ UINT dwflags
	);
	```

## 17. `eventhooks` plugin
- example output:
	``` data
	$ python vol.py -f win7x64.dd --profile=Win7SP1x64 eventhooks
	Volatility Foundation Volatility Framework 2.4
	Handle: 0x300cb, Object: 0xfffff900c01eda10, Session: 1
	Type: TYPE_WINEVENTHOOK, Flags: 0, Thread: 1516, Process: 880
	eventMin: 0x4 EVENT_SYSTEM_MENUSTART
	eventMax: 0x7 EVENT_SYSTEM_MENUPOPUPEND
	Flags: , offPfn: 0xff567cc4, idProcess: 0, idThread: 0
	ihmod: -1
	```
- malicious event hooks generally are global, with hook procedures in attacker-provided DLL

## 18. windows clipboard analysis objectives
- password recovery: clipboard can contain sensitive information
- copied files artifacts: exfiltration often involves copying from *Windows Explorer* to FTP directory

## 19. windows clipboard DS
- `tagCLIP` specifies clipboard format and contains handle to associated `tagCLIPDATA`; address of `tagCLIPDATA` object must be separately obtained and matched with handle value
- easiest way to locate `tagCLIPDATA` objects is walking session handle tables and filtering for `TYPE_CLIPDATA`
- `tagCLIP`:
	``` python
	>>> dt("tagCLIP")
	'tagCLIP' (24 bytes)
	0x0 : fmt ['Enumeration', {'target': 'unsigned
		long', 'choices': {128: 'CF_OWNERDISPLAY', 1: 'CF_TEXT', 2: 'CF_BITMAP', 3:
		'CF_METAFILEPICT', 4: 'CF_SYLK', 5: 'CF_DIF', 6: 'CF_TIFF', 7: 'CF_OEMTEXT', 8:
		'CF_DIB', 9: 'CF_PALETTE', 10: 'CF_PENDATA', 11: 'CF_RIFF', 12: 'CF_WAVE', 13:
		'CF_UNICODETEXT', 14: 'CF_ENHMETAFILE', 15: 'CF_HDROP', 16: 'CF_LOCALE', 17: 'CF_
		DIBV5', 131: 'CF_DSPMETAFILEPICT', 129: 'CF_DSPTEXT', 130: 'CF_DSPBITMAP', 142:
		'CF_DSPENHMETAFILE'}}]
	0x8 : hData ['pointer64', ['void']]
	0x10 : fGlobalHandle ['long']
	```
	- `fmt`:
		- specifies clipboard format
		- applications can create new formats with `RegisterClipboardFormat`
	- `hData`: 
		- handle to associated `tagCLIPDATA` object
		- can be 1 for `DUMMY_TEXT_HANDLE`, 2 for `DUMMY_DIB_HANDLE`, or 0 for certain deferred operations
- `tagCLIPDATA`:
	``` python
	>>> dt("tagCLIPDATA")
	'tagCLIPDATA' (None bytes)
	0x10 : cbData ['unsigned int']
	0x14 : abData ['array', <function <lambda> at 0x1048e5500>, ['unsigned char']]
	```
	- `abData`: array of bytes containing clipboard data
	- `cbData`: length of `abData`

## 20. clipboard extraction
1. enumerate unique `_MM_SESSION_SPACE` structures
2. find `tagSHAREDINFO` for each session and walk USER handle table, collecting all `TYPE_CLIPDATA` objects
3. scan RAM for window station objects and enumerate `tagCLIP` structures from `tagWINDOWSTATION.pClipBase`
4. associate `tagCLIP.hData` handle values with `tagCLIPDATA`
5. cycle through remaining `tagCLIPDATA` objects not associated with `tagCLIP`

## 21. `clipboard` plugin
- example output:
	``` data
	$ python vol.py -f dfrws2008-rodeo-memory.img --profile=WinXPSP2x86 clipboard
	Volatility Foundation Volatility Framework 2.4
	Session WindowStation Format Handle Object Data
	------- ------------- ---------------- ---------- ---------- ------------
	0 WinSta0 CF_UNICODETEXT 0x4900c3 0xe12a7c98 pp -B -p -o out.pl file
	0 WinSta0 CF_LOCALE 0x80043 0xe12362d0
	0 WinSta0 CF_TEXT 0x1 ----------
	0 WinSta0 CF_OEMTEXT 0x1 ----------
	```
	- user in session `0\WinSta0` placed Unicode string `pp -B -p -o out.pl file`; seems to be command to run a *Perl* script
	- `CF_LOCALE` format not shown because it is binary; can be viewed with `-v/--verbose` option
	- no data for `CF_TEXT` or `CF_OEMTEXT` because handle value is 1
- file copied to clipboard:
	``` data
	$ python vol.py -f xpsp3.vmem --profile=WinXPSP3x86 clipboard -v
	Volatility Foundation Volatility Framework 2.4
	[snip]
	0 WinSta0 CF_HDROP 0x10230131 0xe1fa6590
	0xe1fa659c 14 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ................
	0xe1fa65ac 01 00 00 00 43 00 3a 00 5c 00 44 00 6f 00 63 00 ....C.:.\.D.o.c.
	0xe1fa65bc 75 00 6d 00 65 00 6e 00 74 00 73 00 20 00 61 00 u.m.e.n.t.s...a.
	0xe1fa65cc 6e 00 64 00 20 00 53 00 65 00 74 00 74 00 69 00 n.d...S.e.t.t.i.
	0xe1fa65dc 6e 00 67 00 73 00 5c 00 41 00 64 00 6d 00 69 00 n.g.s.\.A.d.m.i.
	0xe1fa65ec 6e 00 69 00 73 00 74 00 72 00 61 00 74 00 6f 00 n.i.s.t.r.a.t.o.
	0xe1fa65fc 72 00 5c 00 44 00 65 00 73 00 6b 00 74 00 6f 00 r.\.D.e.s.k.t.o.
	0xe1fa660c 70 00 5c 00 6e 00 6f 00 74 00 65 00 2e 00 74 00 p.\.n.o.t.e...t.
	0xe1fa661c 78 00 74 00 00 00 00 00 x.t.....
	```
	- object of `CF_HDROP` created with full path of file 

## 22. *Anti Cyber Crime Department of Federal Internet Security Agency(ACCDFISA)* ransomware case study
- ACCDFISA creates new desktop to display ransom note and locks user out of system 
- ransom note:
	![[ACCDFISA_ransom_note.png]]
- malware code disassembly:
	![[ACCDFISA_disassembled.png]]
	- `CreateDesktopA` to create new desktop
	- `SwitchDesktop` to switch
- `deskscan` plugin output:
	``` data
	$ python vol.py –f ACCFISA.vmem --profile=WinXPSP3x86 deskscan
	Volatility Foundation Volatility Framework 2.4
	[snip]
	**************************************************
	Desktop: 0x24675c0, Name: WinSta0\My Desktop 2, Next: 0x820a47d8
	SessionId: 0, DesktopInfo: 0xbc310650, fsHooks: 0
	spwnd: 0xbc3106e8, Windows: 111
	Heap: 0xbc310000, Size: 0x300000, Base: 0xbc310000, Limit: 0xbc610000
	652 (csrss.exe 612 parent 564)
	648 (csrss.exe 612 parent 564)
	308 (svchost.exe 300 parent 240)
	```
	- only thread in desktop (besides *csrss.exe*) is from *svchost.exe*
- `wintree` plugin output:
	``` data
	$ python vol.py -f ACCFISA.vmem --profile=WinXPSP3x86 wintree
	Volatility Foundation Volatility Framework 2.4
	[snip]
	**************************************************
	Window context: 0\WinSta0\My Desktop 2
	[snip]
	.#100e2 csrss.exe:612 -
	.#100e4 csrss.exe:612 -
	#100de (visible) csrss.exe:612 -
	.Anti-Child Porn Spam Protection (18 U.S.C. ? 2252) (visible) svchost.exe:300
	WindowClass_0
	..Send Code (visible) svchost.exe:300 Button
	..#100ee (visible) svchost.exe:300 Edit
	..Your Id #: 1074470467 Our special service email: security11220@gmail.com
	(visible) svchost.exe:300 Static
	..Your ID Number and our contacts (please write down this data): (visible)
	svchost.exe:300 Static
	..#100e8 (visible) svchost.exe:300 Static
	[snip]
	```
	- all windows for ransom message are owned by *svchost.exe*
- `dlllist` plugin output:
	``` data
	$ python vol.py -f ACCFISA.vmem --profile=WinXPSP3x86 dlllist -p 300
	Volatility Foundation Volatility Framework 2.4
	************************************************************************
	svchost.exe pid: 300
	Command line : "C:\wnhsmlud\svchost.exe"
	Service Pack 3
	Base Size Path
	---------- ---------- ----
	0x00400000 0x2f000 C:\wnhsmlud\svchost.exe
	0x7c900000 0xb2000 C:\WINDOWS\system32\ntdll.dll
	0x7c800000 0xf6000 C:\WINDOWS\system32\kernel32.dll
	0x77c10000 0x58000 C:\WINDOWS\system32\MSVCRT.dll
	[snip]
	```
	- hosted out of `C:\wnhsmlud\`; not real *svchost.exe*
- `printkey` plugin output:
	``` data
	$ python vol.py -f ACCFISA.vmem --profile=WinXPSP3x86 printkey
	-K "Microsoft\Windows\CurrentVersion\Run"
	Volatility Foundation Volatility Framework 2.4
	Legend: (S) = Stable (V) = Volatile
	----------------------------
	[snip]
	REG_SZ svchost : (S) C:\wnhsmlud\svchost.exe
	```
	- persistency through registry confirmed
