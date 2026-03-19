## 1. Operating System
- deals with low-level details of hardware
- implements a set of high-level services and interfaces that define how hardware can be accessed

![[Computer Architecture Overview.png]]

## 2. motherboard
- circuit board that connects various components
- communication channels are called *computer buses*

## 3. *Central Processing Unit(CPU)*
- accesses main memory to obtain instructions then executes instructions
- relies on *Memory MAnagement Unit(MMU)* to find data

## 4. MMU
- translates address that processor requests to address in main memory
- DS for address translation also stored in memory
- special cache *Translation Lookaside Buffer(TLB)* used for MMU translation table; used before MMU for quick lookups

## 5. *memory controller*
- mediates potentially concurrent requests for system memory from processor and devices

## 6. *Direct Memory Access(DMA)*
- CPU initiates data transfer and DMA controller manages data transfer
- can directly access contents of physical memory without involivng untrusted software

## 7. *Random Access Memory(RAM)*
- volatile memory; lost without power
- constant access time regardless of location

## 8. address space
- refers to range of valid addresses used to identify data stored
- *linear addresses*, or *virtual addresses*, are translated into *physical addresses* for accessing physical memory

## 9. IA-32
- x86 architectures that support 32-bit computation
- specifies instruction set and programming environment for Intel's 32-bit processors
- little endian machine that uses byte addressing
- max linear/physical address 4GB
- size of physical memory can be expanded to 64GB using *Physical Address Extension(PAE)* 

## 10. IA-32 reigisters
- processor core contains 8 32-bit general purpose registers for logical and arithmetic operations
- EIP register:
	- aka *program counter*
	- contains linear address of next instruction
- 5 control registers that specify configuration of processor and characteristics of executing task
- CR0:
	- flags controlling operating mode of processor
	- e.g. paging
- CR1:
	- reserved
	- should not be addressed
- CR2:
	- linear addresses that caused page fault
- CR3: 
	- physical address of initial structure used for address translation
- CR4: 
	- enable architectural extensions
	- e.g. PAE

## 11. *segmentation*
- divides 32-bit linear address space into multiple variable-length segments
- all IA-32 memory references are addressed using:
	- 16-bit segment selector to identify *segment descriptor*
	- 32-bit offset into specified segment
- segment descriptor:
	- memory-resident DS
	- defines location, size, type, and permissions for given segment
- processor core contains 2 special registers, GDTR and LDTR:
	- GDTR points to *Global Descriptor Table(GDT)*
	- LDTR points to *Local Descriptor Table(LDT)*
- segmentation registers should contain valid segment selectors:
	- CS: code
	- SS: stack
	- DS: data
	- ES: data
	- FS: data
	- GS: data
- segmented addressing is hidden by defining overlapping segments with base address 0

## 12. *paging*
- provides ability to virtualize linear address space
- large linear space is simulated with physical memory and disk storage
- 32-bit linear address space is broken up into fixed-length sections, *pages*
- pages are mapped into physical memory in arbitrary order
- memory-resident *page directories* and *page tables* are used to translate linear addresses to physical addresses
- translating *virtual address(VA)* to *physical address(PA)*:
	- bits 32:12 of CR3 register + bits 32:22 of VA = *Page Directory Entry(PDE)* address
	- bits 31:12 of PDE + bits 21:12 of VA = *Page Table Entry(PTE)* address
	- bits 31:12 of PTE + bits 11:0 of VA = PA

![[address translation example.png]]
![[paging structure address format.png]]

- example translation:

![[VA to PA.png]]

## 13. PAE
- allows processors to support address spaces greater than 4GB
- linear address divided into 4 indexes:
	- *Page Directory Pointer Table(PDPT)*
	- *Page Directory(PD)*
	- *Page Table(PT)*
	- page offset
- paging structure entries are 64-bits
- translating VA to PA:
	- bits 31:30 from VA select PDPTE
	- bits 29:21 select from 512 PDEs
	- if PS flag is set, PDE maps 2MB page; else bits 20:12 select from 512 PTEs
	- bits 11:0 specify offset within page for PA

![[VA to PA with PAE.png]]
![[paging structure address format with PAE.png]]

## 14. Intel 64
- registers in IA-32 have been expanded to hold 64 bits
- most current implmentations use 48-bit linear addresses only
- bits 63:48 are set to all 1 or 0 depending on status bit of 47; *sign-extension*
- supports additional level of paging structures, *Page Map Level 4(PML4)*
- all entries in hierarchy of paging structures are 64-bits; can map to pages of size 4KB, 2MB, or 1GB
![[4KB page address translation.png]]
![[AMD64 paging structure addresses.png]]
	- VA:
		- bits 47:39 - PML4E offset
		- bits 38:30 - PDPTE offset
		- bits 23:21 - PDE offset
		- bits 20:12 - PTE offset
	- PDPTE: if PS set, entry maps to 1GB page
	- PDE: if PS set, entry maps to 2MB page

## 15. *Interrupt Descriptor Table(IDT)*
- stores interrupt software routines
- each processor has IDT composed of 256 8-byte or 16-byte entries
- first 32 entries are reserved for processor-defined exceptions and interrupts
- process:
	- interrupt number serves as index to IDT
	- entry contains address of *Interrupt Service Routine(ISR)*; indirectly references segment in GDT
	- respective handler is called
- IDT is also used to store handlers for other events:
	- system calls
	- debugger break points
	- other faults
- frequent target for malware

## 16. privilege separation
- untrusted/trusted applications run in user/kernel mode
- enforced by IA-32 processor architecture through *protection rings*:
	- ring 0: kernel mode; most privileged
	- ring 3: user mode; least privileged
- apps swtich from user mode to kernel mode through a set of system calls

## 17. system calls
- used by user app to request service from kernel; e.g:
	- file I/O
	- network communication
	- process spawning
- define low-level API between user app and kernel
- OS typically define set of stable APIs that map to one or more system calls; e.g. ntdll.dll and kernel32.dll
- before user app makes syste call, it must pass arguments via registers or the stack
- invoking the system call:
	1. user app executes a software interrupt or architecture-specific instruction
	2. user mode register context is saved
	3. execution mode changed to kernel
	4. kernel stack is initialized
	5. system call handler is invoked
	6. register context restored
	7. execution mode changed to user
	8. control returns to user app
- code used to service system call is inspected by security products

## 18. process
- program executing in memory
- each process has set of attributes, such as unique process ID and address space
- *process address space* contains:
	- application code
	- shared libraries
	- dynamic data
	- runtime stack
- process provides execution environment, resources, and context for *threads*

## 19. thread
- basic unit of CPU utilization and execution
- characterized by:
	- thread ID
	- CPU register set
	- execution stacks
- threads from same process share:
	- code
	- data
	- address space
	- OS resources
- thread data structures often contain useful data, timestamps and starting addresses

## 20. CPU scheduling
- OS capability to distribute execution time among multiple threads
- OS scheduler determines which threads execute and for how long

## 21. context switch
- switching execution to different thread
- context switch steps:
	1. suspend thread
	2. stores execution context in main memory
	3. retrieve execution context of different thread from memory
	4. update register states
	5. resume execution of new thread
- saved execution contexts can provide valuable insight during memory analysis
- execution context includes CPU register values, e.g. instruction pointer

## 22. system resources
- most OS maintain DS for managing:
	- resources actively being accessed
	- which process can access them
	- how they are accessed
	- e.g. Windows leverages object manager to supervise use of system resources and stores info in handler table; handler provides process with unique identifier for accessing and changing resources
- examples of resources:
	- processes
	- threads
	- files
	- network sockets
	- synchronization objects
	- regions of shared memory

## 23. virtual memory
- OS provides process with private VA space
- creates separation between logical memory that process sees and physical memory
- memory manager is responsible for transferring regions of memory to secondary storage to free up physical memory
- memory manager and MMU work together to translate VA into PA
- range of accessible address for a process is frequently partitioned into addresses associated with OS(consistent) and private addresses(varies)

## 24. *demand paging*
- mechanism used to implement virtual memory
- memory management policy for determining which regions are in memory and which are moved to secondary storage
- secondary storage partition or file is called *swap* or *page file*
- relies on *locality of reference*, meaning memory locations are likely to be frequently accessed in a short period of time
- demand paging reduces time to load process and increases number of processes loaded
- thread accessing non-resident page triggers page fault

## 25. shared memory
- memory accessible from more than one *Virtual Address Space(VAS)*
- efficient means of *Inter-Process Communication(IPC)*
- also used to conserve physical memory:
	- shared/dynamic libraries taht contain common code and data
	- mapped as *copy-on-write*

## 26. *stack*
- holds temporary data associated with executing functions
- stored in DS called *stack frame* containing:
	- function parameters
	- local variables
	- info required to recover previous stack frame
- stack frames are pushed when calling a function and popped when returning 
- OS uses separate stack for functions executed within each mode
- provides insight into which code was being exectued and what data was being processed

## 27. *heap*
- can persist for lifetime of process
- stores information whose length/contents can be unknown at compile time
- OS may have regions of memory dynamically allocated within kernel mode; e.g:
	- Windows creates paged and nonpaged regions in kernel called *pools*
- interesting data that can be found:
	- data read from files on disk
	- data transferred over network
	- input typed from keyboard

## 28. *device drivers*
- mechanism for extending the capabilities of kernel to support new devices
- abstracts away details of how device controls data
- typically communicate with registers of device controller
- most CPU architectures map memory and registers of I/O devices into VAS; *memory mapped I/O*
