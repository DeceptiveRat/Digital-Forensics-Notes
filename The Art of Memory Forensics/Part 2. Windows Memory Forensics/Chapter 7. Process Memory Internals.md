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
		- tracks reserved or commited, virtually contiguous collections of pages
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
	- process heaps are read
	- thread stacks are green
	- mapped fields are yellow
