## 1. process memory contents
![[process_memory_contents.png]]
	- *Dynamic Linked Libraries(DLL)*: represents DLLs loaded into address space
	- environment variables: environment variables such as: executable path, temporary directories, home folders, etc
	- *Process Environment Block(PEB)*: 
		- tells where to find several other items in list; e.g. DLLs, heaps, and environment variables
		- contains process' command line arguments, current working directory, and standard handles
	- process heap: dynamic input process receives
	- thread stacks: function arguments, return addresses, and local variables
	- mapped files: content from files on disk
	- app data: everything process needs to work
	- executable: primary block of code and read/write variables for application
- process layout is not constant, especially on systems with *Address Space Layout Randomization(ASLR)*

## 2. memory allocation APIs
![[memory _allocation_APIs.png]]

- permissions:
	- `VirtualAlloc` enables specifying memory permissions; e.g. no-access, readable, writeable, executable, guarded
	- heaps are always readable and writeable
	- heap data can be executable via `HEAP_CREATE_ENABLE_EXECUTE` parameter to `HeapCreate`
- scope and flexibility:
	- `VirtualAllocEx` is the only API that allows one process to allocate memory for another process; used frequently for code injection
	- `VirtualAlloc(Ex)` are the only APIs allowed to reserve contiguous memory before commiting
- memory ranges allocated with `VirtualAlloc(Ex)` can be enumerated with `VirtualQueryEx`

## 3. enumerating process memory
- list of sources of process memory:
	- page tables:
		- used to map VA in process memory to physical offsets in RAM
		- can determine what pages are swapped to disk 
		- anlayze hardware-based-permissions applied to pages
	- *Virtual Address Descriptors(VAD)*:
		- tracks reserved or committed, virtually contiguous collections of pages
	- working set list:
		- describes collection of recently accessed pages in virtual memory that are present in phyiscal memory
	- PFT database:
		- tracks state of each page in physical memory
		- an array of `_MMPFN` structures that you can access from the kernel's debugger data block
	
## 4. process page tables
- `memmap` and `memdump` plugins enable you to extract all pages accessible to a process including kernel mode addresses
- `memdump` visualization
	![[memdump_visualization.png]]
- `memmap` output:
	``` sh
	$ python vol.py -f memory.dmp --profile=Win7SP0x64 memmap –p 864
	Volatility Foundation Volatility Framework 2.4
	winlogon.exe pid: 864
	Virtual Physical Size DumpFileOffset
	------------------ ------------------ ------------------ ------------------
	0x0000000000010000 0x0000000151162000 0x1000 0x0
	0x0000000000020000 0x0000000158de3000 0x1000 0x1000
	0x0000000000021000 0x0000000158c64000 0x1000 0x2000
	0x0000000000022000 0x0000000158d65000 0x1000 0x3000
	0x0000000000023000 0x0000000158de6000 0x1000 0x4000
	0x0000000000024000 0x0000000158d67000 0x1000 0x5000
	[snip]
	0x000000007efe3000 0x000000015200c000 0x1000 0x29f000
	0x000000007efe4000 0x000000015208d000 0x1000 0x2a0000
	0x000000007ffe0000 0x00000000001e6000 0x1000 0x2a1000
	0x00000000ff4c0000 0x000000015170a000 0x1000 0x2a2000
	0x00000000ff4c1000 0x0000000151b77000 0x1000 0x2a3000
	[snip]
	0xfffff80000bab000 0x0000000000bab000 0x1000 0xae4000
	0xfffff80000bac000 0x0000000000bac000 0x1000 0xae5000
	0xfffff80002800000 0x0000000002800000 0x200000 0xae6000
	0xfffff80002a00000 0x0000000002a00000 0x200000 0xce6000
	0xfffff80002c00000 0x0000000002c00000 0x200000 0xee6000
	0xfffff80002e00000 0x0000000002e00000 0x200000 0x10e6000
	0xfffff80003c7a000 0x0000000003c7a000 0x1000 0x12e6000
	[snip]
	```
	- virtually contiguous memory is not physically contiguous
	- VA `0x7ffe0000` is the `KUSER_SHARED_DATA` region, created by kernel and shared with all processes. Does not change per system
	- pages of size `0x200000` are *Page Size Entry(PSE)* pages

## 5. Virtual Address Descriptors
- process VAD tree describes layout of memory segments at higher level than page tables; OS, not the CPU, defines and maintains these DS
- contain:
	- names of memory-mapped files
	- total number of pages in region
	- initial protection (read, write, execute)
	- several other flags that can tell a lot about what type of data the regions contain
- VAD tree is a self-balancing binary tree:
	- each node represents one range in process virtual memory
	- lower memory range node goes to the left and higher to the right

- `vadtree` plugin with `--render=dot` option output:
	![[vadtree_plugin_output.png]]
	- process heaps are red
	- thread stacks are green
	- mapped fields are yellow
	- DLLs are gray

## 6. VAD structures
- `_EPROCESS.VadRoot` points to root of tree
- VAD stucture per version

	| Operating System | VadRoot | Node | 
	| --- | --- | --- | 
	| Windows XP | Pointer to a node | `_MMVAD_SHORT`, `_MMVAD`, `_MMVAD_LONG` | 
	| Windows 2003 Server | Same as XP | Same as XP | 
	| Windows Vista | `_MM_AVL_TABLE` | `_MMADDRESS_NODE`(alias) | 
	| Windows 2008 Server | Same as Vista | Same as Vista | 
	| Windows 7 | Same as Vista | Same as Vista | 
	| Windows 8, 2012 | Same as Vista | `_MM_AVL_NODE`(alias) | 
	| Windows 8.1, 2012R2 | `_RTL_AVL_TREE` | `_RTL_BALANCED_NODE`(alias) | 
	- all nodes are just aliases of `_MMVAD(_SHORT/_LONG)`

- 64-bit Windows 7 DS
	``` python
	>>> dt("_MM_AVL_TABLE")
	'_MM_AVL_TABLE' (64 bytes)
	0x0 : BalancedRoot ['_MMADDRESS_NODE']
	0x28 : DepthOfTree ['BitField', {'end_bit': 5,
	'start_bit': 0, 'native_type': 'unsigned long long'}]
	0x28 : NumberGenericTableElements ['BitField', {'end_bit': 64,
	'start_bit': 8, 'native_type': 'unsigned long long'}]
	0x28 : Unused ['BitField', {'end_bit': 8,
	'start_bit': 5, 'native_type': 'unsigned long long'}]
	0x30 : NodeHint ['pointer64', ['void']]
	0x38 : NodeFreeHint ['pointer64', ['void']]
	```
	``` python
	>>> dt("_MMADDRESS_NODE")
	'_MMADDRESS_NODE' (40 bytes)
	-0xc : Tag ['String', {'length': 4}]
	0x0 : u1 ['__unnamed_15cd']
	0x8 : LeftChild ['pointer64', ['_MMADDRESS_NODE']]
	0x10 : RightChild ['pointer64', ['_MMADDRESS_NODE']]
	0x18 : StartingVpn ['unsigned long long']
	0x20 : EndingVpn ['unsigned long long']
	```
	- can derive first and last *Virtual Page Number(VPN)*
- `_MMVAD` nodes:
	``` python
	>>> dt("_MMVAD_SHORT")
	'_MMVAD_SHORT' (64 bytes)
	-0xc : Tag ['String', {'length': 4}]
	0x0 : u1 ['__unnamed_15bf']
	0x8 : LeftChild ['pointer64', ['_MMVAD']]
	0x10 : RightChild ['pointer64', ['_MMVAD']]
	0x18 : StartingVpn ['unsigned long long']
	0x20 : EndingVpn ['unsigned long long']
	0x28 : u ['__unnamed_15c2']
	0x30 : PushLock ['_EX_PUSH_LOCK']
	0x38 : u5 ['__unnamed_15c5']
	>>> dt("_MMVAD")
	'_MMVAD' (120 bytes)
	-0xc : Tag ['String', {'length': 4}]
	0x0 : u1 ['__unnamed_15bf']
	0x8 : LeftChild ['pointer64', ['_MMVAD']]
	0x10 : RightChild ['pointer64', ['_MMVAD']]
	0x18 : StartingVpn ['unsigned long long']
	0x20 : EndingVpn ['unsigned long long']
	0x28 : u ['__unnamed_15c2']
	0x30 : PushLock ['_EX_PUSH_LOCK']
	0x38 : u5 ['__unnamed_15c5']
	0x40 : u2 ['__unnamed_15d2']
	0x48 : MappedSubsection ['pointer64', ['_MSUBSECTION']]
	0x48 : Subsection ['pointer64', ['_SUBSECTION']]
	0x50 : FirstPrototypePte ['pointer64', ['_MMPTE']]
	0x58 : LastContiguousPte ['pointer64', ['_MMPTE']]
	0x60 : ViewLinks ['_LIST_ENTRY']
	0x70 : VadsProcess ['pointer64', ['_EPROCESS']]
	>>> dt("_MMVAD_LONG")
	'_MMVAD_LONG' (144 bytes)
	-0xc : Tag ['String', {'length': 4}]
	0x0 : u1 ['__unnamed_15bf']
	0x8 : LeftChild ['pointer64', ['_MMVAD']]
	0x10 : RightChild ['pointer64', ['_MMVAD']]
	0x18 : StartingVpn ['unsigned long long']
	0x20 : EndingVpn ['unsigned long long']
	0x28 : u ['__unnamed_15c2']
	0x30 : PushLock ['_EX_PUSH_LOCK']
	0x38 : u5 ['__unnamed_15c5']
	0x40 : u2 ['__unnamed_15d2']
	0x48 : Subsection ['pointer64', ['_SUBSECTION']]
	0x50 : FirstPrototypePte ['pointer64', ['_MMPTE']]
	0x58 : LastContiguousPte ['pointer64', ['_MMPTE']]
	0x60 : ViewLinks ['_LIST_ENTRY']
	0x70 : VadsProcess ['pointer64', ['_EPROCESS']]
	0x78 : u3 ['__unnamed_1c7f']
	0x88 : u4 ['__unnamed_1c85']
	```
	- `Subsection` is used by the OS to track info on files/DLLs mapped into the region
	- regular and long node can be ignored when hunting for code injection

## 7. VAD tags
- exists right before node
- indicates type of data stored:

	| Tag | Node Type | 
	| --- | --- | 
	| Vadl | `_MMVAD_LONG` | 
	| Vadm | `_MMVAD_LONG` | 
	| Vad | `_MMVAD_LONG` | 
	| VadS | `_MMVAD_SHORT` | 
	| VadF | `_MMVAD_SHORT` | 

## 8. VAD flags
- located in embedded unions called `u`, `u1`, `u2`, ...; e.g. short node's `u` member is of type `__unnamed_15c2`
- `__unnamed_15c2` data type:
	``` python
	>>> dt("__unnamed_15c2")
	'__unnamed_15c2' (8 bytes)
	0x0 : LongFlags ['unsigned long long']
	0x0 : VadFlags ['_MMVAD_FLAGS']
	```
	- can be referred to by either
- `_MMVAD_FLAGS`:
	``` python
	>>> dt("_MMVAD_FLAGS")
	'_MMVAD_FLAGS' (8 bytes)
	0x0 : CommitCharge ['BitField', {'end_bit': 51, 'start_bit': 0, 'native_type': 'unsigned long long'}]
	0x0 : NoChange ['BitField', {'end_bit': 52, 'start_bit': 51, 'native_type': 'unsigned long long'}]
	0x0 : VadType ['BitField', {'end_bit': 55, 'start_bit': 52, 'native_type': 'unsigned long long'}]
	0x0 : MemCommit ['BitField', {'end_bit': 56, 'start_bit': 55, 'native_type': 'unsigned long long'}]
	0x0 : Protection ['BitField', {'end_bit': 61, 'start_bit': 56, 'native_type': 'unsigned long long'}]
	0x0 : Spare ['BitField', {'end_bit': 63, 'start_bit': 61, 'native_type': 'unsigned long long'}]
	0x0 : PrivateMemory ['BitField', {'end_bit': 64, 'start_bit': 63, 'native_type': 'unsigned long long'}]
	```
	- commit charge:
		- number of pages committed in region described by node
		- code injecting malware usually commits all pages up front
	- protection:
		- indicates type of access allowed to memory region
		- loosely coupled with memory protection constants passed to virtual allocation APIs
		- values:
			- `PAGE_EXECUTE`: can't be used for mapped files
			- `PAGE_EXECUTE_READ`
			- `PAGE_EXECUTE_READWRITE`: injected code regions almost always have this protection
			- `PAGE_EXECUTE_WRITECOPY`: 
				- execute + copy-on-write access
				- cannot be set via virtual allocation APIs
				- DLLs almost always have this protection
			- `PAGE_NOACCESS`: can't be used for mapped files
			- `PAGE_READONLY`
			- `PAGE_READWRITE`
			- `PAGE_WRITECOPY`: cannot be set via virtual allocation APIs
		- only initial protection for when range was reserved or committed
		- permissions can be applied at page granularity; need to view bits in page table
	- private memory:
		- refers to committed regions that typically can't be shared with or inherited by other processes
		- heaps, stacks, and ranges allocated with virtual allocation APIs are usually marked as private

## 9. VAD plugins
- `vadinfo`: displays most verbose output; start/end address, protection level, flags, full paths to mapped files or DLLs
- `vadtree`: 
	- prints tree-view of nodes
	- supports generating color-coded graphs
- `vaddump`: 
	- extracts range of process memory each VAD node describes to separate file on disk
	- output padded with zeros if any pages in range are swapped to disk; maintains spatial integrity

## 10. traversing VAD in python
- iterating through and searching for signatures in VAD:
	``` python
	$ python vol.py -f memory.dmp --profile=Win7SP1x64 volshell -p 1080
	Volatility Foundation Volatility Framework 2.4
	Current context: process explorer.exe, pid=1080, ppid=1452 DTB=0x19493000
	To get help, type 'hh()'
	>>> process = proc()
	>>> process_space = process.get_process_address_space()
	>>> for vad in process.VadRoot.traverse():
	... data = process_space.read(vad.Start, 1024)
	... if data:
	... found = data.find("MZ")
	... if found != -1:
	... print "Found signature in VAD", hex(vad.Start)
	...
	Found signature in VAD 0x3840000L
	Found signature in VAD 0xd0000L
	[snip]
	```
- searching for Gmail passwords in Chrome memory:
	``` python
	$ python vol.py -f memory.dmp --profile=Win7SP1x64 volshell –p 3660
	Volatility Foundation Volatility Framework 2.4
	Current context: process chrome.exe, pid=3660, ppid=3560 DTB=0x1b9fc000
	To get help, type 'hh()'
	>>> process = proc()
	>>> process_space = process.get_process_address_space()
	>>> criteria = []
	>>> criteria.append("&Email".encode("utf_16_le"))
	>>> criteria.append("&Passwd".encode("utf_16_le"))
	>>> for addr in process.search_process_memory(criteria):
	... string = obj.Object("String",
	... offset = addr, vm = process_space,
	... encoding = "utf16", length = 64)
	... print str(string)
	...
	&Email=hack4life2&Passwd=dater7-
	&Passwd=dater7-tarry&signIn=Sign
	&Email=hack4life200&Passwd=dater
	&Passwd=dater7-tarry&signIn=Sign
	```

## 11. `yarascan` plugin
- virtual memory scanning; fragmentation is not an issue and hits can be attributed to memory owner
- main capabilities:
	- `--pid`: scan multiple processes for a given signature
	- `--offset`: scan physical offset of `_EPROCESS`
	- `--kernel`: scan entire range of kernel memory and show name of kernel module
	- `--yara-file`: find signatures supplied in file
	- `--dump-dir`: extract memory ranges
- scan ideas:
	- IP addresses, domain names, or URLs
	- regular expressions for credit card numbers, phone numbers, birthdays, password hashes, etc
	- packer signatures
	- antivirus signatures
- example output:
	``` sh
	$ python vol.py -f memory.dmp --profile=Win7SP1x64 yarascan --yara-rules="&Email" --wide -p 3560,3660,3808
	Volatility Foundation Volatility Framework 2.4
	Rule: r1
	Owner: Process chrome.exe Pid 3660
	0x03172652 2600 4500 6d00 6100 6900 6c00 3d00 6800 &.E.m.a.i.l.=.h.
	0x03172662 6100 6300 6b00 3400 6c00 6900 6600 6500 a.c.k.4.l.i.f.e.
	0x03172672 3200 2600 5000 6100 7300 7300 7700 6400 2.&.P.a.s.s.w.d.
	0x03172682 3d00 6400 6100 7400 6500 7200 3700 2d00 =.d.a.t.e.r.7.-.
	Rule: r1
	Owner: Process chrome.exe Pid 3660
	0x0260b078 2600 4500 6d00 6100 6900 6c00 3d00 6800 &.E.m.a.i.l.=.h.
	0x0260b088 6100 6300 6b00 3400 6c00 6900 6600 6500 a.c.k.4.l.i.f.e.
	0x0260b098 3200 3000 3000 2600 5000 6100 7300 7300 2.0.0.&.P.a.s.s.
	0x0260b0a8 7700 6400 3d00 6400 6100 7400 6500 7200 w.d.=.d.a.t.e.r
	```
- finding Zeus encryption keys in memory:
	![[Zeus_disassembled.png]]
	- `config_data` contains offset, relative to base of executable, to encrypted config data
	- `real_rc4_key` contains offset to key
	- `signature_function` expressed:
		``` assembly
		PUSH 102h
		LEA EAX, [ESP+????????]
		PUSH EAX
		LEA EAX, [ESP+??]
		PUSH EAX
		CALL ???????? ; custom_memcpy
		MOV EAX, 1E6h
		PUSH EAX
		PUSH OFFSET ???????? ; real_rc4_key
		```
	- `decode_config_data` expressed:
		``` assembly
		PUSH ESI
		MOV EDX, ????0000 ; config size (immediate)
		PUSH EDX
		PUSH OFFSET ???????? ; config_data
		PUSH EAX
		CALL ???????? ; custom_memcpy
		MOV ESI, ???????? ; last_section_rva
		MOV ECX, ???????? ; imagebase
		```
	- yara signature:
		``` yara
		signatures = {
		'namespace1':'rule z1 {strings: $a = {56 BA ?? ?? 00 00 52
		68 ?? ?? ?? ?? 50 E8 ?? ?? ?? ?? 8B 35 ?? ??
		?? ?? 8B 0D ?? ?? ?? ??} condition: $a}',
		'namespace5':'rule z5 {strings: $a = {56 BA ?? ?? 00 00 52
		68 ?? ?? ?? ?? 50 E8 ?? ?? ?? ?? 8B 0D ?? ?? ?? ?? 03
		0D ?? ?? ?? ??} condition: $a}',
		'namespace2':'rule z2 {strings: $a = {55 8B EC 51 A1 ?? ??
		?? ?? 8B 0D ?? ?? ?? ?? 56 8D 34 01 A1 ?? ?? ?? ?? 8B
		0D ?? ?? ?? ??} condition: $a}',
		'namespace3':'rule z3 {strings: $a = {68 02 01 00 00 8D 84
		24 ?? ?? ?? ?? 50 8D 44 24 ?? 50 E8 ?? ?? ?? ?? B8 E6
		01 00 00 50 68 ?? ?? ?? ??} condition: $a}',
		'namespace4':'rule z4 {strings: $a = {68 02 01 00 00 8D 85
		?? ?? ?? ?? 50 8D 85 ?? ?? ?? ?? 50 E8 ?? ?? ?? ?? B8
		E6 01 00 00 50 68 ?? ?? ?? ??} condition: $a}'
		}
		```
	- filtering for VAD nodes that are executable, not backed by a file, and committed narrows down search
