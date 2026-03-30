## 1. `_EPROCESS`
![[basic process resources.png]]

- name of structure Windows uses to represnt a process
- each process has a private VMS containing:
	- list of loaded modules
	- stack
	- heap
	- allocated memory regions containing user input, app-specific DS, etc
- Windows organizes memory regions using *Virtual Address Descriptors(VAD)*
- points to list of SIDs and privilege data

## 2. `_EPROCESS` analysis objectives
- identify critical processes
- generate visualizations
- detect *Direct Kernel Object Manipulation(DKOM)*

## 3. `_EPROCESS` DS
``` python
>>> dt("_EPROCESS")
'_EPROCESS' (1232 bytes)
0x0 : 	Pcb 					['_KPROCESS']
0x160 : ProcessLock 			['_EX_PUSH_LOCK']
0x168 : CreateTime 				['WinTimeStamp', {'is_utc': True}]
0x170 : ExitTime 				['WinTimeStamp', {'is_utc': True}]
0x178 : RundownProtect 			['_EX_RUNDOWN_REF']
0x180 : UniqueProcessId 		['unsigned int']
0x188 : ActiveProcessLinks 		['_LIST_ENTRY']
[snip]
0x1e0 : SessionProcessLinks 	['_LIST_ENTRY']
[snip]
0x200 : ObjectTable 			['pointer64', ['_HANDLE_TABLE']]
0x208 : Token 					['_EX_FAST_REF']
[snip]
0x290 : InheritedFromUniqueProcessId ['unsigned int']
[snip]
0x2d8 : Session 				['pointer64', ['void']]
0x2e0 : ImageFileName 			['String', {'length': 16}]
[snip]
0x308 : ThreadListHead 			['_LIST_ENTRY']
0x318 : SecurityPort 			['pointer64', ['void']]
0x320 : Wow64Process 			['pointer64', ['void']]
0x328 : ActiveThreads 			['unsigned long']
[snip]
0x338 : Peb 					['pointer64', ['_PEB']]
[snip]
0x444 : ExitStatus 				['long']
0x448 : VadRoot 				['_MM_AVL_TABLE']
[snip]
```
- `Pcb`: 
	- kernel's *Process Control Block(PCB)*
	- contains several critical fields, including `DirectoryTableBase` for address translation and amount of time process spent in kernel/user mode
- `ActiveProcessLinks`: 
	- doubly linked list
	- chains active processes together
	- most APIs rely on this
- `SessionProcessLinks`: 
	- chains together processes in same session
- `InheritedFromUniqueProcessId`: 
	- PID of parent
	- remains even after parent terminates
- `Session`:
	- points to `_MM_SESSION_SPACE` that stores info on user's logon session and GUI objects
- `ImageFileName`:
	- file name of process executable
	- stores first 16 ASCII charcters
	- full path available through VAD node or members in *Process Environment Block(PEB)*
- `ThreadListHead`: 
	- chains together process threads; each element is `_ETHREAD`
- `ActiveThreads`: 
	- number of active threads
- `Peb`: 
	- pointer to PEB
	- points to an address in user mode
	- PEB contains:
		- pointers to process DLL list
		- current working directory
		- command line arguments
		- environment variables
		- heaps
		- standard handles
- `VadRoot`:
	- root node of VAD tree
	- contains info on:
		- allocated memory segments
		- original access permissions
		- whether a file is mapped into the region

## 4. `_LIST_ENTRY`
![[_LIST_ENTRY.png]]

- DS in `_EPROCESS`
- contains `Flink` to next `_EPROCESS` and `Blink` to previous `_EPROCESS`

## 5. critical system proceses
- `Idle` and `System`: 
	- have no corresponding executables
	- `Idle` is a container to charge CPU time for idle threads
	- `System` serves as default home for threads that run in kernel mode
	- `System`, pid 4, appears to own any sockets or handles to files that kernel modules open
- `csrss.exe`:
	- client/server runtime subsystem
	- plays a role in creating/deleting processes and threads
	- maintains private list of objects; can be cross-referenced with other sources
	- each session gets a dedicated copy
	- should run from system32
- `services.exe`:
	- *Service Control Manager(SCM)*
	- maintains list of services in private memory
	- should be parent of any `svchost.exe`, `spoolsv.exe`, and `SearchIndexer.exe`
	- only 1 copy should be running
	- should run from system32
- `svchost.exe`:
	- provides container for DLLs that implement services
	- multiple instances can be running
	- parent should be `services.exe`
	- should run from system32
- `lsass.exe`:
	- local security authority subsystem
	- enforces security policy
	- verifies passwords
	- creats access tokens
	- plain text password hashes can be found in memory; often target of code injection
	- only 1 instance should be running
	- parent is `wininit.exe`
	- should run from system32
- `winlogon.exe`:
	- presents interactive logon prompts
	- initiates screen saver when necessary
	- helps load user profiles
	- responds to *Secure Attention Sequence(SAS)* operations such as `CTRL+ALT+DEL`
	- monitors files and directories for chnages on systems that implement *Windows File Protection(WFP)*
	- should run from system32
- `explorer.exe`:
	- one process for each logged-on user
	- responsible for handling GUI-based folder navigation
	- has access to sensitive material; opened documents, FTP credentials
- `smss.exe`:
	- session manager
	- creates sessions that isolate OS services from users who may log on via console or *Remote Desktop Protocol(RDP)*

## 6. analyzing process activity
- commands to extract info on processes:
	- `pslist`: walks doubly linked list of processes and prints summary
	- `pstree`: takes output of `pslist` and formats into tree view
	- `psscan`: scans for `_EPROCESS` objects; can find terminated or hidden processes
	- `psxview`: locates processes using *alternate process listings*
- `pslist` example:
	``` sh
	$ python vol.py -f lab.mem --profile=WinXPSP3x86 pslist
	Volatility Foundation Volatility Framework 2.4
	Offset(V) 	Name 			PID 	PPID 	Thds 	Hnds 	Sess 	Start
	---------- 	-------------- 	------ 	----- 	----- 	------- ----- 	-------------------
	0x823c8830 	System 			4 		0 		56 		537 	-----
	```
	- `Offset` displays VA of `_EPROCESS` structure

- `pstree` example:
	``` sh
	$ python vol.py -f lab.mem --profile=WinXPSP3x86 pstree
	Volatility Foundation Volatility Framework 2.4
	[snip]
	0x82263378:explorer.exe 1300 1188 11 363 2013-03-14 03:02:42
	. 0x81e85da0:TSVNCache.exe 1556 1300 7 53 2013-03-14 03:02:43
	. 0x81e79020:firefox.exe 180 1300 27 447 2013-03-14 03:03:05
	.. 0x8202b398:AcroRd32.exe 3684 180 0 ------ 2013-03-14 14:19:16
	... 0x81ecd3c0:cmd.exe 3812 3684 1 33 2013-03-14 14:19:29
	```
- `psscan` with dot graph renderer (`--output=dot`) can also be used to create process trees

## 7. detecting DKOM attacks
- most common attack is unlinking process to hide it
- ways malware can directly modify kernel objects:
	- loading kernel driver; has unrestricted access to objects in kernel memory
	- mapping writable view of `\Device\PhysicalMemory` object
	- using special native API function `ZwSystemDebugControl`

## 8. Prolaco case study
![[decompiled_Prolaco_sample.png]]
1. enables debug privilege (`SeDebugPrivilege`)
	- access to `ZwSystemDebugControl` gained
2. calls `NtQuerySystemInformation` with `SystemModuleInformation` class to locate base of NT kernel module
3. finds `PsInitialSystemProcess`, global variable exported by NT module that points to first `_EPROCESS` object
4. finds process with `PidOfProcessToHide`
	- `0x88` is the offset to `ActiveProcessLinks` within the structure
5. calls `WriteKernelMemory`, wrapper around `ZwSystemDebugControl`

## 9. alternate process listings
- process object scanning:
	- pool scanning approach
	- pool tags are nonessential; can also be manipulated
- thread scanning:
	- can scan for `_ETHREAD` objects and map them back to owning process
	- rootkits would also have to modify pool tags for all their threads
- CSRSS handle table
- PspCid table:
	- special handle table located in kernel memory
	- stores reference to all active process and thread objects
	- pointed to by `PspCidTable` member of kernel debugger DS
	- possible to remove processes from table to hide them
- session processes
- desktop threads:
	- `tagDESKTOP` structures store a list of all threads attached to each desktop

## 10. Process Cross-View plugin
![[psxview_plugin_output.png]]

- `psxview` plugin enumerates processes in 7 different ways
- output displays 7 columns for each process it finds, identifying which method was used
- `--apply-rules` option displays `Okay` to indicate process meets one of the valid exceptions:
	- processes that start before `csrss.exe` are not in the CSRSS handle table
	- processes that start before `smss.exe` are not in the session process or desktop thread list
	- processes that have exited will not be found by any method except process object scanning and thread scanning

## 11. process tokens
- describes security context:
	- SIDs of users or groups that process is running as
	- various privileges
- consulted when kernel needs to decide whether a process can access an object or call a particular API

## 12. process token analysis objectives
- map SIDs to usernames
- detect lateral movement
- profile process behaviors:
	- privileges of a process provide clues on what the process did or will do
- detect privilege escalation:
	- live tools can report less privileges for a process
	- memory forensics can provide more accuracy

## 13. process token DS
- `_TOKEN` for 64-bit Windows 7:
	``` python
	>>> dt("_TOKEN")
	'_TOKEN' (784 bytes)
	0x0 : TokenSource ['_TOKEN_SOURCE']
	0x10 : TokenId ['_LUID']
	0x18 : AuthenticationId ['_LUID']
	[snip]
	0x40 : Privileges ['_SEP_TOKEN_PRIVILEGES']
	0x58 : AuditPolicy ['_SEP_AUDIT_POLICY']
	0x74 : SessionId ['unsigned long']
	0x78 : UserAndGroupCount ['unsigned long']
	[snip]
	0x90 : UserAndGroups ['pointer', ['array', lambda x: x.UserAndGroupCount, ['_SID_AND_ATTRIBUTES']]]
	```
	- `UserAndGroups`: 
		- array of `_SID_AND_ATTRIBUTES` structure associated with token
		- each element describes a different user/group process is member of
		- `Sid` member of `_SID_AND_ATTRIBUTES` points to `_SID` structure, which has `IdentifierAuthority` and `SubAuthority` members you can combine to form `S-1-5-[snip]` SID strings
		- `Privileges`: instance of `_SET_TOKEN_PRIVILEGES`, which has 3 parallel 64-bit values (`Present`, `Enabled`, `EnabledByDefault`) where bit position corresponds to a particular privilege
- `_SEP_TOKEN_PRIVILEGES` for 64-bit Windows 7:
	``` python
	>>> dt("_SEP_TOKEN_PRIVILEGES")
	'_SEP_TOKEN_PRIVILEGES' (24 bytes)
	0x0 : Present ['unsigned long long']
	0x8 : Enabled ['unsigned long long']
	0x10 : EnabledByDefault ['unsigned long long']
	```

## 14. accessing tokens
- a process can access its own token via `OpenProcessToken` API
- `GetTokenInformation` can be used to enumerate SIDs or privileges

## 15. extracting and translating SIDs in memory
- known SIDs hardcoded into Windows can be hardcoded into Volatility
- service SIDs composed of the SHA1 hash of the service name in Unicode uppercase and prefixed with `S-1-5-80`
- user SIDs are made up of:
	- `S`: prefix indicating string is SID
	- `1`: revision level
	- `5`: identifier authority value from `_SID.IdentifierAuthority.Value`
	- `21-4010035002-774237572-2085959976`: local computer or domain identifier from `_SID.SubAuthority` values
	- `1000`: relative identifier that represents any user or group that doesn't exist by default
- SID string can be mapped via querying the registry:
	``` sh
	$ python vol.py -f memory.img --profile=Win7SP0x86 printkey -K "Microsoft\WindowsNT\CurrentVersion\ProfileList\S-1-5-21-4010035002-774237572-2085959976-1000"
	Volatility Foundation Volatility Framework 2.4
	Legend: (S) = Stable (V) = Volatile
	----------------------------
	Registry: User Specified
	Key name: S-1-5-21-4010035002-774237572-2085959976-1000 (S)
	Last updated: 2011-06-09 19:50:32
	Subkeys:

	Values:
	REG_EXPAND_SZ ProfileImagePath : (S) C:\Users\nkESis3ns88S
	REG_DWORD Flags : (S) 0
	REG_DWORD State : (S) 0
	REG_BINARY Sid : (S)
	0x00000000 01 05 00 00 00 00 00 05 15 00 00 00 3a 47 04 ef ............:G..
	0x00000010 84 ed 25 2e 28 39 55 7c e8 03 00 00 ..%.(9U|....
	[snip]
	```

## 16. detecting lateral movement
- `getsids` plugin example:
	``` sh
	$ python vol.py –f grrcon.img --profile=WinXPSP3x86 getsids –p 1096
	Volatility Foundation Volatility Framework 2.4
	explorer.exe: S-1-5-21-2682149276-1333600406-3352121115-500 (administrator)
	explorer.exe: S-1-5-21-2682149276-1333600406-3352121115-513 (Domain Users)
	explorer.exe: S-1-1-0 (Everyone)
	explorer.exe: S-1-5-32-545 (Users)
	explorer.exe: S-1-5-32-544 (Administrators)
	explorer.exe: S-1-5-4 (Interactive)
	explorer.exe: S-1-5-11 (Authenticated Users)
	explorer.exe: S-1-5-5-0-206541 (Logon Session)
	explorer.exe: S-1-2-0 (Local (Users with the ability to log in locally))
	explorer.exe: S-1-5-21-2682149276-1333600406-3352121115-519 (Enterprise Admins)
	explorer.exe: S-1-5-21-2682149276-1333600406-3352121115-1115
	explorer.exe: S-1-5-21-2682149276-1333600406-3352121115-518 (Schema Admins)
	explorer.exe: S-1-5-21-2682149276-1333600406-3352121115-512 (Domain Admins)
	```
- used to view level of access attacker gained

## 17. privileges
- before a process can enable a privilege, it must be present in the token
- admins decide which privileges are present by configuring them in the *Local Security Policy(LSP)* or by calling `LsaAddAccountRights`
- methods to enable privileges:
	- enabled by default
	- inheritence: unless specified otherwise, child processes inherit security context of their parent
	- explicit enabling: can be done via `AdjustTokenPrivileges` API

![[local_security_policy_editor.png]]

## 18. commonly exploited privileges
- privileges to watch for if explicitly enabled:
	- `SeBackupPrivilege`: 
		- grants read access to any file on FS, regardless of *Access Control List(ACL)*
	- `SeDebugPrivilege`: 
		- grants ability to read from or write to another process' private memory
	- `SeLoadDriverPrivilege`:
		- grants ability to load/unload kernel drivers
	- `SeChangeNotifyPrivilege`:
		- allows caller to register callback function that executes when specific files/directories change
		- notifies attacker to removal of config or executable files
	- `SeShutdownPrivilege`:
		- allows caller to reboot or shut down system
		- triggers infections that modify the MBR
- `privs` plugin output:
	``` sh
	$ python vol.py -f grrcon.img privs -p 1096
	Volatility Foundation Volatility Framework 2.4
	Pid Process Privilege Attributes
	------ ------------- ------------------------------- ------------------------
	1096 explorer.exe SeChangeNotifyPrivilege Present,Enabled,Default
	1096 explorer.exe SeShutdownPrivilege Present
	1096 explorer.exe SeUndockPrivilege Present,Enabled
	1096 explorer.exe SeTakeOwnershipPrivilege Present
	1096 explorer.exe SeDebugPrivilege Present,Enabled
	1096 explorer.exe SeLoadDriverPrivilege Present,Enabled
	1096 explorer.exe SeCreatePagefilePrivilege Present
	1096 explorer.exe SeCreateGlobalPrivilege Present,Enabled,Default
	1096 explorer.exe SeImpersonatePrivilege Present,Enabled,Default
	```

## 19. token manipulation
- setting `Enabled` member of `_SEP_TOKEN_PRIVILEGES` to `0xFFFFFFFFFFFFFFFF` enables all privileges, even ones not present
- example using volatility on VM memory:
	1. get context of process
	``` sh
	$ python vol.py -f VistaSP0x64.vmem --profile=VistaSP2x64 volshell --write
	Volatility Foundation Volatility Framework 2.4
	Write support requested. Please type "Yes, I want to enable write support"
	Yes, I want to enable write support
	Current context: process System, pid=4, ppid=0 DTB=0x124000
	To get help, type 'hh()'
	>>> cc(pid = 1824)
	Current context: process explorer.exe, pid=1824, ppid=1668 DTB=0x918d000
	```
	2. obtain pointer to `_TOKEN` structure
	``` python
	>>> token = proc().get_token()
	>>> bin(token.Privileges.Present)
	'0b11000000010100010000000000000000000'
	>>> bin(token.Privileges.Enabled)
	'0b100000000000000000000000'
	```
	3. set all enabled
	``` python
	>>> token.Privileges.Enabled = 0xFFFFFFFFFFFFFFFF
	>>> bin(token.Privileges.Present)
	'0b11000000010100010000000000000000000'
	>>> bin(token.Privileges.Enabled)
	'0b1111111111111111111111111111111111111111111111111111111111111111'
	>>> quit()
	```
	- process explorer output:
		![[process_explorer_privileges_output.png]]
		- only 5 privileges are shown as enabled
	- `privs` plugin output:	
		``` sh
		$ python vol.py -f VistaSP0x64.vmem --profile=VistaSP2x64 privs -p 1824
		Volatility Foundation Volatility Framework 2.4
		Pid Process Privilege Attributes
		----- ------------ -------------------------------- --------------------
		1824 explorer.exe SeCreateTokenPrivilege Enabled
		1824 explorer.exe SeAssignPrimaryTokenPrivilege Enabled
		[snip]
		1824 explorer.exe SeRestorePrivilege Enabled
		1824 explorer.exe SeShutdownPrivilege Present,Enabled
		1824 explorer.exe SeDebugPrivilege Enabled
		1824 explorer.exe SeAuditPrivilege Enabled
		1824 explorer.exe SeSystemEnvironmentPrivilege Enabled
		1824 explorer.exe SeChangeNotifyPrivilege Present,Enabled,Default
		```
		- privileges that aren't even present are enabled

## 20. process handles
- reference to an open instance of a kernel object; e.g. file, registry key, mutex, process, thread
- handle creation example:
	1. creation API is called
	2. API places pointer to object in first available slot of calling process' handle table
	3. API returns Windows data type `HANDLE`, index into handle table
	4. handle count of object is incremented
- handle usage example:
	1. access API is called
	2. API finds base address of handle table
	3. seek index given with `HANDLE`
	4. retieve pointer
	5. carry out operation
- closing handles decrements the object's handle count and removes the pointer from the handle table
- handle table benefits:
	- convenience: no need to pass full name or path for each operation
	- security: prevents processes in user mode from having direct access to kernel objects

## 21. kernel handles
- kernel modules, or threads in kernel mode can call similar kernel APIs (i.e. `NtCreateFile`, `NtReadFile`, `NtCreateMutex`) to obtain handles
- kernel handles are allocated from `System` (PID 4) process handle table
- kernel code can access objects directly via `ObReferenceObjectByPointer` if address is known:
	- increments reference count, not handle count
	- `ObDereferenceObject` should be called to dereference object or objects may persist (reference leak)

## 22. handle analysis objectives
- handle table internals
- targeted object attribution
- unknown process investigation
- detect registry persistence
- identify remote mapped drives

## 23. handle table
- each process' `_EPROCESS.ObjectTable` member points to `_HANDLE_TABLE`
- has `TableCode` that serves 2 purposes:
	- specifies number of levels in table
	- points to base of first level
- all processes start with a single-level table
- single-level table:
	![[single_level_table.png]]
- two-level table contains a first level that points to an array of `_HANDLE_TABLE_ENTRY` structures, allowing 1024x512 handles on x86
- theoretically three-level table can contain up to 1024x1024x512 handles; observed limtits are fewer
- handle table DS for 64-bit Windows 7:
	``` python
	>>> dt("_HANDLE_TABLE")
	'_HANDLE_TABLE' (104 bytes)
	0x0 : TableCode ['unsigned long long']
	0x8 : QuotaProcess ['pointer64', ['_EPROCESS']]
	0x10 : UniqueProcessId ['pointer64', ['void']]
	0x18 : HandleLock ['_EX_PUSH_LOCK']
	0x20 : HandleTableList ['_LIST_ENTRY']
	0x30 : HandleContentionEvent ['_EX_PUSH_LOCK']
	0x38 : DebugInfo ['pointer64', ['_HANDLE_TRACE_DEBUG_INFO']]
	0x40 : ExtraInfoPages ['long']
	0x44 : Flags ['unsigned long']
	0x44 : StrictFIFO ['BitField', {'end_bit': 1, 'start_bit': 0, 'native_type': 'unsigned char'}]
	0x48 : FirstFreeHandle ['unsigned long']
	0x50 : LastFreeHandleEntry ['pointer64', ['_HANDLE_TABLE_ENTRY']]
	0x58 : HandleCount ['unsigned long']
	0x5c : NextHandleNeedingPool ['unsigned long']
	0x60 : HandleCountHighWatermark ['unsigned long']
	```
	- `TableCode`: `TableCode & 7` obtains number of tables, `TableCode & ~7` obtains address of top-level table
	- `QuotaProcess`: pointer to process
	- `HandleTableList`: linked list of process handle tables 
- table entry DS
	``` python
	>>> dt('_HANDLE_TABLE_ENTRY')
	'_HANDLE_TABLE_ENTRY' (16 bytes)
	0x0 : InfoTable ['pointer64', ['_HANDLE_TABLE_ENTRY_INFO']]
	0x0 : ObAttributes ['unsigned long']
	0x0 : Object ['_EX_FAST_REF']
	0x0 : Value ['unsigned long long']
	0x8 : GrantedAccess ['unsigned long']
	0x8 : GrantedAccessIndex ['unsigned short']
	0x8 : NextFreeTableEntry ['unsigned long']
	0xa : CreatorBackTraceIndex ['unsigned short']
	```
	- `Object`: pointer to `_OBJECT_HEADER`. `_EX_FAST_REF` is a special data type that combines reference count information into least significant bits of pointer
	- `GrantedAccess`: bit mask specifying granted access owning process has obtained for object

## 24. `handles` plugin
- generates output by walking handle table DS
- using without options generates output for all handles for all object types in all processes
- filtering options:
	- `-p/--pid`: filter by PID
	- `-o/-offset`: filter by physical offset of `_EPROCESS`
	- `-t/--object-type`: filter by object type
	- `--silent`: suppresses handles to unnamed objects

## 25. Multiple *Universal Naming Convention(UNC)* Provider
- aka MUP
- kernel-mode component that channels requests to access remote files using UNC names to the appropriate redirector
- `LanmanRedirector` handles SMB
- search for file handles prefixed with `\Device\Mup` and `\Device\LanmanRedirector` for evidence of remote mapped drives
- `handles` plugin output:
	``` sh
	$ python vol.py -f hop.mem --profile=VistaSP2x64 handles -t File | grep Mup
	Volatility Foundation Volatility Framework 2.4
	Offset(V) Pid Handle Access Type Details
	------------------ ------ ------------ ----------- -------- -------
	[snip]
	0xfffffa8001345c80 752 0xfc 0x100000 File \Device\Mup\;P:000000000002210f\WIN-464MMR8O7GF\Users
	0xfffffa8003f02050 752 0x200 0x100000 File \Device\Mup
	0xfffffa80042c9f20 752 0x204 0x100000 File \Device\Mup
	0xfffffa800134b190 752 0x264 0x100000 File \Device\Mup\;Q:000000000002210f\LH-7J277PJ9J85I\C$
	0xfffffa800132b450 1544 0x8 0x100020 File \Device\Mup\;Q:000000000002210f\LH-7J277PJ9J85I\C$\Users\Jimmy\Documents
	```
- inspecting symbolic links to associate drive letter with redirected path:
	``` sh
	$ python vol.py -f hop.mem --profile=VistaSP2x64 symlinkscan
	Volatility Foundation Volatility Framework 2.4
	Offset(P) #Ptr #Hnd Creation time From To
	------------------ ------ ------ ------------------------------ ---------- --
	0x0000000024b0c6c0 1 0 2014-02-25 21:41:12 UTC+0000 Q:\Device\LanmanRedirector\;Q:0...00002210f\LH-7J277PJ9J85I\C$
	0x0000000026f4a800 1 0 2014-02-25 21:40:45 UTC+0000 P:\Device\LanmanRedirector\;P:0...02210f\WIN-464MMR8O7GF\Users
	```
