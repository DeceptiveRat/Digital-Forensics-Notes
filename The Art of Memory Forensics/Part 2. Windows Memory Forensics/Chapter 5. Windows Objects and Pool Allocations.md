## 1. *Windows Executive Objects*
- managed by Windows Object Manager
- a structure becomes an executive object when OS prepends headers to it in order to manage services:
	- naming
	- access control
	- reference count
- forensically relelvant objects (object name - structure):
	- file - `_FILE_OBJECT`:
		- represents process or kernel module's access into a file
		- includes:
			- permissions
			- regions of memory that store content
			- file name
	- process - `_EPROCESS`:
		- container that allows threads to execute within private VAS
		- maintains open handles to resources such as files, registry keys, etc
	- symbolic link - `_OBJECT_SYMBOLIC_LINK`:
		- supports aliases
		- helps map network share paths and removable media devices to drive letters
	- token - `_TOKEN`:
		- stores security context information for processes/threads; e.g. SIDs and privileges
	- thread - `_ETHREAD`:
		- represents schduled execution entity within a process and its associated CPU context
	- mutant - `_KMUTANT`:
		- represents mutual exclusion
		- used for synchronization purposes or controlling access to resources
	- WindowStation - `tagWINDOWSTATION`:
		- security boundary for processes and desktops
		- contains clipboard and atom tables
	- desktop - `tagDESKTOP`:
		- represents displayable screen surface
		- contains user objects such as windows, menus, and buttons
	- driver - `_DRIVER_OBJECT`:
		- represents image of loaded kernel-mode driver
		- contains addresses of driver's input/output control handler functions
	- key - `_CM_KEY_BODY`:
		- instance of open registry key
		- contains info about key's values and data
	- type - `_OBJECT_TYPE`:
		- object with metadata
		- describes common properties of all other objects

## 2. object headers
![[64-bit Windows executive objects and headers.png]]

- one of the common traits betwen executive object types
- immediately precedes executive object structure in memory
- any optional headers precede object header in fixed order
- object header structure for 64-bit Windows 7:
	``` python
	>>> dt("_OBJECT_HEADER")
	'_OBJECT_HEADER' (56 bytes)
	0x0 : PointerCount 				['long long']
	0x8 : HandleCount 				['long long']
	0x8 : NextToFree 				['pointer64', ['void']]
	0x10 : Lock 					['_EX_PUSH_LOCK']
	0x18 : TypeIndex 				['unsigned char']
	0x19 : TraceFlags 				['unsigned char']
	0x1a : InfoMask 				['unsigned char']
	0x1b : Flags 					['unsigned char']
	0x20 : ObjectCreateInfo 		['pointer64', ['_OBJECT_CREATE_INFORMATION']]
	0x20 : QuotaBlockCharged 		['pointer64', ['void']]
	0x28 : SecurityDescriptor 		['pointer64', ['void']]
	0x30 : Body 					['_QUAD']
	```
	- `PointerCount`: contains total number of points to object
	- `HandleCount`: contains number of open handles to object
	- `TypeIndex`: type of object; e.g. process, thread, file
	- `InfoMask`: present optional headers
	- `SecurityDescriptor`: stores information on security restrictions on object; which users can access it, etc
	- `Body`: placeholder representing start of structure contained in object

## 3. optional headers
- optional header list:

	| Name | Structure | Bit Mask | Size(Bytes) | Description | 
	| --- | --- | --- | --- | --- | 
	| Creator Info | `_OBJECT_HEADER_CREATOR_INFO` | 0x1 | 32 | Stores information on the creator of the object | 
	| Name Info | `_OBJECT_HEADER_NAME_INFO` |  0x2 | 32 | Stores the object’s name | 
	| Handle Info | `_OBJECT_HEADER_HANDLE_INFO` | 0x4 | 16 | Maintains data about processes with open handles to the object | 
	| Quota Info | `_OBJECT_HEADER_QUOTA_INFO` | 0x8 | 32 | Tracks usage and resource stats | 
	| Process Info | `_OBJECT_HEADER_PROCESS_INFO` | 0x10 | 16 | Identifies the owning process 

## 4. object type objects
- `TypeIndex` of `_OBJECT_HEADER` is an index into `nt!ObTypeIndexTable`, an array of type objects
- example of object type structure for 64-bit Windows 7:
	``` python
	>>> dt("_OBJECT_TYPE")
	'_OBJECT_TYPE' (208 bytes)
	0x0 : TypeList 							['_LIST_ENTRY']
	0x10 : Name 							['_UNICODE_STRING']
	0x20 : DefaultObject 					['pointer64', ['void']]
	0x28 : Index 							['unsigned char']
	0x2c : TotalNumberOfObjects 			['unsigned long']
	0x30 : TotalNumberOfHandles 			['unsigned long']
	0x34 : HighWaterNumberOfObjects 		['unsigned long']
	0x38 : HighWaterNumberOfHandles 		['unsigned long']
	0x40 : TypeInfo 						['_OBJECT_TYPE_INITIALIZER']
	0xb0 : TypeLock 						['_EX_PUSH_LOCK']
	0xb8 : Key 								['unsigned long']
	0xc0 : CallbackList 					['_LIST_ENTRY']
	```
	- `Name`: Unicode string name of object type
	- `TotalNumberOfObjects`: total number of objects of this object type in system
	- `TotalNumberOfHandles`: total number of open handles to objects of this type
	- `TypeInfo`: `_OBJECT_TYPE_INITIALIZER` structure used to denote type of memory used to allocate instances of these objects; e.g. paged/nonpaged memory
	- `Key`: 4-byte tag used to uniquely mark memory allocations that contain objects of this type
- `TypeInfo` and `Key` tell how to find all instances of a particular object type
- getting `TypeInfo` and `Key` from memory dump:
	``` sh
	$ python vol.py –f memory.dmp --profile=Win7SP1x64 volshell
	```
	``` python
	>>> kernel_space = addrspace()
	>>> ObTypeIndexTable = 0xFFFFF80002870300
	>>> ptrs = obj.Object("Array",
	... 					targetType = "Pointer",
	... 					offset = ObTypeIndexTable,
	... 					count = 100,
	... 					vm = kernel_space)
	>>> for i, ptr in enumerate(ptrs):
	... 	objtype = ptr.dereference_as("_OBJECT_TYPE")
	... 	if objtype.is_valid():
	... 		print i, str(objtype.Name), "in",
	... 		str(objtype.TypeInfo.PoolType),
	... 		"with key",
	... 		str(objtype.Key)
	...
	2 Type in NonPagedPool with key ObjT
	3 Directory in PagedPool with key Dire
	4 SymbolicLink in PagedPool with key Symb
	5 Token in PagedPool with key Toke
	6 Job in NonPagedPool with key Job
	7 Process in NonPagedPool with key Proc
	```
- getting `TypeInfo` and `Key` without address of `nt!ObTypeIndexTable`:
	``` sh
	$ python vol.py -f win7x64cmd.dd --profile=Win7SP0x64 objtypescan
	```
	``` python 
	Volatility Foundation Volatility Framework 2.4
	Offset 				nObjects 	nHandles 	Key 	Name 					PoolType
	------------------	---------- 	---------- 	----- 	-------------------- 	---------
	0xfffffa8001840190	0x2a 		0x0 		ObjT 	Type 					NonPaged
	0xfffffa80018469f0 	0x1 		0x1 		IoCo 	IoCompletionReserve 	NonPaged
	```

## 5. kernel pool allocations
![[executive object with pool header.png]]

- *kernel pool* is a range of memory that can be divided up into smaller blocks for storing any type of data that a kernel-mode component requests
- each allocated block has a header, `_POOL_HEADER` that contains accounting/debugging info; used to attribute blocks back to driver that owns them
- pool header in 64-bit Windows 7:
	``` python
	>>> dt("_POOL_HEADER")
	'_POOL_HEADER' (16 bytes)
	0x0 : BlockSize 					['BitField', {'end_bit': 24, 'start_bit': 16, 'native_type': 'unsigned long'}]
	0x0 : PoolIndex 					['BitField', {'end_bit': 16, 'start_bit': 8, 'native_type': 'unsigned long'}]
	0x0 : PoolType 						['BitField', {'end_bit': 32, 'start_bit': 24, 'native_type': 'unsigned long'}]
	0x0 : PreviousSize 					['BitField', {'end_bit': 8, 'start_bit': 0, 'native_type': 'unsigned long'}]
	0x0 : Ulong1 						['unsigned long']
	0x4 : PoolTag 						['unsigned long']
	0x8 : AllocatorBackTraceIndex 		['unsigned short']
	0x8 : ProcessBilled 				['pointer64', ['_EPROCESS']]
	0xa : PoolTagHash 					['unsigned short']
	```
	- `BlockSize`: size of allocation; includes ppl header, object header and optional headers
	- `PoolType`: type of system memory(paged, nonpaged, etc)
	- `PoolTag`: 4-byte value to uniquely identify code path taken produce allocation; troubled blocks can be traced back to source

## 6. allocation
- `ExAllocatePoolWithTag` prototype:
	``` C
	PVOID ExAllocatePoolWithTag(
		_In_ POOL_TYPE PoolType,
		_In_ SIZE_T NumberOfBytes,
		_In_ ULONG Tag
	);
	```
	- `PoolType` specifies type of system memory. `NonPagedPool`(0) and `PagedPool`(1)
	- `Ta` is derived from `_OBJECT_TYPE.Key` for executive objects
- creating a new file using Windows API:
	1. process calls `CreateFileA` or `CreateFileW` from `kernel32.dll`
	2. create file APIs lead into `ntdll.dll`, which calls `NtCreateFile` function
	3. `NtCreateFile` calls `ObCreateObject` to request new file object type
	4. `ObCreateObject` calculates size of `_FILE_OBJECT` including header sizes
	5. `ObCreateObject` finds `_OBJECT_TYPE` structure and determines pool type and tag
	6. `ExAllocatePoolWithTag` is called with appropriate size, type, and tag

## 7. de-allocation
- when an object is not being used by any process, it is released to pool's "free list"
- data remains until it is overwritten

## 8. pool-tag scanning
- refers to finding allocations based on 4-byte tag values
- volatility builds a more robust signature including:
	- size of allocation
	- type of memory
- pool tag data used by volatilty 

	| Object | Tag | Tag (Protected) | Min Size (Win7 x64) | Memory Type | Plugin |
	| --- | --- | --- | --- | --- | --- |
	| Process | Proc | Pro\xe3 | 1304 | Nonpaged | psscan | 
	| Threads | Thrd | Thr\xe4 | 1248 | Nonpaged | thrdscan | 
	| Desktops | Desk | Des\xeb | 296 | Nonpaged | deskscan | 
	| Window Stations | Wind | Win\xe4 | 224 | Nonpaged | wndscan | 
	| Mutants | Mute | Mut\xe5 | 128 | Nonpaged | mutantscan | 
	| File Objects | File | Fil\xe5 | 288 | Nonpaged | filescan | 
	| Drivers | Driv | Dri\xf6 | 408 | Nonpaged | driverscan | 
	| Symbolic Links | Link | Lin\xeb | 104 | Nonpaged | symlinkscan | 
	- values should be adjusted for Windows type
- some installations of the Windows *Driver Development Kit(DDK)* and debugging tools include a `pooltag.txt` you can use to perform lookups

## 10. PoolMon
- memory pool monitor distributed with DDK
- reports live updates of pool tags in use along with:
	- memory type
	- number of allocations
	- number of frees
	- total number of bytes occupied by allocations
	- average bytes per allocation
- example output:
	``` cmd
	C:\WinDDK\7600.16385.1\tools\Other\i386> poolmon.exe -b
	Memory: 2096696K Avail: 1150336K PageFlts: 8135 InRam Krnl: 5004K P:158756K
	Commit:1535208K Limit:4193392K Peak:2779016K Pool N:43452K P:187796K
	System pool information
	Tag 	Type 	Allocs 		Frees 		Diff 	Bytes 			Per Alloc
	CM31 	Paged 	169392( 0) 	153744( 0) 	15648 	74838016( 0) 	4782
	MmSt 	Paged 	673616( 16) 656049( 17) 17567 	28286672( -184) 1610
	MmRe 	Paged 	67417( 0) 	66213( 0) 	1204 	12613400( 0) 	10476
	[snip]
	```
- Windows kernel debuger can also help determine pool tag associations:
	``` cmd
	kd> !poolfind Proc
	Searching NonPaged pool (fffffa8000c02000 : ffffffe000000000) for Tag: Proc
	*fffffa8000c77000 size: 430 previous size: 0  (Allocated) Proc (Protected)
	*fffffa8001346000 size: 430 previous size: 0  (Allocated) Proc (Protected)
	*fffffa8001361000 size: 430 previous size: 0  (Allocated) Proc (Protected)
	*fffffa800138f7a0 size: 430 previous size: 30 (Free) 	  Pro.
	*fffffa80013cb1e0 size: 430 previous size: c0 (Allocated) Proc (Protected)
	*fffffa80013e4460 size: 430 previous size: f0 (Allocated) Proc (Protected)
	*fffffa80014fd000 size: 430 previous size: 0  (Allocated) Proc (Protected)
	*fffffa800153ebd0 size: 10 	previous size: 70 (Free) 	  Pro.
	[snip]
	```
	- free block at 138f7a0 likely contains terminated `_EPROCESS`

## 11. pool tracker plugin
- statistics PoolMon reads can be accessed via `_KDDEBUGGER_DATA64` that stores active process and loaded modules lists
- `PoolTrackTable` member points to an array of `_POOL_TRACKER_TABLE` structures
- `_POOL_TRACKER_TABLE` structure:
	``` python
	>>> dt("_POOL_TRACKER_TABLE")
	'_POOL_TRACKER_TABLE' (40 bytes)
	0x0 : Key 				['long']
	0x4 : NonPagedAllocs 	['long']
	0x8 : NonPagedFrees 	['long']
	0x10 : NonPagedBytes 	['unsigned long long']
	0x18 : PagedAllocs 		['unsigned long']
	0x1c : PagedFrees 		['unsigned long']
	0x20 : PagedBytes 		['unsigned long long']
	```
	- each table has a `key`, the 4-byte tag
- `pooltracker` plugin usage:
	``` sh
	$ python vol.py -f win7x64.dd pooltracker --profile=Win7SP0x64 --tags=Proc,File,Driv,Thre
	Volatility Foundation Volatility Framework 2.4
	Tag 	NpAllocs 	NpFrees 	NpBytes 	PgAllocs PgFrees  PgBytes
	------ 	-------- 	-------- 	-------- 	-------- -------- --------
	Thre 	614895 		614419 		606688 		0 		 0 		  0
	File 	75346601 	75336591 	3350912 	0 		 0 		  0
	Proc 	4193 		4154 		51728 		0 		 0 		  0
	Driv 	143 		6 			67504 		0 		 0 		  0
	```
	- running without `--tags` will display statistics for all pool tags
	- `--tagfile` can be used to integrate data from `pooltag.txt`; output is labeled with description and owning kernel driver

## 12. psscan plugin
- `psscan` usage:
	``` sh
	$ python vol.py -f win7x64.dd --profile=Win7SP0x64 psscan
	Volatility Foundation Volatility Framework 2.4
	Offset(P) 	Name 			PID 	PPID 	Time created 		Time exited
	--------- 	-------------- 	------ 	------ 	------------------- -------------------
	0x02dc1a70 	svchost.exe 	2016 	448		2011-12-30 07:29:13
	0x0af80b30 	conhost.exe 	2476 	360		2012-01-20 17:54:37
	[snip]
	0x7e263470 PING.EXE 		788 	2708 	2012-03-11 09:00:03 2012-03-11 09:00:11
	0x7e36e8e0 IPCONFIG.exe 	2072 	2708 	2012-03-11 09:00:02 2012-03-11 09:00:02
	```

## 13. pool scanning limitations
- non-malicious limitations:
	- untagged pool memory: pool memory may not be allocated with tag
	- false positives
	- large allocations: pool tag scanning doesn't work for allocations larger than 4096 bytes
- malicious limitations due to anti-forensics:
	- arbitrary tags: malicious drivers can use same tag benign drivers use
	- decoy tags: decoy objects can mislead investigators
	- manipulated tags: rootkits can modify pool tags
- most anti-forensics techniques can be countered by corroborating with other sources

## 14. big page pool
- blocks of memory are allocated from here if size exceeds 1 page (4096 bytes)
- `_POOL_HEADER` is not used

## 15. big page track tables
- do not store statistics like pool track tables
- includes addresses of allocations
- `nt!PoolBigPageTable`, which points to array of `_POOL_TRACKER_BIG_PAGES` is not exported nor copied to kernel debugger data block
- can always be found at a predictable location relative to `nt!PoolTrackTable`
- bit page track table structure:
	``` python
	>>> dt("_POOL_TRACKER_BIG_PAGES")
	'_POOL_TRACKER_BIG_PAGES' (24 bytes)
	0x0 : Va 				['pointer64', ['void']]
	0x8 : Key 				['unsigned long']
	0xc : PoolType 			['unsigned long']
	0x10 : NumberOfBytes 	['unsigned long long']
	```
	- `Va` points to base address of allocation

## 16. bigpools plugin
- `bitpools` plugin output:
	``` sh
	$ python vol.py -f win7x64cmd.dd --profile=Win7SP0x64 bigpools
	Volatility Foundation Volatility Framework 2.4
	Allocation 			Tag 	 PoolType 						NumberOfBytes
	------------------ 	-------- -------------------------- 	-------------
	0xfffff8a003747000 	CM31 	 PagedPoolCacheAligned 			0x1000L
	0xfffff8a00f9a8001 	CM31 	 PagedPoolCacheAligned 			0x1000L
	[snip]
	```
	- addresses that end with 1 means marked as free

## 17. pool scanning alternatives
- dispatcher head scans:
	- to enable synchronization functionality for certain objects, their state info is stored in `_DISPATCHER_HEADER` at start of executive object structures
	- `_DISPATCHER_HEADER` contains values consistent accross memory dumps; easy to build signature
